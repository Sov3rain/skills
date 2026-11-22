---
name: improve-logging
description: Audit a codebase for missing or poor logging, then refactor it toward wide events, high-cardinality context, and tail sampling.
disable-model-invocation: true
---

# Improve Logging

Transform scattered, low-context logs into **wide events** (canonical log lines): one structured, high-cardinality, high-dimensionality event emitted per unit of work (HTTP request, RPC call, background job, queue message, webhook) per service hop.

Stay stack-agnostic: adapt to the codebase's existing language, runtime, structured logger, and request/execution context propagation mechanism.

Read [WIDE-EVENTS.md](WIDE-EVENTS.md) for the target event anatomy, enrichment rules, tail-sampling criteria, and PII guardrails.

## Process

### 1. Scope & Explore

Decide where to focus first:
- If the user named a scope (service, module, endpoint, worker), target that area and its shared logging infrastructure.
- Otherwise, check recent `git log --oneline` hot spots and map all execution boundaries (HTTP/GraphQL/RPC entry points, background jobs, queue consumers, cron tasks, webhooks, outbound client wrappers) plus existing logging/telemetry setup.

Spawn a sub-agent with a fresh context window to audit the target scope and return an inventory covering every file and boundary with:

1. **Missing instrumentation**: execution boundaries, critical business flows, or error handlers that emit no structured event at all.
2. **Log chaos (scattered logs)**: single requests or jobs emitting multiple step-by-step log lines ("starting X", "fetched Y", "done Z") instead of accumulating into one wide event.
3. **Unstructured / inconsistent logs**: plain-string logs (`console.log`, `print`, interpolated strings) or the same identifier logged under inconsistent keys (`userId`, `user_id`, `uid`).
4. **Low cardinality / missing correlation**: events lacking stable identifiers (`request_id`, `trace_id`, `user_id`, entity IDs, `service`, `version`, `deployment_id`).
5. **Low dimensionality (missing business context)**: technical-only logs or bare OpenTelemetry spans that omit domain context (account tier, feature flags, item/cart/resource attributes, retry attempts, downstream latency and error codes).
6. **Vague or swallowed errors**: catch blocks logging only a generic message or discarding error type, code, retriability, or upstream cause.
7. **Sampling gaps**: high-volume endpoints with no sampling strategy, or naive random head-sampling that can drop errors and slow requests.

**Done when**: every boundary and logging hotspot in scope is accounted for with exact file paths, line ranges, and category.

### 2. Present Findings & Proposed Strategy

Present a concise, prioritized Markdown report directly in the chat:
- **Current state**: existing logger/telemetry libraries, context propagation mechanism, and top gaps found.
- **Findings**: grouped by severity (`Critical` / `High` / `Medium`), with clickable file references and concrete before/after examples.
- **Proposed strategy**:
  - The shared **wide event** mechanism to introduce or upgrade (middleware / wrapper / context accumulator emitted once in teardown/`finally`).
  - The base envelope + domain context fields to capture.
  - PII / secret redaction boundaries.
  - Whether **tail sampling** is warranted for the project's scale, and the proposed keep rules.

Wait for explicit user approval of the scope and strategy before editing any code.

### 3. Implement the Logging Strategy

Once approved, execute in order:

1. **Foundation first**: implement or refactor the central wide-event accumulator and emission hook (middleware, decorator, interceptor, or job wrapper) using the stack's idiomatic context propagation. Emit once on completion (`finally`/defer/teardown) with duration, outcome, status, and deployment metadata.
2. **Enrich boundaries & handlers**: attach high-cardinality IDs and business context as the unit of work progresses; capture structured error details (`type`, `code`, `message`, `retriable`, upstream cause) on failure.
3. **Eliminate log chaos**: replace redundant step-by-step logs with fields on the active wide event. Keep standalone logs only for lifecycle events outside a unit of work (startup, shutdown, fatal crashes).
4. **Add tail sampling (if in scope)**: decide after completion — keep 100% of errors, slow requests above threshold, and flagged/VIP traffic; sample nominal traffic.
5. **Enforce redaction**: ensure tokens, passwords, raw payment data, and sensitive PII are redacted or omitted centrally.

### 4. Verify

- Run the project's existing test suite, typechecker, and linter.
- Add or update tests verifying that:
  - A single wide event is emitted per request/job on both success and failure.
  - Expected correlation IDs, business context, and structured error fields are present.
  - Redacted fields never appear in emitted events.
- Summarize the changes made, the resulting wide-event schema, and any remaining follow-ups.
