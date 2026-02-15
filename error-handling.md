# Error Handling

## Applies To

- Web applications and APIs
- Mobile and desktop clients
- Background workers and scheduled jobs
- Distributed services and microservices

## Summary

Secure error handling ensures failures are controlled, observable, and safe.
Users should receive clear but minimal error responses, while operators get detailed diagnostics through protected logs and telemetry.

Poor error handling commonly causes:

- Information disclosure (stack traces, SQL/schema details, internal paths)
- Inconsistent application state
- Availability issues from retry storms or crash loops
- Missed detection of attacks and abuse patterns

## Security Objectives

1. Prevent sensitive information leakage in error responses.
2. Preserve system integrity and consistent state on failure.
3. Maintain service availability under fault conditions.
4. Provide actionable diagnostics for investigation and recovery.

## Core Principles

1. **Fail safely.** Default to deny/block behavior on security-relevant uncertainty.
2. **Separate user errors from system faults.** Handle validation and business errors differently from infrastructure failures.
3. **Never trust exception content for client output.** Treat exception messages as internal diagnostics only.
4. **Centralize handling.** Use global handlers/middleware to enforce consistent response and logging policy.
5. **Log rich context, return minimal detail.** Correlate events with IDs and traces, not raw internal state.
6. **Preserve invariants.** Use transactions/compensation to avoid partial commits and corruption.

## Error Taxonomy (Recommended)

- **Client/Input errors**: malformed request, validation failure, unsupported format.
- **Authorization errors**: unauthenticated/forbidden actions.
- **Domain errors**: business rule violations.
- **Dependency errors**: DB, queue, cache, third-party API failures.
- **System errors**: runtime faults, resource exhaustion, unhandled exceptions.

Define explicit mappings from each class to:

- HTTP/status codes (or equivalent protocol codes)
- Retry policy
- Logging severity
- Alerting thresholds

## Implementation Guidance

1. **Create a standard error model.**
   Include stable machine-readable fields (code, message key, correlation ID, timestamp).
   Avoid embedding stack traces or internal class names in client responses.

2. **Use global exception handlers.**
   Enforce one path for unexpected exceptions with safe fallback responses.

3. **Validate early and return deterministic errors.**
   Input/contract validation should fail before expensive processing.

4. **Apply transaction boundaries.**
   Roll back on failure; avoid partial writes.

5. **Set timeout and circuit-breaker policies for dependencies.**
   Prevent cascading failures and blocked request threads.

6. **Bound retries and use jittered backoff.**
   Never retry indefinitely; avoid synchronized retry storms.

7. **Protect logs.**
   Exclude secrets and sensitive personal data. Mask or tokenize where needed.

8. **Use correlation IDs end-to-end.**
   Include them in logs, traces, and client-safe error payloads.

9. **Support graceful degradation.**
   Disable non-critical features under dependency failure rather than full outage.

10. **Test error paths as first-class behavior.**
   Add unit, integration, and chaos/failure tests for timeout, partial failure, and malformed input scenarios.

## Client Response Rules

- Do return:
  - stable error code
  - short user-safe message
  - correlation/request ID
- Do not return:
  - stack trace
  - SQL queries/schema names
  - hostnames, internal IPs, filesystem paths
  - secrets/tokens/credentials

## Logging and Monitoring Checklist

- [ ] Centralized structured logging is enabled
- [ ] Correlation IDs are present in all error logs
- [ ] Sensitive fields are redacted/masked
- [ ] Alerting is configured for error-rate spikes and repeated fault codes
- [ ] Distinct dashboards exist for client errors vs server/dependency errors
- [ ] Dead-letter queues and retry exhaustion are monitored

## Common Anti-Patterns

- Returning raw exception messages to clients
- Catching and suppressing exceptions without telemetry
- Using the same generic 500 for every failure type
- Infinite retries on failing dependencies
- Logging full request bodies containing sensitive data
- No rollback/compensation for failed multi-step operations

## Minimal Incident Playbook (Error Spike)

1. Identify scope (routes/services/tenants/regions).
2. Triage by error class (client, dependency, system).
3. Mitigate quickly (feature flags, rate limits, circuit breaker tuning, rollback).
4. Preserve evidence (logs, traces, metrics, deployment diffs).
5. Communicate impact and ETA.
6. Add regression tests and update runbooks after resolution.

## Related Controls

- Input validation
- Access control and authorization
- Secure logging
- Resilience engineering (timeouts, retries, circuit breakers)
- Incident response and postmortems

