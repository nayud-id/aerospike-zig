# Aerospike–Zig Execution Plan and Checklist

## 1) Points to Note Before Executing Each Task

- Security above all:
  - Never log or print secrets or credentials. Redact all sensitive values by default.
  - Store secrets in a single, safest location (encrypted file or secure vault); support environment variables as a first-class source with a strict schema and validation.
  - Zeroize sensitive memory after use; avoid long-lived in-memory copies; prefer immutable views and scoped lifetimes.
  - Enforce defensive defaults, input validation, bounds checks, and deny-by-default behaviors.
- Pure Zig only:
  - No C-family code, no FFI. Use Zig 0.15.1 standard library and native facilities. Add external dependencies only if absolutely necessary and keep them minimal.
- CE/EE compatibility:
  - Design for identical behavior on Community and Enterprise where possible. Where not possible, degrade gracefully and document a precise EE-only list.
- Configuration coverage and defaults:
  - Expose every Aerospike configuration option with sensible production defaults.
  - No hard-coded values; all tunables are read from a config file, environment, or secrets store with clear precedence.
- Production defaults target:
  - Aerospike 8.1.0 oriented for in-memory with persistence. Defaults reflect large-scale production needs; consumers can override every setting.
- Modular design and file-size discipline:
  - MVC-inspired layout to keep roles clear and testable. Keep each file ≤ 300 lines; split into submodules when needed.
  - Compose small functions; avoid deeply nested logic; keep code DRY.
- Observability and error handling:
  - Structured logging (context + severity). Rich errors with classification and user guidance. No sensitive data in logs.
- High availability and zero downtime:
  - Built-in failover, load balancing, connection pooling, exponential backoff, health checks, rolling (hot) reconfiguration.
- Testing strategy:
  - Unit + fuzz tests for parsers and critical code; deterministic concurrency tests; integration harness for CE and EE.
- Documentation:
  - Every function has multi-line, human-readable docs with purpose, parameters, examples, and returns. Mark incomplete items with TODO.
- Packaging:
  - Ship as a Zig package installable as a dependency; clean build.zig and build.zig.zon; no global side effects.

Proposed directory layout (technical/MVC oriented):
- model/
  - config/ (typed config models, defaults registry)
  - cluster/ (node/partition state models)
  - credentials/ (schemas for secrets, env var names)
- controller/
  - config/ (load, merge, validate, hot-reload)
  - cluster/ (discovery, health, partition map orchestration)
  - failover/ (circuit breaker, retries, backoff)
  - loadbalancer/ (policies: RR, least-conns, latency-aware)
  - secrets/ (vault/env/file providers; memory hygiene)
- view/
  - logging/ (sinks, formats, redaction)
  - metrics/ (counters, histograms, timers; exporter interfaces)
- common/
  - constants/, errors/, types/, util/

## 2) Task Execution List (ordered, granular)

1. [ ] Project guardrails and metadata
   - [x] Pin Zig 0.15.1 in documentation and add a preflight version assertion.
   - [ ] Verify build.zig.zon metadata; set minimum_zig_version=0.15.1.
   - [ ] Add security coding standards and logging redaction policy notes.

2. [ ] Create directory layout and scaffolding
   - [ ] Create model/, controller/, view/, common/ with subfolders listed above.
   - [ ] Add minimal module files with top-of-file documentation and TODOs.

3. [ ] Security and secrets foundation
   - [ ] Define complete credentials schema (env var names, file layout, vault keys).
   - [ ] Implement secrets providers: Environment, EncryptedFile (with KMS/vault hook), and NoSecret (for local dev) with strict redaction.
   - [ ] Implement memory hygiene helpers (zeroize, wipe-on-drop patterns).
   - [ ] Add validation and precedence rules; unit and fuzz tests for parsers.

4. [ ] Configuration system (complete Aerospike coverage)
   - [ ] Define typed config models covering all Aerospike options and contexts.
   - [ ] Build a defaults registry tuned for production (in-memory with persistence) with exhaustive documentation.
   - [ ] Implement loader: JSON file parser (std), env overrides, secret references; deterministic merge order.
   - [ ] Implement validation (types, ranges, cross-field constraints).
   - [ ] Add hot-reloadable config with safe, atomic apply and rollback.
   - [ ] Unit + fuzz tests for parsing/merging/validation; property tests for invariants.

5. [ ] CE/EE parity plan
   - [ ] Research EE-only features; produce an enumerated list and compatibility strategy.
   - [ ] Implement capability flags; feature gating; graceful degradation on CE.
   - [ ] Document behavior parity and any unavoidable EE-only gaps.

6. [ ] Cluster management and health
   - [ ] Seed node loading and discovery strategy abstraction.
   - [ ] Health checks, node scoring, exponential backoff, jitter; quarantine logic.
   - [ ] Partition/routing map abstraction (pluggable, future server-driven sync).
   - [ ] Concurrency-safe cluster state with readers/writers; deterministic tests.

7. [ ] Connection pooling and TLS
   - [ ] Design connection pool with fairness and backpressure; timeouts and deadlines.
   - [ ] TLS configuration model; secure defaults; cert rotation hooks.
   - [ ] Failure injection tests; leak checks.

8. [ ] Load balancing and routing policies
   - [ ] Implement policies (round-robin, least-connections, latency-aware).
   - [ ] Policy selection via config; live switching during hot-reload.
   - [ ] Performance tests under contention; starvation prevention checks.

9. [ ] Failover and zero-downtime operations
   - [ ] Circuit breaker and retry policy with idempotency hints.
   - [ ] Rolling reconfiguration with atomic apply/rollback and state handoff.
   - [ ] Soak tests and chaos scenarios.

10. [ ] Observability (logging and metrics)
    - [ ] Structured logging with contexts; sinks (stderr/file); redaction filters.
    - [ ] Metrics interfaces and in-memory exporter; hooks for external exporters.
    - [ ] Log/metric schemas; documentation and examples.

11. [ ] Error taxonomy and handling
    - [ ] Centralized error types and classification (transient vs permanent).
    - [ ] User-facing guidance; remediation hints; mapping to exit codes.

12. [ ] Packaging and consumer experience
    - [ ] Ensure package usable as dependency; expose modules cleanly; no side effects.
    - [ ] Update build.zig to export only safe, documented surfaces.
    - [ ] Versioning (semver) and changelog scaffolding.

13. [ ] Testing and quality gates
    - [ ] Unit, fuzz, concurrency, and integration test suites.
    - [ ] CI scripts (lint/format/test/build) and minimal coverage tracking.

14. [ ] Documentation pass
    - [ ] Ensure every function has multi-line human-readable doc comments with examples.
    - [ ] Add configuration and credential schema reference docs.

15. [ ] Security review and final hardening
    - [ ] Threat model, dependency review, memory-safety pass, redaction audit.
    - [ ] Prepare EE-only capabilities document and release notes.

Acceptance on each step requires: passing tests, zero panics, no secret leakage, and compliance with file-size and documentation rules.