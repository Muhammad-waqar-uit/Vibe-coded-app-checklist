# Vibe Coded Production Checklist

AI tools can get an app running in an afternoon. They rarely tell you what it needs to survive real users, real traffic and real attackers. This repo is a checklist of 111 things that usually get skipped, each with a concrete todo you can tick off before you call a vibe coded app "production ready".

The list of topics comes from the "Who's gonna tell vibe coders about...". The todos are written to be practical: read the topic, do the action, tick the box.

## How to use this

1. Copy this file into your project (or fork the repo).
2. Go section by section. Mark items `N/A` if they truly do not apply, with a one line reason.
3. Do sections 12 (Security) and 6 (Database performance) first. They hurt the most when ignored.
4. Re-run the list before every major launch.

Legend: `[ ]` todo, `[x]` done.

## Quick start priorities

If you only have one weekend, do these in order:

1. Secrets management, IAM, TLS, CORS, CSRF, SQL injection, XSS, SSRF (items 67 to 80)
2. Backups, failover and a tested restore (items 83 to 85)
3. Rate limiting, timeouts, retries with backoff (items 1, 11, 12, 13)
4. Database indexes, N+1 fixes, connection pooling (items 25 to 28)
5. Logging, metrics, alerts, health checks (items 56 to 62)
6. CI/CD with rollbacks (items 6, 55)

---

## 1. Traffic and API layer

- [ ] **1. Rate Limiting**: Limit requests per IP and per user on every public endpoint, with stricter limits on login, signup, password reset and anything that costs money.
- [ ] **2. Caching**: Cache expensive reads (in memory or Redis) with a clear TTL. Never cache user specific data under a shared key.
- [ ] **3. Load Balancing**: Run at least two instances behind a load balancer so one crash does not mean downtime.
- [ ] **4. Reverse Proxies**: Put Nginx, Caddy or a managed equivalent in front of the app for TLS, compression, request size limits and header cleanup.
- [ ] **5. API Gateways**: Centralize auth, throttling, routing and request logging at one entry point instead of re-implementing them in every service.

## 2. Build, ship and containers

- [ ] **6. CI/CD**: Run lint, type check, tests and a build on every push. Deploy only from the main branch through the pipeline, never from a laptop.
- [ ] **7. Docker**: Use a pinned base image, multi-stage build, non-root user and a `.dockerignore`. Keep secrets out of image layers.
- [ ] **8. Kubernetes**: Only adopt it if you need it. If you do, set resource requests and limits, replicas, pod disruption budgets and namespaces.
- [ ] **9. Service Discovery**: Do not hardcode service addresses. Use DNS based discovery or your platform's service names.

## 3. Resilience

- [ ] **10. Circuit Breakers**: Stop calling a failing dependency for a cool down period and return a fallback instead of piling on.
- [ ] **11. Timeouts**: Set explicit timeouts on every HTTP call, DB query and queue operation. The default is often "forever".
- [ ] **12. Retries**: Retry only safe, transient failures, with a maximum attempt count.
- [ ] **13. Exponential Backoff**: Add increasing delays plus random jitter between retries to avoid retry storms.
- [ ] **14. Idempotency**: Accept an idempotency key on payments, orders and any write that a client might repeat, so a retry never double charges or double creates.

## 4. Async work and messaging

- [ ] **15. Message Queues**: Move slow work (emails, image processing, webhooks, AI calls) off the request path and into a queue with workers.
- [ ] **16. Pub/Sub**: Use publish and subscribe when several consumers need the same event, so producers do not know about consumers.
- [ ] **17. Event-Driven Architecture**: Define event names and schemas, version them, and document who produces and who consumes each.
- [ ] **18. Distributed Transactions**: Avoid them where you can. If one business action spans services, decide up front how partial failure is handled.
- [ ] **19. Saga Pattern**: For multi step flows, write a compensating action for every step (refund, release stock, cancel booking).
- [ ] **20. Dead Letter Queues**: Send messages that keep failing to a DLQ, alert on its size, and have a way to inspect and replay them.
- [ ] **21. Cron Jobs**: Make scheduled jobs idempotent, locked so two instances do not run the same job, logged, and monitored for missed runs.

## 5. Realtime

- [ ] **22. WebSockets**: Authenticate on connect, add heartbeats, handle reconnects, cap connections per user and plan how to scale across instances.
- [ ] **23. Long Polling**: Use it as a fallback only. Set server timeouts and avoid holding a worker thread per client.
- [ ] **24. Server-Sent Events**: Good for one way streams. Handle reconnect with `Last-Event-ID` and watch proxy buffering.

## 6. Database performance

- [ ] **25. Database Indexing**: Index every column used in `WHERE`, `JOIN` and `ORDER BY` on large tables. Check unused and duplicate indexes too.
- [ ] **26. Query Optimization**: Run `EXPLAIN ANALYZE` on your slowest queries. Select only the columns you need and paginate everything.
- [ ] **27. N+1 Queries**: Find loops that run one query per row. Fix with joins, eager loading or batching (DataLoader).
- [ ] **28. Connection Pooling**: Use a pool with a sensible max size. Serverless setups need a pooler (PgBouncer or the provider's equivalent).
- [ ] **29. Read Replicas**: Send heavy reads and reports to a replica. Accept and handle replication lag.

## 7. Scaling and consistency of data

- [ ] **30. Sharding**: Leave it until you truly need it. If you do, pick the shard key carefully, since it is very hard to change later.
- [ ] **31. Partitioning**: Partition very large tables (usually by time) and automate dropping or archiving old partitions.
- [ ] **32. Replication**: Set up replication for durability, monitor lag, and know how promotion works.
- [ ] **33. Leader Election**: If you run singletons (schedulers, consumers), use a proper lock or consensus tool, not "whoever starts first".
- [ ] **34. CAP Theorem**: For each datastore, write down whether you favor consistency or availability during a network partition.
- [ ] **35. Eventual Consistency**: Identify where users may see stale data and design the UI and retries around it.

## 8. Concurrency and runtime health

- [ ] **36. Optimistic Locking**: Add a version column and fail updates when it changed, for records that many users edit.
- [ ] **37. Pessimistic Locking**: Use `SELECT ... FOR UPDATE` for short critical sections like inventory or balances. Keep transactions small.
- [ ] **38. Distributed Locks**: If you use Redis or similar locks, add expiry, unique owner tokens and fencing where correctness matters.
- [ ] **39. Race Conditions**: Review check then act code (balance checks, unique usernames, coupon redemption). Enforce with DB constraints and atomic updates.
- [ ] **40. Deadlocks**: Acquire locks in a consistent order, keep transactions short and retry on deadlock errors.
- [ ] **41. Memory Leaks**: Watch memory over time in production. Look for growing caches, listeners never removed and unclosed connections.
- [ ] **42. Garbage Collection**: Know your runtime's GC behavior. Set heap limits and watch pause times on hot services.
- [ ] **43. Thread Safety**: Avoid shared mutable state across requests, workers and async tasks. Use immutability or proper synchronization.

## 9. Scaling and delivery

- [ ] **44. Backpressure**: When consumers cannot keep up, slow or reject producers (bounded queues, 429, 503) instead of running out of memory.
- [ ] **45. Autoscaling**: Scale on a meaningful metric (CPU, queue depth, latency) with min and max limits and a cool down.
- [ ] **46. Horizontal Scaling**: Make the app stateless. Sessions, uploads and caches must live outside the instance.
- [ ] **47. Vertical Scaling**: Know your instance size limits and the point where scaling up stops being cheaper or possible.
- [ ] **48. CDN**: Serve static assets and media through a CDN with long cache headers and hashed file names.
- [ ] **49. Edge Caching**: Cache public, rarely changing responses at the edge. Set `Cache-Control` and `Vary` headers correctly.
- [ ] **50. Cache Invalidation**: Define how and when each cache is cleared on writes. Prefer short TTLs plus targeted purges over guessing.

## 10. Release strategy

- [ ] **51. Feature Flags**: Ship risky features behind flags so you can turn them off without a deploy. Remove old flags.
- [ ] **52. Blue-Green Deployment**: Keep two environments and switch traffic when the new one is healthy.
- [ ] **53. Canary Releases**: Send a small share of traffic to the new version first and watch errors before going wider.
- [ ] **54. Rolling Deployments**: Replace instances gradually with readiness checks, so there is no downtime.
- [ ] **55. Rollbacks**: Know the one command that returns to the last good version, and test it. Plan for database changes that cannot be undone.

## 11. Health and observability

- [ ] **56. Health Checks**: Expose `/health` that checks real dependencies, and wire it to your load balancer and uptime monitor.
- [ ] **57. Liveness and Readiness Probes**: Liveness says restart me, readiness says send me traffic. Do not mix them up.
- [ ] **58. Monitoring**: Track CPU, memory, disk, error rate and latency with dashboards you actually look at.
- [ ] **59. Logging**: Use structured JSON logs with request IDs and levels. Never log passwords, tokens or full card data.
- [ ] **60. Distributed Tracing**: Add trace IDs across services (OpenTelemetry) so one slow request can be followed end to end.
- [ ] **61. Metrics**: Export counters, gauges and histograms for business and technical events (signups, payments, queue depth).
- [ ] **62. Alerts**: Alert on symptoms users feel (errors, latency, downtime), route to a real person and prune noisy alerts.
- [ ] **63. SLOs**: Pick a few targets, such as 99.9% of requests succeed, and write them down.
- [ ] **64. SLIs**: Define the exact measurement behind each SLO (for example, successful responses divided by total responses).
- [ ] **65. Error Budgets**: Use the gap between 100% and your SLO to decide when to ship features and when to freeze and fix.
- [ ] **66. Observability**: Make sure logs, metrics and traces together let you answer "why is this broken" without redeploying.

## 12. Security

- [ ] **67. Secrets Management**: No secrets in git, front end bundles or AI chat prompts. Use a secrets manager or env vars from the platform. Rotate anything that ever leaked.
- [ ] **68. IAM**: Least privilege for every service and person. No wildcard permissions, no shared root accounts, MFA on.
- [ ] **69. OAuth**: Use a vetted library or provider. Validate `state`, use PKCE and restrict redirect URIs exactly.
- [ ] **70. JWT Rotation**: Short lived access tokens, refresh token rotation, a way to revoke, and a plan to rotate signing keys.
- [ ] **71. TLS**: HTTPS everywhere with auto renewing certificates, HSTS and no mixed content.
- [ ] **72. Encryption at Rest**: Encrypt databases, disks and backups. Hash passwords with Argon2 or bcrypt, never encrypt or store plain.
- [ ] **73. Encryption in Transit**: Use TLS for internal service and database connections too, not only the public edge.
- [ ] **74. WAF**: Put a web application firewall in front of public apps and turn on managed rule sets.
- [ ] **75. DDoS Protection**: Use a provider with DDoS mitigation, and make sure origin servers are not directly reachable.
- [ ] **76. CORS**: Allow only the exact origins you need. Never use `*` with credentials.
- [ ] **77. CSRF**: Use SameSite cookies and CSRF tokens on state changing requests that rely on cookie auth.
- [ ] **78. SQL Injection**: Use parameterized queries or a trusted ORM everywhere. Never build SQL with string concatenation.
- [ ] **79. XSS**: Escape output, avoid `dangerouslySetInnerHTML` and `innerHTML`, and set a Content Security Policy.
- [ ] **80. SSRF**: When your server fetches user supplied URLs, block internal IP ranges and cloud metadata addresses, and use allowlists.

## 13. Changing data safely

- [ ] **81. Database Migrations**: Use a migration tool, commit every migration and run them in CI or deploy. Never edit production by hand.
- [ ] **82. Schema Versioning**: Make changes backward compatible (add, backfill, switch, then remove) so old and new code can both run during deploys.

## 14. Reliability and cost

- [ ] **83. Disaster Recovery**: Write a short runbook: what is lost, how long you can be down (RTO), how much data you can lose (RPO), and who does what.
- [ ] **84. Backups**: Automate backups, store them in a separate account or region, and test a full restore on a schedule.
- [ ] **85. Failover**: Have a standby database or region and know how to switch. Practice it.
- [ ] **86. Multi-Region Deployment**: Only if your uptime needs justify it. Plan for data replication and latency.
- [ ] **87. Chaos Engineering**: Once the basics work, kill an instance or block a dependency on purpose in a safe window and see what breaks.
- [ ] **88. Cost Optimization**: Set budgets and billing alerts, tag resources, delete idle ones and check what your AI API usage costs per user.
- [ ] **89. Cold Starts**: For serverless, measure startup time. Trim bundle size, reuse connections and use provisioned concurrency for critical paths.
- [ ] **90. Serverless Limits**: Check timeout, memory, payload size and concurrency limits against your real workloads.

## 15. Performance and networking

- [ ] **91. Latency**: Measure real user latency per endpoint and set a target.
- [ ] **92. Throughput**: Load test to find requests per second at which the app degrades, before users find it for you.
- [ ] **93. P99 Latency**: Track the 99th percentile, not just the average. Averages hide pain.
- [ ] **94. Tail Latency**: Investigate the slowest 1%. Look at GC pauses, cold caches, slow queries and retries stacking up.
- [ ] **95. Network Partitions**: Decide what each service does when it cannot reach another (fail, queue or serve stale) and test it.
- [ ] **96. Clock Skew**: Do not rely on timestamps from different machines for ordering. Use NTP, DB generated time or logical clocks.
- [ ] **97. DNS**: Know your TTLs, use a reliable provider, set up records for email (SPF, DKIM, DMARC) and protect the domain registrar account.
- [ ] **98. TCP vs UDP**: Know which one your use case needs. Default to TCP unless you can explain why not.
- [ ] **99. HTTP/2 and HTTP/3**: Enable them at your proxy or CDN for better multiplexing and connection speed.
- [ ] **100. gRPC**: Consider it for internal service to service calls. Define protobuf contracts and handle deadlines and retries.
- [ ] **101. Webhooks**: Verify signatures, respond fast, process async, handle duplicates and retry failed deliveries with backoff.

## 16. Versioning, infrastructure and operations

- [ ] **102. API Versioning**: Version public APIs (`/v1`), document breaking changes and set a deprecation policy.
- [ ] **103. Semantic Versioning**: Use `MAJOR.MINOR.PATCH` and keep a changelog.
- [ ] **104. Infrastructure as Code**: Define infrastructure in version controlled files so environments can be rebuilt and reviewed.
- [ ] **105. Terraform**: Use remote state with locking, separate environments and review `plan` output before every apply.
- [ ] **106. Helm Charts**: If on Kubernetes, template your manifests, pin chart versions and keep values per environment.
- [ ] **107. Build Caching**: Cache dependencies and build layers in CI to keep pipelines fast.
- [ ] **108. Dependency Hell**: Commit lockfiles, pin versions, run `npm audit` or equivalent and review packages AI suggests. Some of them do not exist or are malicious lookalikes.
- [ ] **109. Production Incidents**: Write a simple incident process: who is in charge, where people talk, how you tell users.
- [ ] **110. On-Call**: Decide who gets paged, when, and what they can do. Document common fixes.
- [ ] **111. Postmortems**: After every incident, write a blameless summary: timeline, root cause, what to change. Track the follow ups to completion.

---

## Progress tracker

Copy this block into your project and update it as you go.

| Section | Items | Done |
| --- | --- | --- |
| 1. Traffic and API layer | 1 to 5 | 0 / 5 |
| 2. Build, ship and containers | 6 to 9 | 0 / 4 |
| 3. Resilience | 10 to 14 | 0 / 5 |
| 4. Async work and messaging | 15 to 21 | 0 / 7 |
| 5. Realtime | 22 to 24 | 0 / 3 |
| 6. Database performance | 25 to 29 | 0 / 5 |
| 7. Scaling and consistency of data | 30 to 35 | 0 / 6 |
| 8. Concurrency and runtime health | 36 to 43 | 0 / 8 |
| 9. Scaling and delivery | 44 to 50 | 0 / 7 |
| 10. Release strategy | 51 to 55 | 0 / 5 |
| 11. Health and observability | 56 to 66 | 0 / 11 |
| 12. Security | 67 to 80 | 0 / 14 |
| 13. Changing data safely | 81 to 82 | 0 / 2 |
| 14. Reliability and cost | 83 to 90 | 0 / 8 |
| 15. Performance and networking | 91 to 101 | 0 / 11 |
| 16. Versioning, infrastructure and operations | 102 to 111 | 0 / 10 |

## A note on honesty

Not every app needs all 111 items. A weekend side project does not need Kubernetes or multi-region failover. The goal is that you know which items you are skipping and why, instead of finding out in production.

## Contributing

Pull requests are welcome. Good contributions: clearer todos, links to good guides for an item, stack specific notes (Next.js, Supabase, Firebase, Solidity and so on) and missing topics. See [CONTRIBUTING.md](CONTRIBUTING.md).

## License

MIT, see [LICENSE](LICENSE).
