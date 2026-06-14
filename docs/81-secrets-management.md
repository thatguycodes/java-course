# Secrets Management

[← Back to README](../README.md)

---

**Secrets management** is the practice of storing, accessing, and rotating sensitive values (database passwords, API keys, TLS certificates) outside of application code and environment variables checked into source control. The two most common solutions in the Java/Spring ecosystem are **HashiCorp Vault** and **AWS Secrets Manager**.

```mermaid
flowchart LR
    dev["Developer\n(writes code)"]
    ci["CI/CD Pipeline"]
    vault["HashiCorp Vault\nor AWS Secrets Manager"]
    app["Spring Boot App\n(running in prod)"]

    dev -->|no secrets in code| ci
    ci -->|deploy| app
    app -->|fetch at startup| vault
    vault -->|inject into Spring env| app
```

---

## What NOT to Do

```bash
# Never commit secrets
DB_PASSWORD=supersecret        # in .env → committed to Git
spring.datasource.password=abc # in application.yml → in source repo
```

Use `.gitignore` for all `.env` files and rotate any secret ever committed.

---

## HashiCorp Vault

### Running Vault Locally

```yaml
# compose.yml
services:
  vault:
    image: hashicorp/vault:1.17
    ports:
      - "8200:8200"
    environment:
      VAULT_DEV_ROOT_TOKEN_ID: dev-root-token
      VAULT_DEV_LISTEN_ADDRESS: 0.0.0.0:8200
    cap_add:
      - IPC_LOCK
```

### Store Secrets in Vault

```bash
export VAULT_ADDR=http://localhost:8200
export VAULT_TOKEN=dev-root-token

# KV secrets engine (v2)
vault kv put secret/order-service \
  db.password=secretpassword \
  stripe.api-key=sk_live_abc123 \
  jwt.secret=myjwtsecret
```

### Spring Cloud Vault Setup

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-vault-config</artifactId>
</dependency>
```

```yaml
# application.yml
spring:
  application:
    name: order-service
  cloud:
    vault:
      uri: http://localhost:8200
      token: ${VAULT_TOKEN}           # injected at deploy time, not hardcoded
      kv:
        enabled: true
        default-context: order-service  # reads secret/order-service
        backend: secret
      config:
        lifecycle:
          enabled: true               # auto-renew leases
```

Spring Cloud Vault fetches secrets at startup and injects them as Spring `Environment` properties:

```java
@Value("${db.password}")
private String dbPassword;

@Value("${stripe.api-key}")
private String stripeApiKey;
```

---

## Dynamic Database Credentials

Vault can generate short-lived database credentials that expire automatically — no static password to rotate:

```bash
# Enable database secrets engine
vault secrets enable database

# Configure PostgreSQL connection
vault write database/config/orders-db \
  plugin_name=postgresql-database-plugin \
  connection_url="postgresql://{{username}}:{{password}}@localhost:5432/orders" \
  allowed_roles="order-service-role" \
  username="vault_admin" \
  password="vaultpassword"

# Create a role — Vault generates credentials with a 1-hour TTL
vault write database/roles/order-service-role \
  db_name=orders-db \
  creation_statements="CREATE ROLE \"{{name}}\" WITH LOGIN PASSWORD '{{password}}' VALID UNTIL '{{expiration}}';" \
  default_ttl="1h" \
  max_ttl="24h"
```

```yaml
spring:
  cloud:
    vault:
      database:
        enabled: true
        role: order-service-role
        backend: database
```

Spring rotates the credentials before they expire — no restarts needed.

---

## AWS Secrets Manager

### Maven Dependency

```xml
<dependency>
    <groupId>io.awspring.cloud</groupId>
    <artifactId>spring-cloud-aws-starter-secrets-manager</artifactId>
    <version>3.1.1</version>
</dependency>
```

### Store Secrets in AWS

```bash
aws secretsmanager create-secret \
  --name /order-service/prod/db \
  --secret-string '{"password":"supersecret","url":"jdbc:postgresql://prod:5432/orders"}'
```

### Configuration

```yaml
spring:
  config:
    import: aws-secretsmanager:/order-service/prod/db

  cloud:
    aws:
      region:
        static: eu-west-1
      credentials:
        # Uses IAM role when running on EC2/ECS — no explicit credentials needed
```

Secrets from AWS Secrets Manager are mapped to Spring properties automatically:

```java
@Value("${password}")   // mapped from the JSON key
private String dbPassword;
```

### Using the AWS SDK Directly

```java
@Service
public class SecretService {

    private final SecretsManagerClient secretsManager;

    public String getSecret(String secretName) {
        GetSecretValueRequest request = GetSecretValueRequest.builder()
            .secretId(secretName)
            .build();

        return secretsManager.getSecretValue(request).secretString();
    }
}
```

---

## Environment Variables as a Bridge

When Spring Cloud Vault / AWS is not available (local dev), use environment variables:

```yaml
# application.yml
spring:
  datasource:
    password: ${DB_PASSWORD}    # from env, CI secret, or Vault
```

```bash
# Local dev — .env file (gitignored)
DB_PASSWORD=localdevpassword

# CI/CD — GitHub Actions secret
env:
  DB_PASSWORD: ${{ secrets.DB_PASSWORD }}
```

---

## Kubernetes Secrets

```yaml
# k8s secret — base64 encoded values
apiVersion: v1
kind: Secret
metadata:
  name: order-service-secrets
type: Opaque
stringData:
  db-password: supersecret      # stringData auto base64-encodes
  stripe-api-key: sk_live_abc
```

```yaml
# Mount as environment variables in the pod
env:
  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef:
        name: order-service-secrets
        key: db-password
```

For production, use the **External Secrets Operator** to sync Vault or AWS Secrets Manager into Kubernetes Secrets automatically:

```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: order-service-secrets
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: vault-backend
    kind: SecretStore
  target:
    name: order-service-secrets
  data:
    - secretKey: db-password
      remoteRef:
        key: secret/order-service
        property: db.password
```

---

## Secrets Management Summary

| Tool | Strengths | Use Case |
|------|-----------|----------|
| HashiCorp Vault | Dynamic creds, fine-grained policies, self-hosted | On-premise, multi-cloud |
| AWS Secrets Manager | Native AWS integration, auto-rotation | AWS workloads |
| Kubernetes Secrets | Built-in, no extra infra | K8s clusters (pair with External Secrets) |
| GitHub Actions Secrets | CI/CD only | Build and deploy pipelines |

| Best Practice | Detail |
|---------------|--------|
| Never commit secrets | Use `.gitignore` for `.env`; rotate any leaked secret immediately |
| Use IAM roles | Avoid static credentials for AWS — prefer instance/task roles |
| Rotate regularly | Dynamic credentials (Vault DB engine) expire automatically |
| Least privilege | Each service only has access to its own secrets |
| Audit access | Vault and AWS log every secret access |

---

[← Back to README](../README.md)
