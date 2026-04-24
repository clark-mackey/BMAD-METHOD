# Anti-Patterns Library

Patterns spotted across projects. Reference during code reviews.

## Concurrency & Async
- SQLAlchemy sync session shared across async coroutines: NOT thread-safe, causes connection corruption, deadlocks, stale reads. Each coroutine needs isolated session.
- Check-then-act race conditions in shutdown handling: use atomic operations inside critical sections.
- Sequential fallback chains blocking on slow primaries: use asyncio.wait_for with race conditions between primary and fallback.
- Orchestrator deduplication race conditions: check-then-act between query and worker spawning allows duplicate work. Use atomic upsert or distributed locks.

## Resource Management
- Concurrency math errors on resource-constrained platforms: 20 concurrent HTTP requests = 80MB+ RAM minimum, exceeds Railway 512MB. Account for connection buffers.
- Test plans that ignore platform constraints: Railway 512MB + 20 concurrent HTTP = guaranteed OOM.
- Unbounded result accumulation: collecting all results in memory before write causes OOM on large domains (5K+ URLs). Stream in batches.
- Per-resource semaphores ignore global rate limits: 2 URLs × 5 concurrent = 10 simultaneous against 20 RPM budget.

## Error Handling
- Silent exception catching breaks user control: except Exception catches KeyboardInterrupt, making prompts uncancellable.
- Silent exception swallowing in concurrent gather: asyncio.gather with return_exceptions=True followed by continue silently drops failures.
- Pipeline restart on partial failure: outer retry discards ALL prior stage work when only last stage fails.
- Division by zero in success rate calculations: attempts can be 0 for new domains/tiers.

## API & External Services
- API response structure assumptions: json.loads followed by direct key access crashes on unexpected format. Always validate keys.
- Synchronous external API calls in user-facing code: 15s timeout blocks response delivery. Use async/background.
- HTTP Retry-After header parsing: headers can be seconds OR HTTP dates, not just integers.
- Database constraint misalignment with external API: CHECK constraint allowing only subset of API enum values causes 500 errors.

## Data & Metrics
- In-memory aggregation in resumable jobs: collecting from current session only loses historical data on resume. Query database for complete history.
- Percentile calculation without interpolation: int(len * 0.95) - 1 gives wrong percentile for small datasets.
- Mixed semantics in throughput metrics: success count / total time misleads when failures dominate. Separate metrics.
- Disconnected cost tracking: estimating proxy costs by attempt count ignores bandwidth-based charging.

## Scraping & Web
- Shared cookie cache in scraping swarms: single point of failure where one domain ban kills entire pool. Use per-worker cookie pools.
- Daily strategy updates for real-time arbitrage: auction sites change anti-bot hourly. Use event-driven updates.
- "Deterministic scraping" assumption: modern anti-bot uses behavioral analysis. Static tiers are easily fingerprinted.
- Single search pattern assumption: hardcoded search misses category pages, featured listings — 80%+ of inventory.
- Worker pool cascade failures: single URL failure doubling delay kills entire pool efficiency. Need per-URL isolation.

## Architecture
- Throwing away agent swarm parallelism for "async workers": loses massive parallel ephemeral advantage.
- Side effects mixed with business logic: scrape_url with HTTP + cookie mutations violates FP separation.
- Observability overengineering for small teams: 4-service monitoring deployment creates massive overhead for 1-2 devs.
- WhatsApp-as-UI disguised SaaS: recreating dashboard widgets in messaging format. True agent-first is pure text + links.

## WordPress / CMS
- Output buffering for selective content removal: buffering entire page to strip script tags is wasteful, conflicts with caching.
- Schema conflict detection during wp_head misses late-priority hooks: run detection at priority 999 or wp_footer.
- Full-page output buffering for selective JSON-LD removal: use tiered suppression (filter hooks → script_loader_tag → action tracking).
- Schema conflict detection on complex pages risks plugin conflicts. Use minimal test page with wp_installing constant.

## SSE / Streaming
- SSE parsing with naive UTF-8 decoding: chunk boundaries can split multi-byte sequences. Buffer bytes until complete character available.

## Webhook & Idempotency
- Webhook idempotency after business logic: insert idempotency record FIRST with flush, then process, then commit atomically.

## Corrections
- Medical schema: MedicalWebPage is ONLY for educational/reference content, NOT medical practice marketing. Use plain WebPage for practice homepages.

