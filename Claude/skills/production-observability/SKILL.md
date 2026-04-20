---
name: production-observability
description: Use when adding logging, metrics, tracing, or alerting to production systems; debugging intermittent failures; or instrumenting code for monitoring.
---

# Production Observability

## Overview
Make systems observable through logging (events), metrics (trends), tracing (flows), and alerting (notifications). Core principle: Instrument proactively - gather evidence before issues occur to turn unknowns into knowns.

## When to Use
- Debugging production-only or intermittent failures
- Adding monitoring to new features before deploy
- Investigating performance, errors, or anomalies
- Setting up alerts for critical thresholds
- When systematic-debugging needs production data

Do NOT use for local dev bugs (use systematic-debugging).

## Core Pattern
1. Logs: Structured, contextual, leveled
2. Metrics: Counts, timings, gauges
3. Traces: Spans with propagation
4. Alerts: Threshold/anomaly-based

## Quick Reference
| Aspect | Key Practices | Tools |
|--------|---------------|-------|
| Logging | JSON format, correlation IDs, redact sensitive data | SLF4J/Logback, ELK, CloudWatch |
| Metrics | Requests/errors/latency, custom business metrics | Micrometer, Prometheus, StatsD |
| Tracing | Request flows, error propagation | OpenTelemetry, Jaeger, Zipkin |
| Alerting | SLO-based, fatigue prevention | PagerDuty, Opsgenie, Grafana |

## Implementation
Follow phases:
1. **Assess Current**: Search codebase for existing instrumentation (grep for log/metrics libs).
2. **Instrument**: Add at boundaries/errors/decisions. Example (Java/Spring):
   ```java
   @Service
   class UserService {
     private static final Logger log = LoggerFactory.getLogger(UserService.class);
     private final MeterRegistry metrics;

     public User login(String username) {
       log.info("login_attempt username={}", username);
       try (Scope span = tracer.startScopedSpan("user.login")) {
         // logic
         metrics.counter("logins.success").increment();
         return user;
       } catch (Exception e) {
         log.error("login_failed username={}", username, e);
         metrics.counter("logins.failure").increment();
         throw e;
       }
     }
   }
   ```
3. **Monitor/Alert**: Set dashboards/alerts (e.g., error rate >1%).
4. **Verify**: Check logs/metrics in staging; deploy canary.
5. **Analyze**: Use data for systematic-debugging.

## Common Mistakes
- Over-logging without sampling → High costs/noise
- Missing context (IDs, timestamps) → Hard to correlate
- Alerting on everything → Fatigue/ignored alerts
- No redaction → Leaking sensitive data in logs

## Red Flags
- "Can't reproduce in dev" → Add production instrumentation first
- Guessing without data → Gather evidence
- "We'll monitor later" → Instrument before deploy
- High alert volume → Tune thresholds, add suppression

## Rationalizations Table
| Excuse | Reality |
|--------|---------|
| "Too busy for monitoring" | Unmonitored code leads to longer outages |
| "Simple feature, no need" | Simple code fails; basics take minutes |
| "We'll add later" | Retroactive instrumentation misses initial failures |
| "Console.log is enough" | Unstructured logs are hard to query/alert on |

Pair with systematic-debugging for analysis, test-driven-development for instrumentation code.