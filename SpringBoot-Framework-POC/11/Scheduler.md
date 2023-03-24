# Cron Scheduler

### Task completed

1. Set up scheduler with **cron expression**
2. Add delay to a route
3. Allow a route at specific time only

<br>
## 1. Maven Dependencies

```xml
<dependency>
	<groupId>org.apache.camel</groupId>
	<artifactId>camel-cron</artifactId>
	<version>3.14.5</version>
</dependency>
<!-- This is for the subtask 3 -->
<dependency>
	<groupId>org.apache.camel</groupId>
	<artifactId>camel-quartz</artifactId>
	<version>3.14.5</version>
</dependency>
```

<br>
## 2. Cron Expression

It is a string with 6-7 subexpressions which describe the details of the scheduling. The format is:

**second  minute  hour  day_of_month  month  day_of_week  year**

where year is optional.

### Some of the special characters (* / -)

- `*` means any time (e.g. every day)
- `0/5` means every 5 (e.g. minutes)
- `MON-FRI` means every Monday to Friday

<br>
## 3. Route triggered with Cron Scheduling

Create a endpoint which schedule in specific time with cron expression.

```java
// Schedule to the endpoint every 5 sec
// sec(opt) min hr dayOfMon Mon dayOfWk Yr(opt)
from("cron:anyName?schedule=0/5 * * * * ?")
	.to("direct:endpoint_You_Want_To_Trigger");
```

<br>
## 4. Simple Delay

You can create a simple delay for a route by `delay()` .

```java
// Introduce a 5 sec delay every time
from("direct:start")
	.delay(5000)
	.to("direct:end");
```

<br>
## 5. Camel Quartz

It can perform the similar scheduling task as camel cron, while the syntax is a little more complicated. While it can **start**, **suspend**, **resume** and **stop** a route.

```java
// Create a route policy: only allow to enter at the first 10 sec every minute
CronScheduledRoutePolicy policy = new CronScheduledRoutePolicy();
policy.setRouteStartTime("0 * * * * ?");
policy.setRouteSuspendTime("10 * * * * ?");
policy.setRouteResumeTime("0 * * * * ?");

// Apply to a route
from("direct:10secGap")
	.routePolicy(policy).noAutoStartup()
	.to("direct:somewhere");
```