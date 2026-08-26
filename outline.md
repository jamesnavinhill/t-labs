/# Transformer Lab Outline

/## Goals

* Stand up Transformer Lab on-top of our Cloud Compute for research and post-training projects
* Investigate SkyPilot for use-case and fit
* Audit full suite of platform credits and existing cloud compute power
* Exhaust quota, free-tiers, credits to create robust map of compute

/## Providers with Active Credits:

* Modal $30/mo x4 accounts
* AWS $800
* Cloudflare $20k/2 accts (10k ea.)
* Galaxy $600 [https://docs.galaxycloud.app/docs/apps/webapps/python](https://docs.galaxycloud.app/docs/apps/webapps/python)
* Oracle Always-Free
* GCP Always-Free
* Others?
* Current Promotions/Free Tiers?

/## Repo Structure

labwork Parent local directory

* t-labs//* - System code
* projects//* - Individual projects

/---

/### t-labs - Transformer Lab Framework

t-labs/

.agents/

* Skills/ /*Vendor skills

.changelog/

* changelog.txt /*Required session changelog

/_ops/

* /_legacy/ /*finished/complete docs
* audits/ /*troubleshooting audits
* feasibility/ /*feasibility reports
* roadmaps/ /*active plans
* user-notes/ /*scratchpad for user notes

docs/

* architecture/ /*Canon home for architecture docs
* operations/ /*Canon home for operation docs
* runbooks/ /*Canon home for runbooks/cheat-sheets
* standards/ /*Canon home for repo standards /*/*Needs refreshed for this project/*/*
* t-labs/ /*Canon source docs from Transformer Labs

README.md

.env

.env.example

.gitignore

/---

/### projects

liquid-primus/ /*Active research project, live GitHub repo /*/*Im not sure what the best flow for this is.. i obv want this system repo (t-lab) committed, and we'll want our projects (liquid-primus) committed in their own repos. maybe best to move projects/ to a 'sister' dir of t-labs? im going to have a lot of projects. i obv want them organized into a parent repo for organization sake. t-lab (master system repo) and t-lab-projects (master projects parent dir) would rather both live under one t-lab directory..

future-project/

/---
