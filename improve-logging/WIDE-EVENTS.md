# Wide Events Reference

## Core Mental Model

Log **what happened to this unit of work**, not a diary of what the code executed. Emit **one wide event per unit of work per service hop** at completion.

## Anatomy of a Wide Event

Adapt field names to the codebase's existing naming convention (e.g. `snake_case` or `camelCase`), staying consistent across every service:

- **Envelope & infrastructure**: `timestamp`, `service`, `version`, `deployment_id`, `region`/`env`
- **Correlation (high cardinality)**: `request_id`, `trace_id`, `span_id`, `job_id`
- **Boundary & outcome**: `method`/`operation`, `path`/`route`, `status_code`, `outcome` (`success` | `error`), `duration_ms`
- **Actor / business context (high dimensionality)**: `user` (`id`, `tier`/`plan`, `org_id`, …), domain entity summary (e.g. `order`, `cart`, `document` counts/amounts/IDs), active `feature_flags`
- **Dependencies**: downstream call latencies, cache hit/miss, DB query count/duration, retry `attempt`
- **Structured error (on failure)**: `error.type`, `error.code`, `error.message`, `error.retriable`, upstream/provider error code

## OpenTelemetry Note

If the project already uses OpenTelemetry, enrich the root span of each service hop with these same high-cardinality and business attributes rather than adding a parallel logging pipeline. OTel is the transport; wide-event attributes provide the observability.

## Tail Sampling Rules

When volume requires sampling, evaluate **after** the unit of work finishes:
1. Keep **100%** of errors (`status_code >= 500`, unhandled exceptions, domain failures).
2. Keep **100%** of slow executions (above p99 latency threshold).
3. Keep **100%** of targeted traffic (VIP/enterprise accounts, internal debug accounts, rollout feature flags).
4. Randomly sample the remaining fast, successful traffic (e.g., 1–5%).

## Safety & Redaction

Never emit secrets or raw sensitive data: passwords, API keys, auth headers, session tokens, full credit card numbers, or unredacted PII not needed for debugging. Enforce redaction centrally in the logger/serializer rather than relying on every call site.
