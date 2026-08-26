# Agency Development Cycle Standard

## Purpose

Agency uses one development cycle for local work, Docker runtime validation,
observability, and CI. The cycle must reduce rediscovery and manual
maintenance; it must not become a parallel process or a gatekeeping ritual.

## Owners

- `scripts/` owns developer commands and runtime orchestration.
- `config/` owns durable LiteLLM and Docker runtime configuration.
- `.github/` owns hosted automation.
- `tests/` owns deterministic behavior checks.
- `docs/` owns operating instructions and `_ops/` owns records of completed
  work, decisions, and active plans.

## Verification Ladder

1. Run the narrowest test, lint, or contract that can falsify the change.
2. Run the normal local integrity command from `scripts/` for implementation
   changes.
3. Run the foundation command from `scripts/` for gateway, Docker, surface,
   model, or observability changes. It validates host chat/image/audio and the
   Docker Admin image, synchronizes declared surfaces, emits observability, and
   restores any pre-existing Admin state after the check. If it starts Docker
   Desktop, it shuts it down again during cleanup.
4. Run pre-commit before publishing when it covers the changed surface.
5. CI must run the same deterministic checks and build the configured Docker
   runtime image without provider inference.

Do not add a new checker when one of these steps already proves the behavior.
Expand the owning command instead of creating a parallel script or daemon.

## Docker

Docker configuration is a versioned runtime contract. Local foundation work
and CI boot the same configuration and image through a free health endpoint. A
developer should not need to remember a second build, migration, or
synchronization ritual after editing runtime code or configuration.

The full cycle may use the free smoke model and observability delivery. Use
targeted live single-model status probes when a specific provider/model path
needs real verification; do not gate normal development on broad live matrices.

## Observability

Full trace context and evidence dumps are intentional for Agency. The
development cycle records them under the transient output owner and routes
durable summaries to the appropriate record under `_ops/` when they change a
decision or plan. Do not introduce redaction, sampling, or retention limits as
a default response to generic concerns.
