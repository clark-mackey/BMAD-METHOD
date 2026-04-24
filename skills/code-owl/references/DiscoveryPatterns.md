# Discovery & Multi-Agent Patterns

## Discovery Pipeline Patterns
- Stage 1 needs entity-type checklists, not blind discovery
- HTML snippet targeting prevents token waste (50k → 5k per entity)
- Validation loops must be pipeline-aware, not legacy single-call
- Discovery architecture is correct, implementation needs structure
- 6→12+ entity discovery via structured hunting (medical, business, media, interactive categories)
- Legacy validation collision: disable retries or make pipeline-aware
- Cost optimization: snippet targeting + cheaper models for simple entities

## Multi-Agent Reliability Patterns
- Exponential backoff retry (2^attempt + jitter) for inter-agent calls
- Circuit breakers prevent cascading failures (3 failures → 60s open)
- Message delivery confirmation with correlation IDs for tracing
- Graceful degradation to cached responses when specialists fail
- Dynamic timeouts based on routing complexity (30s→180s for multi-agent)
- Dead letter queues for failed message recovery
- Proactive agent health monitoring before routing
- No global timeouts for multi-step workflows — they kill valid operations

## Circuit Breaker Details
- HalfOpen → Open should preserve accumulated failure count
- Environment variable overrides need range validation
- HTTP Retry-After can be seconds OR HTTP dates, not just integers
- Jitter calculations: clamp to minimum viable delay (avoid negative)

## Pipeline Anti-Patterns
- Per-resource semaphores ignore global rate limits (2 URLs × 5 concurrent = 10 against 20 RPM budget)
- Silent exception swallowing in concurrent gather: surface and count all failure types
- Pipeline restart on partial failure: retry only the failed stage, not the entire pipeline
- Multi-call prompt conflicts: base prompts override page-specific constraints

