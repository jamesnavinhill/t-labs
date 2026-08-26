# How Agency keeps docs / contracts current

## Principle

**Official first.** Prefer vendor-generated API truth + short Agency ops docs over
hand-maintained OpenAPI clones that drift.

## Recommended split

| Layer | Tool / source | What it owns |
| --- | --- | --- |
| **Live API surface (chat)** | LiteLLM Swagger at `http://127.0.0.1:8787/` + [LiteLLM docs](https://docs.litellm.ai) | Paths, request/response shapes, admin CRUD |
| **Agency ops contracts** | Markdown under `docs/` (this repo) | How *we* run: ports, env, free vs live checks, CF/Neon defects |
| **Provider defects** | `docs/providers/*` | Upstream quirks (not LiteLLM bugs) |
| **Vendor link index** | `_ops/reference/vendor-resources.md` | Canonical external URLs |
| **Roadmaps** | `_ops/roadmaps/*` while active → `_ops/_legacy/roadmaps/*` when done | Execution plans only |

## Do we need a doc platform?

| Option | When | Notes |
| --- | --- | --- |
| **Markdown in-repo (current)** | Default | Matches `docs/standards/*`; greppable; PR-reviewed |
| **Mintlify / Docusaurus** | Public site later | Nice site, extra publish pipeline — not required for local Agency |
| **Redocly / Spectral on OpenAPI** | If we publish a *stable* public API | Pull OpenAPI from LiteLLM export when frozen; lint contracts |
| **Stoplight / ReadMe** | Multi-team product portal | Overkill for single-operator gateway |

**Recommendation for now:** stay markdown-in-repo + link LiteLLM Swagger for
endpoint truth. Add Redocly only if Agency becomes a multi-consumer product API
with a versioned OpenAPI artifact.

## What agents should update

After foundation changes, update the owning operator or feature documentation
under `docs/`, environment-variable names in the root template, the root quick
start only when entry points change, and the active record under `_ops/`.
Archive finished plans beneath `_ops/_legacy/` after their durable rules have
been promoted.

Do **not** duplicate full LiteLLM endpoint catalogs in Agency markdown.
