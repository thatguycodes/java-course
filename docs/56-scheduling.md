# Scheduling

[← Back to README](../README.md)

---

Scheduling runs tasks on a timed basis — nightly reports, periodic cache evictions, polling for updates, cleanup jobs. Spring Boot provides `@Scheduled` for simple in-process scheduling and integrates with Quartz for distributed, persistent job scheduling.

---

## Enabling Scheduling

```java
@SpringBootApplication
@EnableScheduling
public class App { ... }
```

---

## @Scheduled

```java
import org.springframework.scheduling.annotation.Scheduled;

@Component
public class ReportScheduler {

    private static final Logger log = LoggerFactory.getLogger(ReportScheduler.class);

    // fixed delay — wait 5s AFTER the previous execution finishes
    @Scheduled(fixedDelay = 5_000)
    public void pollForUpdates() {
        log.info("Polling for updates...");
    }

    // fixed rate — start every 10s regardless of how long the task takes
    @Scheduled(fixedRate = 10_000)
    public void syncCache() {
        log.info("Syncing cache...");
    }

    // initial delay — wait 30s before the first execution
    @Scheduled(fixedRate = 60_000, initialDelay = 30_000)
    public void warmUp() {
        log.info("Warming up...");
    }

    // cron expression — at 2:30 AM every day
    @Scheduled(cron = "0 30 2 * * *")
    public void generateNightlyReport() {
        log.info("Generating nightly report...");
    }

    // cron with zone
    @Scheduled(cron = "0 0 8 * * MON-FRI", zone = "Africa/Johannesburg")
    public void morningDigest() {
        log.info("Sending morning digest...");
    }
}
```

---

## Cron Expression Format

```
┌────────── second        (0-59)
│ ┌──────── minute        (0-59)
│ │ ┌────── hour          (0-23)
│ │ │ ┌──── day of month  (1-31)
│ │ │ │ ┌── month         (1-12 or JAN-DEC)
│ │ │ │ │ ┌ day of week   (0-7 or SUN-SAT, 0 and 7 = Sunday)
│ │ │ │ │ │
* * * * * *
```

| Expression | Meaning |
|-----------|---------|
| `0 * * * * *` | Every minute (at second 0) |
| `0 0 * * * *` | Every hour |
| `0 0 8 * * *` | Every day at 08:00 |
| `0 0 8 * * MON-FRI` | Weekdays at 08:00 |
| `0 0 0 1 * *` | First day of every month at midnight |
| `0 0 12 ? * SUN` | Every Sunday at noon |
| `0 */15 * * * *` | Every 15 minutes |
| `0 0 8,12,18 * * *` | At 08:00, 12:00, and 18:00 |

---

## Reading Delay from Configuration

```java
// fixed values in code are hard to change without recompiling
@Scheduled(fixedRateString = "${scheduler.sync-rate-ms:60000}")
public void syncData() { ... }

@Scheduled(cron = "${scheduler.report-cron:0 0 2 * * *}")
public void report() { ... }
```

```yaml
# application.yml
scheduler:
  sync-rate-ms: 30000
  report-cron: "0 30 1 * * *"
```

---

## Async Scheduled Tasks

By default all `@Scheduled` tasks run on a single thread — if one task runs long, others are delayed. Use `@Async` to run each task on its own thread.

```java
@Configuration
@EnableAsync
@EnableScheduling
public class SchedulerConfig {

    @Bean
    public TaskScheduler taskScheduler() {
        ThreadPoolTaskScheduler scheduler = new ThreadPoolTaskScheduler();
        scheduler.setPoolSize(5);
        scheduler.setThreadNamePrefix("scheduled-");
        scheduler.initialize();
        return scheduler;
    }

    @Bean
    public Executor asyncExecutor() {
        ThreadPoolTaskExecutor exec = new ThreadPoolTaskExecutor();
        exec.setCorePoolSize(4);
        exec.setMaxPoolSize(10);
        exec.setQueueCapacity(100);
        exec.setThreadNamePrefix("async-");
        exec.initialize();
        return exec;
    }
}

@Component
public class DataSyncScheduler {

    @Async
    @Scheduled(fixedRate = 30_000)
    public void syncUsersAsync() {
        // runs on async thread pool — won't block other @Scheduled methods
    }
}
```

---

## Programmatic Scheduling

For dynamic schedules (e.g., per-user job frequency from a database):

```java
import org.springframework.scheduling.TaskScheduler;
import org.springframework.scheduling.support.CronTrigger;
import java.time.Duration;

@Service
public class DynamicScheduler {

    private final TaskScheduler taskScheduler;
    private final Map<String, ScheduledFuture<?>> jobs = new ConcurrentHashMap<>();

    public DynamicScheduler(TaskScheduler taskScheduler) {
        this.taskScheduler = taskScheduler;
    }

    public void scheduleJob(String jobId, String cronExpression, Runnable task) {
        cancelJob(jobId);   // cancel existing if any
        ScheduledFuture<?> future = taskScheduler.schedule(task, new CronTrigger(cronExpression));
        jobs.put(jobId, future);
        log.info("Scheduled job {} with cron: {}", jobId, cronExpression);
    }

    public void scheduleAtFixedRate(String jobId, Duration rate, Runnable task) {
        cancelJob(jobId);
        ScheduledFuture<?> future = taskScheduler.scheduleAtFixedRate(task, rate);
        jobs.put(jobId, future);
    }

    public void cancelJob(String jobId) {
        ScheduledFuture<?> existing = jobs.remove(jobId);
        if (existing != null) existing.cancel(false);
    }
}
```

---

## Quartz — Distributed Scheduling

For production systems running multiple instances, `@Scheduled` runs on every node — Quartz ensures a job runs on **exactly one** node.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-quartz</artifactId>
</dependency>
```

```yaml
spring:
  quartz:
    job-store-type: jdbc          # state persisted in DB — survives restarts
    jdbc:
      initialize-schema: always   # create quartz tables automatically
    properties:
      org.quartz.jobStore.isClustered: true   # only one node runs each job
```

```java
@Component
public class ReportJob implements Job {

    @Autowired
    private ReportService reportService;   // Spring beans injected via AutowiringSpringBeanJobFactory

    @Override
    public void execute(JobExecutionContext context) throws JobExecutionException {
        reportService.generateReport();
    }
}

@Configuration
public class QuartzConfig {

    @Bean
    public JobDetail reportJobDetail() {
        return JobBuilder.newJob(ReportJob.class)
            .withIdentity("reportJob")
            .storeDurably()
            .build();
    }

    @Bean
    public Trigger reportTrigger(JobDetail reportJobDetail) {
        return TriggerBuilder.newTrigger()
            .forJob(reportJobDetail)
            .withIdentity("reportTrigger")
            .withSchedule(CronScheduleBuilder.cronSchedule("0 0 2 * * ?"))
            .build();
    }
}
```

---

## @Scheduled vs Quartz

| | `@Scheduled` | Quartz |
|--|-------------|--------|
| Setup | `@EnableScheduling` only | `spring-boot-starter-quartz` + DB |
| Distributed | Runs on every node | Runs on exactly one node |
| Persistence | No — lost on restart | Yes — survives restarts |
| Dynamic jobs | Limited | Full support |
| Best for | Single-instance, simple schedules | Multi-instance, critical jobs |

---

## Scheduling Summary

| Task | Annotation / API |
|------|-----------------|
| Enable | `@EnableScheduling` |
| Fixed delay (after task) | `@Scheduled(fixedDelay = ms)` |
| Fixed rate (every n ms) | `@Scheduled(fixedRate = ms)` |
| Cron expression | `@Scheduled(cron = "0 0 2 * * *")` |
| Timezone | `@Scheduled(cron = "...", zone = "Africa/Johannesburg")` |
| From config | `@Scheduled(fixedRateString = "${key}")` |
| Async execution | `@Async` + `@EnableAsync` + thread pool bean |
| Dynamic scheduling | `TaskScheduler.schedule(task, trigger)` |
| Distributed (one node) | Quartz with `job-store-type: jdbc` |

---

[← Back to README](../README.md)
