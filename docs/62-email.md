# Email with Spring Mail

[← Back to README](../README.md)

---

Spring Mail wraps the JavaMail API, making it straightforward to send plain text, HTML, and emails with attachments from a Spring Boot application.

---

## Maven Dependency

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-mail</artifactId>
</dependency>

<!-- Thymeleaf for HTML email templates (optional) -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
<dependency>
    <groupId>org.thymeleaf.extras</groupId>
    <artifactId>thymeleaf-extras-java8time</artifactId>
</dependency>
```

---

## Configuration

```yaml
# application.yml

# Gmail
spring:
  mail:
    host: smtp.gmail.com
    port: 587
    username: ${GMAIL_ADDRESS}
    password: ${GMAIL_APP_PASSWORD}   # use an App Password, not your account password
    properties:
      mail:
        smtp:
          auth: true
          starttls:
            enable: true
            required: true

# AWS SES
spring:
  mail:
    host: email-smtp.eu-west-1.amazonaws.com
    port: 587
    username: ${SES_SMTP_USER}
    password: ${SES_SMTP_PASSWORD}
    properties:
      mail.smtp.auth: true
      mail.smtp.starttls.enable: true

# SendGrid
spring:
  mail:
    host: smtp.sendgrid.net
    port: 587
    username: apikey
    password: ${SENDGRID_API_KEY}
    properties:
      mail.smtp.auth: true
      mail.smtp.starttls.enable: true
```

---

## Sending a Simple Text Email

```java
import org.springframework.mail.SimpleMailMessage;
import org.springframework.mail.javamail.JavaMailSender;

@Service
public class EmailService {

    private final JavaMailSender mailSender;

    @Value("${spring.mail.username}")
    private String fromAddress;

    public EmailService(JavaMailSender mailSender) {
        this.mailSender = mailSender;
    }

    public void sendText(String to, String subject, String body) {
        SimpleMailMessage message = new SimpleMailMessage();
        message.setFrom(fromAddress);
        message.setTo(to);
        message.setSubject(subject);
        message.setText(body);
        mailSender.send(message);
    }
}
```

---

## HTML Email

```java
import org.springframework.mail.javamail.MimeMessageHelper;
import jakarta.mail.internet.MimeMessage;

public void sendHtml(String to, String subject, String htmlBody) throws MessagingException {
    MimeMessage mime = mailSender.createMimeMessage();
    MimeMessageHelper helper = new MimeMessageHelper(mime, "UTF-8");

    helper.setFrom(fromAddress);
    helper.setTo(to);
    helper.setSubject(subject);
    helper.setText(htmlBody, true);   // true = HTML

    mailSender.send(mime);
}
```

---

## HTML Template with Thymeleaf

```html
<!-- src/main/resources/templates/email/welcome.html -->
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<body>
    <h1>Welcome, <span th:text="${name}">User</span>!</h1>
    <p>Your account has been created with email: <strong th:text="${email}"></strong></p>
    <p>
        <a th:href="${activationUrl}">Activate your account</a>
    </p>
    <p>This link expires on <span th:text="${#temporals.format(expiresAt, 'dd MMMM yyyy')}"></span>.</p>
</body>
</html>
```

```java
import org.thymeleaf.TemplateEngine;
import org.thymeleaf.context.Context;

@Service
public class TemplatedEmailService {

    private final JavaMailSender mailSender;
    private final TemplateEngine templateEngine;

    @Value("${spring.mail.username}")
    private String fromAddress;

    public TemplatedEmailService(JavaMailSender mailSender, TemplateEngine templateEngine) {
        this.mailSender     = mailSender;
        this.templateEngine = templateEngine;
    }

    public void sendWelcome(String to, String name, String activationToken)
            throws MessagingException {

        Context ctx = new Context();
        ctx.setVariable("name", name);
        ctx.setVariable("email", to);
        ctx.setVariable("activationUrl",
            "https://app.example.com/activate?token=" + activationToken);
        ctx.setVariable("expiresAt", LocalDateTime.now().plusHours(24));

        String html = templateEngine.process("email/welcome", ctx);

        MimeMessage mime = mailSender.createMimeMessage();
        MimeMessageHelper helper = new MimeMessageHelper(mime, true, "UTF-8");
        helper.setFrom("noreply@example.com", "My App");
        helper.setTo(to);
        helper.setSubject("Welcome to My App");
        helper.setText(html, true);

        mailSender.send(mime);
    }
}
```

---

## Email with Attachments

```java
public void sendWithAttachment(String to, String subject,
                               String body, File attachment) throws MessagingException {
    MimeMessage mime = mailSender.createMimeMessage();
    MimeMessageHelper helper = new MimeMessageHelper(mime, true);  // true = multipart

    helper.setFrom(fromAddress);
    helper.setTo(to);
    helper.setSubject(subject);
    helper.setText(body, true);

    helper.addAttachment(attachment.getName(), attachment);

    // attach from byte array
    helper.addAttachment("report.pdf",
        new ByteArrayResource(generatePdfBytes()));

    // attach from classpath resource
    helper.addAttachment("logo.png",
        new ClassPathResource("static/images/logo.png"));

    mailSender.send(mime);
}
```

---

## Inline Images

```java
public void sendWithInlineImage(String to, String subject, byte[] imageBytes)
        throws MessagingException {

    MimeMessage mime = mailSender.createMimeMessage();
    MimeMessageHelper helper = new MimeMessageHelper(mime, true);

    helper.setFrom(fromAddress);
    helper.setTo(to);
    helper.setSubject(subject);

    String html = "<html><body><h1>Hello!</h1>"
                + "<img src='cid:logo'/>"
                + "</body></html>";
    helper.setText(html, true);

    // cid:logo matches the content ID below
    helper.addInline("logo", new ByteArrayResource(imageBytes), "image/png");

    mailSender.send(mime);
}
```

---

## Async Email Sending

Email sending blocks the current thread (network I/O). Always send asynchronously in production.

```java
@Service
public class AsyncEmailService {

    private final EmailService emailService;

    @Async   // requires @EnableAsync
    public CompletableFuture<Void> sendWelcomeAsync(String to, String name, String token) {
        try {
            emailService.sendWelcome(to, name, token);
            return CompletableFuture.completedFuture(null);
        } catch (Exception e) {
            log.error("Failed to send welcome email to {}", to, e);
            return CompletableFuture.failedFuture(e);
        }
    }
}
```

---

## Testing Email in Development

**Mailpit** is a local SMTP server with a web UI — emails are captured and displayed without being delivered.

```yaml
# compose.yml
services:
  mailpit:
    image: axllent/mailpit:latest
    ports:
      - "1025:1025"   # SMTP
      - "8025:8025"   # web UI → http://localhost:8025
```

```yaml
# application-local.yml
spring:
  mail:
    host: localhost
    port: 1025
    properties:
      mail.smtp.auth: false
      mail.smtp.starttls.enable: false
```

All emails sent by the app appear in the Mailpit UI — no real emails are sent.

---

## Email Summary

| Task | API |
|------|-----|
| Simple text email | `SimpleMailMessage` + `JavaMailSender.send()` |
| HTML email | `MimeMessageHelper(mime, "UTF-8")` + `setText(html, true)` |
| Thymeleaf template | `TemplateEngine.process("template", ctx)` |
| Attachment | `MimeMessageHelper.addAttachment(name, file)` |
| Inline image | `MimeMessageHelper.addInline(cid, resource, type)` |
| Async sending | `@Async` + `@EnableAsync` |
| Local testing | Mailpit on port 1025 |
| Display name | `helper.setFrom("addr", "Display Name")` |

---

[← Back to README](../README.md)
