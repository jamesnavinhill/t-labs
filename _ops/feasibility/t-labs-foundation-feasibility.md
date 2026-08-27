# t-labs Foundation Feasibility Report — Transformer Lab on Our Compute, Repo Shape, Vendors

- **Date**: 2026-08-26
- **Status**: Draft for discussion (research + audit complete; no spend made)
- **Request**: From `t-labs/_ops/user-notes/outline.md` — research and audit official/up-to-date sources for all vendors and tech in the project; decide repo structure; decide compute provider and storage for standing up Transformer Lab; map credits; plan skills, standards, AGENTS.md, dev cycle, observability.
- **Source scope**: Local repo walk (t-labs, liquid-primus/tidepool, references/transformerlab-app + transformerlab-examples, vendored docs under docs/t-labs/) + official/current external sources (docs.transformerlab.ai, docs.skypilot.ai, modal.com, aws.amazon.com terms, developers.cloudflare.com, galaxycloud.app, oracle.com, cloud.google.com, official GitHub repos). No live credential/console reads performed; every external fact below is dated and re-verifiable.
- **Owner**: james@jami.studio (+ agent)

---

## 1. Executive Summary

Everything in the outline is feasible. Transformer Lab is a real, actively-developed open-source research platform (MIT, `transformerlab-app` upstream) that runs as a control plane over compute providers — including **SkyPilot**. For this project the cleanest shape is:

- **Repo**: keep `t-labs` as the system repo and `projects/` as a sibling master-project parent — **not nested**. Use separate repos per project (liquid-primus already is its own repo).
- **Compute provider for Transformer Lab**: **SkyPilot** as the single provider-agnostic layer over our multi-cloud credit pool (AWS + GCP + Oracle + others). AWS/GCP *native* will stay viable but are explicitly **beta** in TFL.
- **Storage backend**: **AWS S3 first** (or GCS), using the fsspec-backed storage — credits cover it; avoid the AWS *Marketplace* carve-out.
- **Credits map**: Modal $120/mo for interactive/dev compute; AWS $800 for primary training (eligible services only); GCP $300 trial + Always Free for bursts/secondary; Cloudflare for edge inference/Workers AI (computational limit, not GPU); Galaxy $600 for app/tooling hosting (no GPU); Oracle Always-Free now **CPU-only (2 OCPU)** — not a GPU pool.
- **Observability**: stay OTel-native; add **self-hosted Langfuse (MIT)** in parity with the existing tracing stack.
- **Skills**: use `npx skills` **project-scoped** for vendor skills (AWS, GCP, Cloudflare, Langfuse, SkyPilot agent skill, etc.), matching the user's standing preference.
- **Next**: after decisions, refresh `docs/standards/`, write `AGENTS.md`, set up dev cycle/automation/changelog, and stand up TFL.

No compute spend is proposed here; nothing is burned by this report.

---

## 2. Question Being Answered

Turn the outline into concrete decisions:

1. What repo shape keeps `t-labs` (system) and many `projects/` committed cleanly?
2. Which compute provider should Transformer Lab use, given the credits we hold?
3. Which storage backend for TFL shared storage?
4. What is the actionable credits map across Modal / AWS / Cloudflare / Galaxy / Oracle / GCP, and what promotions/free tiers should we use?
5. What observability path gives parity with the existing tracing system?
6. Which vendor skills to install (project-scoped), per the plan?

---

## 3. Source Scope And Method

**Local sources read**
- `t-labs/_ops/user-notes/outline.md`, `t-labs/README.md`, `.env.example`, `docs/standards/*` (all six standards), `docs/t-labs/**` (vendored TFL docs: overview, concepts, setup incl. install/skypilot/configure-{aws,gcp,runpod,skypilot}/advanced-install/s3-alts, operations, examples), `.changelog/` (empty), `.gitignore` (only `.env`), git state (remotes, branches, reflog).
- `liquid-primus/readme.md` + `tidepool/` (overview.md, budget.json, operator-notes.md, stages structure).
- `references/transformerlab-app/` (upstream clone; `lab-sdk/src/lab/lab_facade.py` — verified `lab.init/get_config/log/update_progress/save_checkpoint/finish(message=, score=dict)/error(message=)` signatures; `CLAUDE.md`/`AGENTS.md`; CLI provider code via GitHub) and `references/transformerlab-examples/` (`unsloth-llm-train/task.yaml` + `train.py`).

**Official / external sources checked (2026-08-26)**
- Transformer Lab: docs.transformerlab.ai install page; transformerlab.ai/for-teams/install; github.com/transformerlab/transformerlab-app (README, AGENTS, lab-sdk, CLI COMMANDS.md).
- SkyPilot: docs.skypilot.ai (installation, API server deploy, cloud list); github.com/skypilot-org/skypilot (README, releases).
- Modal: modal.com/pricing; secondary 2026 pricing tables (flagged as approximations; verify live).
- AWS: aws.amazon.com/awscredits Promotional Credit Terms & Conditions (Dec 16, 2024); AWS re:Post on Marketplace; aws.amazon.com/activate/terms.
- Cloudflare: developers.cloudflare.com/workers-ai/platform/pricing; R2 limits.
- Galaxy: galaxycloud.app (Savings Plan, product pages); docs.galaxycloud.app python apps page.
- Oracle: OCI Always-Free documentation updates (Jun 15, 2026 change; Aug 18, 2026 enforcement), OCI GPU pricing, egress policy.
- GCP: cloud.google.com free tier + Vertex AI free tier/trial docs; community 2026 writeups (flag as context).
- Langfuse: langfuse.com + community; ClickHouse acquisition (Jan 2026), MIT, Docker Compose self-host, OTel ingestion.

**Sources intentionally not checked**: individual cloud consoles (no credentials read); live marketplace listing terms; vendor contract documents beyond the credit terms above. Those are the next-step verification items.

---

## 4. Current Project State

- **t-labs** is a committed repo (`jamesnavinhill/t-labs`, main @ `5edc147`) with:
  - `docs/standards/` — six Agency-style standards (dev-docs, development-cycle, docs-tooling, planning-style, repository-layout, report-style). They reference an "Agency" context, not yet t-labs.
  - `docs/architecture/`, `docs/operations/`, `docs/runbooks/` — all **empty**.
  - `docs/t-labs/` — vendored Transformer Lab docs (useful but the source-of-truth for TFL is upstream; mark as reference snapshot).
  - `.changelog/changelog.md` — **empty**.
  - `.agents/skills/` — only `untard/` + `upstream/` (user's personal skills). No vendor skills installed.
  - `.env.example` lists placeholders for HF_TOKEN, GITHUB_PAT_TOKEN, WANDB_API_KEY, CLOUDFLARE_, GCP_, AZURE_, AWS_, GALAXY_, GATEWAY_, NEON_, SUPABASE_, SENTRY_, POSTHOG_, OTEL_, CLICKHOUSE_, LANGFUSE_.
  - `.gitignore` only ignores `.env` — sensitive **files** beyond `.env` are not yet covered.
  - Git: uncommitted working-tree changes (`D outline.md`, `?? _ops/`).
- **liquid-primus** is a live research repo with `tidepool/` — an active project at stage s5 (budget.json: 145 GPU-h planned, 200 GPU-h enforced allowance, 2 GPUs, L40S default card; operator notes + datasets verified). This is the first consumer of the future TFL + compute setup.
- **references/** holds working clones of `transformerlab-app` + `transformerlab-examples` at upstream main.

---

## 5. Official / External Findings

### 5.1 Transformer Lab (official)
- Install: `uv tool install transformerlab-cli`; `lab server install` (interactive); run `~/.transformerlab/src/run.sh`; default port **8338**; default `admin@example.com` / `admin123` (change immediately).
- Provider types: **slurm, skypilot, runpod, vastai, dstack, aws, gcp, azure, local** — AWS/GCP/Azure are **beta**.
- Storage: fsspec abstraction — **AWS S3, GCS, Azure Blob, or localfs** (with NFS for SkyPilot). Storage backend configured at install; cannot be changed via web UI.
- CLI: `lab provider add --no-interactive --name ... --type skypilot --config '{"server_url":"...","api_token":"..."}'`.
- SDK contract (verified in upstream `lab-sdk`): `lab.finish(message=..., score={...})` — **no `status` argument**; `lab.error(message=...)` for failure. This matches the tidepool `budget.json` s5.1 note where `lab.finish(status=...)` raised.
- Upstream is a full-stack app (Electron+React+FastAPI+SQLite) with active releases and CI; docs split into "for-teams" hosting guide + docs site.

### 5.2 SkyPilot (official)
- Install: `uv tool install --with pip "skypilot[kubernetes,aws,gcp]"` (Python 3.9–3.13; `--with pip` required); run `sky api start --deploy`; verify `sky check`.
- v0.13.0 (Jul 2026). Supports **~25 clouds** incl. Kubernetes, Slurm, AWS, GCP, Azure, OCI, CoreWeave, Nebius, Lambda, RunPod, Fluidstack, Cudo, DigitalOcean, Paperspace, Cloudflare, Samsung, IBM, Vast, VMware, Seeweb, Prime Intellect, Shadeform, Verda, Crusoe.
- Team/multi-user mode: deploy the **SkyPilot API server** (Helm or `sky api start --deploy`), then clients `sky api` connect. TFL's SkyPilot provider is configured via server URL + user ID/name (+ optional Docker image, region/zone).
- Edge: **GPU Compass** (gpus.skypilot.co) for live price/capacity; official **SkyPilot Skill** for agents.
- SkyPilot is strongly aligned with TFL: it is TFL's documented path for "multiple clouds + Kubernetes" and the vendored docs show SkyPilot NFS-mounted storage setup.

### 5.3 Compute & credit market (2026)
- **AWS** official Promotional Credit terms (Dec 16, 2024): credits apply to Eligible Services only; **excluded**: AWS Marketplace (except Bedrock 3P model spend under Activate), Professional Services, Training, Certification, Route 53 registrations, and **upfront fees for Savings Plans/Reserved Instances**. Implication: EC2 GPU on-demand (g4/g5/p3/p4/p5) is eligible; marketplace GPU listings are not.
- **Modal**: Starter $0/mo + **$30/mo credits** (3 seats, 100 containers, 10 GPU concurrency); Team $250/mo + $100 credits; per-second billing; T4 ~$0.59, L4 ~$0.80, L40S ~$1.95, A100 40G ~$2.10, A100 80G ~$2.50, H100 ~$3.95, H200 ~$4.54, B200 ~$6.25/hr (approx, re-verify).
- **Cloudflare**: Workers AI **10,000 Neurons/day free** (pooled, resets daily), then $0.011/1k neurons; ~80 models incl. Llama 3.x/4, Qwen, Gemma, GPT-OSS, DeepSeek distills, FLUX, Whisper; no card needed on Workers Free; R2 10 GB free + 1M ops/mo. This is **edge inference, not GPU training** — the $20k/2-account budget is for Workers/edge workloads, not TFL training nodes.
- **Galaxy**: PaaS for Node/Python/Meteor/AdonisJS with managed Mongo/Postgres/Redis; **no GPU**; $600 credit; free tier with no expiration. Vocally a fit for app/tooling hosting, not TFL compute.
- **Oracle Cloud**: Always-Free **changed June 15, 2026** — Ampere A1 cut from 4 OCPU/24 GB to **2 OCPU/12 GB** (1,500 OCPU-hr + 9,000 GB-hr/mo), enforcement Aug 18, 2026; 2 AMD micro instances + 200 GB storage unchanged; **no GPUs in Always Free**; `$300`/30-day trial ≈ 100h A10 or A100; egress now **$0** globally.
- **GCP**: new-account `$300`/90-day trial; Always Free = 1 e2-micro VM (limited regions), 30 GB disk, Cloud Run quotas, Firestore, BigQuery, Gemini API allowances; **Vertex free tier = limited Gemini requests + 50 vCPU-hr/mo Agent Engine, no free GPU compute**. GPUs on-demand are paid.
- **Langfuse**: MIT, self-host via Docker Compose; acquired by ClickHouse (Jan 2026); **OTel ingestion first-class**; the natural addition for full-parity tracing.

---

## 6. Industry Standard Shape

The standard pattern for a "research platform over our own compute" is:

```
[User/agent] -> Transformer Lab (control plane, web UI + CLI + SDK)
              -> Compute Provider: SkyPilot (provider-agnostic)
                   -> AWS | GCP | Oracle | (Modal, RunPod, ...)
              -> Shared storage: S3/GCS (fsspec)
              -> Results -> experiment tracking (Langfuse/W&B) + model registry (HF)
```

Plus:
- Repo: **parent dir with independent repos** (t-labs = system, projects/ = many project repos), not a monorepo with nested submodules — unless the team explicitly wants one commit surface (they don't).
- Skills: **project-scoped** vendor skills via `npx skills add <owner/repo> --project ...` (user's standing preference; not global).
- Observability: **OTel-native** everywhere; Langfuse self-host for traces; Sentry/PostHog for app/product; ClickHouse for analytics.
- Tests: narrow, deterministic, meaningful — no trivial tests (outline's hard rule).

---

## 7. Implementation Options

### Option A — SkyPilot as the single TFL compute provider (recommended)

- `lab server install` → Provider type **skypilot** → deploy SkyPilot API server on the coordinator → configure AWS/GCP/Oracle credentials in `~/.sky/config.yaml` → `lab provider add --type skypilot --config '{"server_url":..., "api_token":...}'`.
- TFL routes all jobs through SkyPilot; SkyPilot picks the cheapest/available cloud per job.
- **When it fits**: multi-cloud credit pool; want one control plane; want burst/spot + autostop.
- **Tradeoffs**: + provider-agnostic, one TFL integration, efficient binpacking/spot, cost-aware across AWS/GCP/Oracle; − an extra layer to run/secure; SkyPilot version churn (v0.13+); some advanced TFL features (e.g. provider-specific log fetch) are delegated to SkyPilot.

### Option B — Native AWS provider in TFL (beta) for $800 spend

- Add TFL Provider type **aws** (beta) with an IAM user + policy; TFL launches EC2 directly.
- **When it fits**: if we only ever use AWS; want the simplest possible setup; accept beta.
- **Tradeoffs**: + no extra layer, direct EC2; − beta, AWS-only, loses burst to GCP/Oracle; still must avoid Marketplace for credit coverage.

### Option C — RunPod / dstack (community/dedicated GPU cloud) provider

- TFL supports runpod + dstack providers natively; both are simpler than SkyPilot for a single cloud GPU vendor.
- **When it fits**: if we buy GPU time from a neocloud (RunPod, etc.) rather than hyperscalers; dstack is open-source like SkyPilot.
- **Tradeoffs**: + simpler, no k8s; − we hold no meaningful RunPod credits; adds a provider whose credits we do not have; narrower than SkyPilot.

### Option D — Local-first (no provider) + evaluate later

- Run TFL purely local on the machine where it's installed; no shared storage; no cloud provider.
- **When it fits**: initial POC, single-machine dev, before spending any credits.
- **Tradeoffs**: + zero-cost ramp; − no multi-node, no artifact roaming, no burst; is not the final shape.

**Recommendation**: **Option A** (SkyPilot) for the real setup, with Option D as the zero-cost test bed in stage 1 of standing up TFL. Options B/C remain available but are not the primary path given our multi-cloud credit pool.

---

## 8. Technical Implications

- **Storage**: if we use SkyPilot + localfs, SkyPilot pods must mount the same `TFL_STORAGE_URI` (NFS) at the same container path (vendored `skypilot.md` confirms). With S3/GCS storage, fsspec mounts the bucket — no NFS needed. Recommend **S3 (or GCS)** to avoid NFS plumbing.
- **Compute**: TFL tasks declare `resources.accelerators` (e.g. `L40S:1`, `A100:1`) and SkyPilot maps them to the cheapest provider with capacity. We must pin providers/cards where determinism matters (tidepool's measurement protocol runs matched reruns on fixed hardware).
- **Network/egress**: Oracle egress now $0; AWS/Azure still charge egress (~$0.09/GB). Cross-cloud checkpoint pulls should account for this.
- **SDK**: use `lab.finish(message=..., score={...})` + `lab.error(...)` — never `status=` (verified against upstream lab-sdk; matches the tidepool budget.json s5.1 failure).
- **Observability**: the TFL SDK has no native Langfuse export; we keep the existing OTel pipeline and export via OTel exporter (Langfuse OTLP endpoint) or the lab SDK's logging/profiling to the existing stack.
- **Secrets**: `t-labs/.gitignore` should cover `.env*`, key files, and `*.key`/service-account JSONs; keep promoting the no-secrets-in-docs rule.

---

## 9. Project Implications

- **Repo**: create `projects/` as a sibling of `t-labs/` (or use the existing `labwork` parent). `t-labs` keeps system code; each project keeps its own repo (liquid-primus already does). **Do not** nest `projects/` inside the `t-labs` repo — it would couple a master system repo to every project's history and break the "one repo per project" flow.
- **Skills**: install vendor skills **project-scoped** in `t-labs` (and per-project where needed) after the decision: AWS, GCP, Cloudflare, Langfuse, PostHog, Sentry, Neon, Better-Auth, Stripe, SkyPilot agent skill. Keep the user's hub model for global symlinks.
- **Standards**: refresh `docs/standards/*` to t-labs reality (the current files say "Agency"); this is outline item 3 and should follow the framework decisions.
- **AGENTS.md**: create at `t-labs/AGENTS.md` pointing to durable docs and the standards.
- **Dev cycle**: build scripts/ + config/ + .github/ as the standard describes; changelog via `.changelog/changelog.md` (currently empty — needs a convention).
- **Observability parity**: add Langfuse self-host + OTel exporter to the existing stack; keep Sentry/PostHog/ClickHouse.
- **Docs/tests**: cover TFL install/ops; test critical paths only (install, provider health, job lifecycle, storage mount), not trivial units.

---

## 10. Risks And Constraints

- **Credit terms drift**: AWS promo terms, Modal pricing, Cloudflare neurons, Oracle free tier all change; **re-verify in cons consoles before spending** (this report is a snapshot dated 2026-08-26).
- **AWS Marketplace carve-out**: if any planned spend is Marketplace (e.g., third-party AMIs/GPU listings), the $800 credit may not apply. Prefer first-party AMIs and on-demand EC2 to stay eligible.
- **Oracle free-tier change**: existing Always-Free accounts above 2 OCPU/12 GB will be terminated (Aug 18, 2026) — plan around the new limits; no free GPUs.
- **GCP has no free GPU compute**: $300 trial is the only "free" GPU window on GCP; plan accordingly.
- **Modal credits are per-account**: 4×$30 = ~$120/mo; fine for interactive/dev, not for sustained multi-GPU training.
- **Cloudflare is not a training node**: Workers AI is edge inference; do not expect TFL training there.
- **Beta providers**: native AWS/GCP/Azure providers in TFL are beta; SkyPilot path avoids beta-dependence.
- **SkyPilot operational surface**: an extra service to secure + keep healthy; needs the coordinator host to reach cloud APIs.
- **Observability cost**: self-hosted Langfuse/ClickHouse has storage/compute cost; monitor retention.
- **Uncommitted t-labs state**: `D outline.md` + `?? _ops/` should be committed as part of the next step (or the deletion finalized) to keep the repo clean.
- **Assumption**: the credits listed (Modal $30×4, AWS $800, Cloudflare $20k/2accts, Galaxy $600, Oracle/GCP always-free) are as described by the user; **account-level verification is still required** — no console reads were performed for this report.

---

## 11. Recommended Direction

1. **Repo shape**: keep `t-labs` + `projects/` as **siblings under one parent**; each project its own repo. Do not nest projects inside t-labs.
2. **Stand up TFL** on a coordinator host, **provider = SkyPilot**, **storage = S3 first** (GCS alternative), using AWS credit-eligible services.
3. **Compute map** (verify in consoles): Modal $120/mo dev/interactive → AWS $800 primary training → GCP $300 trial + Always Free secondary → Cloudflare edge inference → Galaxy $600 for app hosting (not GPU) → Oracle Always-Free CPU-only services.
4. **Observability**: add self-hosted Langfuse (MIT) + OTel; keep Sentry/PostHog/ClickHouse.
5. **Skills**: `npx skills` project-scoped installs in `t-labs` (AWS/GCP/Cloudflare/Langfuse/PostHog/Sentry/Neon/Better-Auth/Stripe + SkyPilot skill).
6. **Docs**: refresh standards, create AGENTS.md, fill architecture/operations/runbooks, changelog convention, and add a zero-cost local POC step before any cloud spend.

---

## 12. Decision Points

### 12.1 Repo structure
- **Options**: A) `t-labs/` + `projects/` siblings under one parent (multi-repo, per-project repos) · B) monorepo with everything in one repo · C) `t-labs` repo containing `projects/` submodules.
- **Tradeoffs**: A: clean boundaries, project repos independent, evolving org-friendly; B: single commit surface, but couples a master system repo to every project's history and grows unbounded; C: submodules add a layer of git ceremony and stale-submodule footguns.
- **Recommendation**: **A**.
- **Why**: matches how liquid-primus already works (own repo), keeps `t-labs` a stable system repo, and scales to "a lot of projects."
- **Implication if different**: B/C change commit/sharing workflows and require additional tooling (submodule sync) without real benefit here.

### 12.2 Compute provider for Transformer Lab
- **Options**: A) SkyPilot (recommended) · B) native AWS provider (beta) · C) RunPod/dstack · D) local-only first.
- **Tradeoffs**: A: provider-agnostic, multi-cloud, cost-aware, mature; one extra service to run. B: simplest for AWS-only, but beta and cloud-locked. C: simpler but no credit base and narrower. D: zero-cost POC, not final.
- **Recommendation**: **A**, with **D** as the first no-spend test step.
- **Why**: the credit pool spans AWS/GCP/Oracle — SkyPilot is the standard way to use all of it through one TFL integration; matches official TFL docs.
- **Implication if different**: B/C/D each limit us to one cloud or a POC, reducing the value of the multi-credit pool.

### 12.3 TFL storage backend
- **Options**: A) AWS S3 (recommended) · B) GCS · C) localfs/NFS (SkyPilot-mount) · D) Azure Blob.
- **Tradeoffs**: A: fsspec-native, credit-eligible, no NFS; B: same but GCP-credit-eligible; C: no object-store cost but requires shared NFS mounted identically on host + pods and no cross-cloud roaming; D: no Azure credits we hold.
- **Recommendation**: **A** first, **B** as fallback.
- **Why**: S3 is the documented default, credits cover it, and it unblocks cross-cloud job mobility (SkyPilot) without NFS plumbing.
- **Implication if different**: C adds NFS mount complexity and kills cross-cloud mobility; D wastes effort with no credit.

### 12.4 Credits map & free tiers
- **Options**: A) Use everything as mapped (recommended) · B) Consolidate on AWS only · C) Consolidate on Modal only.
- **Tradeoffs**: A: maximum spread, each credit at its natural use; B: simplest but leaves $120/mo Modal, $300 GCP, Cloudflare, Galaxy, Oracle idle; C: serverless-friendly but limited GPU-hours/month and no persistent cluster.
- **Recommendation**: **A**.
- **Why**: Modal for dev/interactive, AWS for primary training, GCP/Oracle/Cloudflare/Galaxy for their niches — uses every credit where it's strongest.
- **Implication if different**: B/C forfeit meaningful free/cheap capacity.

### 12.5 Observability
- **Options**: A) Add self-hosted Langfuse + OTel (recommended) · B) Keep current stack only · C) Move to a managed LLM observability vendor.
- **Tradeoffs**: A: MIT, OTel-native, full parity, self-host cost = storage/compute; B: gap in LLM trace usability/parity; C: simplest but monthly cost + data-residency tradeoffs.
- **Recommendation**: **A**.
- **Why**: self-hosted Langfuse matches the existing OTel/Langfuse-in-.env direction and keeps data on our infra.
- **Implication if different**: B leaves parity incomplete; C adds cost/lock-in.

### 12.6 Vendor skills install (project-scoped)
- **Options**: A) Install the full useful set in `t-labs` (recommended) · B) Install only as needed per project · C) Global melt.
- **Tradeoffs**: A: one system repo with durable vendor knowledge; B: lean but re-discovers per project; C: violates the user's standing "project-scoped, not global" preference.
- **Recommendation**: **A** (project-scoped in `t-labs`; per-project add-ons when a project needs them).
- **Why**: matches the outline item 2 and the user's standing rules; keeps global skills hub untouched.
- **Implication if different**: B/C increase friction or violate the standing preference.

---

## 13. Decision Questions For Discussion

1. **Repo**: OK to keep `t-labs` + `projects/` as siblings under the parent (multi-repo, one repo per project)?
2. **Compute provider**: go with **SkyPilot** as the TFL provider (with a local, no-spend POC first)?
3. **Storage**: S3 first (GCS fallback) for TFL shared storage?
4. **Credits**: use the mapped plan — Modal dev/interactive, AWS primary training ($800), GCP trial secondary, Cloudflare edge, Galaxy app hosting, Oracle CPU-only — and verify each in its console before spend?
5. **Observability**: add self-hosted Langfuse + OTel for full parity?
6. **Skills**: install the vendor skills project-scoped in `t-labs` (AWS/GCP/Cloudflare/Langfuse/PostHog/Sentry/Neon/Better-Auth/Stripe + SkyPilot skill)?
7. **Standing up TFL**: phase it as local POC (zero spend) → SkyPilot provider with AWS+GCP → first real tidepool job through TFL?

---

## 14. Next Step If Accepted

1. Commit the current t-labs working tree (`D outline.md` → keep this report as the plan; `_ops/` now non-empty) with a clean, conventional commit.
2. Refresh `docs/standards/*` for t-labs + create `AGENTS.md` (outline items 3–4).
3. Build `scripts/`, `config/`, `.github/`, changelog convention (outline item 5).
4. Install vendor skills project-scoped in `t-labs` (outline item 2).
5. Stand up TFL: local POC → **SkyPilot** provider → S3/GCS storage → verify one job end-to-end (outline item 10).
6. Add self-hosted Langfuse + OTel exporter for parity (outline item 6).
7. Verify credits in consoles; build the durable compute/credits map doc (outline items 8–9).
8. Write the implementation plan for the above once the decisions are locked.

---

## 15. Sources

**Local**
- `t-labs/_ops/user-notes/outline.md`
- `t-labs/docs/standards/*` (dev-docs-standard.md, development-cycle.md, docs-tooling.md, planning-style.md, report-style.md, repository-layout.md, README.md)
- `t-labs/docs/t-labs/**` (vendored Transformer Lab docs: overview.md, concepts.md, setup/*, operations/*, examples/*)
- `t-labs/.env.example`, `t-labs/.gitignore`, `t-labs/.changelog/changelog.md`
- `liquid-primus/readme.md`, `liquid-primus/tidepool/overview.md`, `budget.json`, `operator-notes.md`
- `references/transformerlab-app/lab-sdk/src/lab/lab_facade.py`; `references/transformerlab-examples/unsloth-llm-train/{task.yaml,train.py}`

**Official / external**
- Transformer Lab: https://docs.transformerlab.ai/install/installation · https://transformerlab.ai/for-teams/install · https://github.com/transformerlab/transformerlab-app
- SkyPilot: https://docs.skypilot.ai/en/latest/getting-started/installation.html · https://docs.skypilot.ai/en/latest/reference/api-server/api-server-admin-deploy.html · https://github.com/skypilot-org/skypilot
- Modal: https://modal.com/pricing
- AWS: https://aws.amazon.com/awscredits (Promotional Credit Terms, Dec 16 2024) · https://aws.amazon.com/activate/terms · AWS re:Post "AWS Credits for purchases on the AWS Marketplace?"
- Cloudflare: https://developers.cloudflare.com/workers-ai/platform/pricing
- Galaxy: https://galaxycloud.app · https://galaxycloud.app/savings-plan · https://docs.galaxycloud.app/docs/apps/webapps/python
- Oracle: https://www.oracle.com/cloud/free/ · OCI Always-Free docs (Jun 15, 2026 change) · community reporting on the Ampere cut (terminalbytes.com, fullmetalbrackets.com, InfoQ)
- GCP: https://cloud.google.com/free · Vertex AI free tier/trial docs
- Langfuse: https://langfuse.com · https://docs.langfuse.com (self-host, OTel)
- Market context (flag as secondary, re-verify): Spheron "Free GPU Cloud Credits 2026", thundercompute "Free GPU Credits 2026", Beam/Blaxel/GMI 2026 pricing writeups.

*Revision note: every external pricing/limit fact is a 2026-08-26 snapshot and must be re-verified in the vendor console before any spend.*