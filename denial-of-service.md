# Denial of Service (DoS)

## Applies To

- Internet-facing applications and APIs
- Internal services and microservices
- Message-driven and batch systems
- Cloud and on-prem deployments

## Summary

A Denial of Service (DoS) condition happens when an application, service, or supporting
infrastructure is made unavailable or severely degraded for legitimate users.

DoS risk is not only network flooding. Many outages are caused by application-layer abuse:
expensive queries, unbounded parsing, hot-path lock contention, queue starvation, and
resource exhaustion (CPU, memory, threads, connections, disk, I/O, or downstream quotas).

## Common DoS Attack Paths

- Request floods against expensive endpoints
- Resource exhaustion via large payloads or deep object graphs
- Slow request attacks (slowloris-style behavior)
- Unbounded regex or parsing (e.g., ReDoS, XML/JSON abuse)
- Abuse of search/sort/filter operations without limits
- Queue/topic flooding and consumer lag amplification
- Cache bypass causing repeated expensive computations
- Chained downstream timeouts causing cascading failure

## Security Objectives

1. Keep critical services available under stress.
2. Isolate failures to prevent cascading outages.
3. Detect abuse quickly and respond automatically where possible.
4. Recover fast with clear operational runbooks.

## Core Design Principles

1. **Bound everything.** Set strict limits on payload size, request rate, object depth,
   collection size, query complexity, and execution time.
2. **Fail fast and cheaply.** Reject abusive traffic early at edge/API gateway.
3. **Protect critical paths first.** Prioritize auth, checkout, and core transaction flows.
4. **Degrade gracefully.** Shed non-critical work under load.
5. **Isolate dependencies.** Use timeouts, retries with jitter, and circuit breakers.
6. **Assume partial failure.** Design for downstream slowness and backpressure.

## Preventive Controls

### Edge and Network Controls

- Rate limiting by IP/user/token/tenant
- WAF and bot management rules for known abusive patterns
- DDoS protection service and anycast/CDN where appropriate
- Connection limits, SYN protection, and request timeout tuning

### Application Controls

- Per-endpoint request budgets (RPS/concurrency)
- Authentication and authorization before expensive operations
- Input validation with size/depth/format constraints
- Pagination, max page sizes, and bounded filters/sorts
- Query cost controls and statement timeouts
- Async/offline processing for expensive workloads
- Cache hot responses and negative lookups where safe

### Platform and Data Controls

- Thread/worker pool isolation by workload class
- DB connection pool limits and query timeout policies
- Queue backlog thresholds and dead-letter handling
- Resource quotas (CPU/memory) and autoscaling guardrails

## Abuse-Resistant Implementation Checklist

- [ ] Global and per-route rate limits are defined and tested
- [ ] Maximum request body size is enforced
- [ ] JSON/XML/object depth and field count limits are enforced
- [ ] Regex patterns are bounded and reviewed for catastrophic backtracking
- [ ] DB queries have timeout and result-size limits
- [ ] Endpoint-specific concurrency caps exist for expensive handlers
- [ ] Critical dependencies use circuit breakers and bounded retries
- [ ] Background jobs are idempotent and queue consumers apply backpressure
- [ ] Non-critical features can be disabled via feature flag

## Monitoring and Detection

Track at minimum:

- Request rate, error rate, and latency percentiles (p50/p95/p99)
- Saturation metrics (CPU, memory, thread pool, connection pool, queue depth)
- Rejection metrics (rate-limit hits, WAF blocks, timeout counts)
- Dependency health (timeouts, circuit-open counts, retry storms)

Alert on:

- Sudden RPS spikes outside historical bands
- Sustained p99 latency growth
- Queue lag/backlog growth beyond SLO thresholds
- Elevated 429/503/504 patterns

## Testing Strategy

1. Run load tests to expected and peak traffic.
2. Run stress tests beyond peak to identify breaking points.
3. Run soak tests for memory/resource leak detection.
4. Run chaos/failure tests for dependency timeout and retry behavior.
5. Include abuse cases: oversized payloads, high-cardinality filters, expensive search terms.

## Incident Response (DoS)

1. Confirm impact scope (which routes, regions, tenants).
2. Activate mitigation controls:
   - tighten rate limits
   - block abusive signatures
   - disable non-critical features
   - raise edge protections
3. Protect data plane:
   - reduce expensive query paths
   - increase timeouts/circuit isolation only where safe
   - preserve critical workflows
4. Communicate status and user impact.
5. Capture evidence (traffic patterns, top offenders, bottlenecks).
6. Perform post-incident hardening and regression testing.

## Common Anti-Patterns

- Unlimited pagination or unbounded search
- Retry storms without jitter/backoff
- Shared pools across critical and non-critical workloads
- Accepting giant payloads by default
- No timeout on outbound calls
- Logging every failed request synchronously under attack

## Related Controls

- Input validation
- Secure error handling
- Caching strategy
- Capacity planning and SLO engineering
- WAF and bot defense
- Observability and incident management

