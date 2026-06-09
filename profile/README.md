<div align="center">

<img src="https://img.shields.io/badge/Estonia-e--resident-0072CE?style=flat-square&logo=data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9IjAgMCA5IDYiPjxyZWN0IHdpZHRoPSI5IiBoZWlnaHQ9IjIiIGZpbGw9IiMwMDcyQ0UiLz48cmVjdCB5PSIyIiB3aWR0aD0iOSIgaGVpZ2h0PSIyIiBmaWxsPSIjMDAwIi8+PHJlY3QgeT0iNCIgd2lkdGg9IjkiIGhlaWdodD0iMiIgZmlsbD0iI0ZGRiIvPjwvc3ZnPg==" alt="Estonia"/>

```
╔══════════════════════════════════════════════════════════════════╗
║                                                                  ║
║    ██╗  ██╗███████╗ █████╗ ██╗     ███████╗██████╗ ████████╗    ║
║    ██║  ██║██╔════╝██╔══██╗██║     ██╔════╝██╔══██╗╚══██╔══╝    ║
║    ███████║█████╗  ███████║██║     █████╗  ██████╔╝   ██║       ║
║    ██╔══██║██╔══╝  ██╔══██║██║     ██╔══╝  ██╔══██╗   ██║       ║
║    ██║  ██║███████╗██║  ██║███████╗███████╗██║  ██║   ██║       ║
║    ╚═╝  ╚═╝╚══════╝╚═╝  ╚═╝╚══════╝╚══════╝╚═╝  ╚═╝   ╚═╝       ║
║                                                                 ║
║          Friction Intelligence Platform  ·  v0.1.0 Coral        ║
║                                                                 ║
╚═════════════════════════════════════════════════════════════════╝
```

**The world's first open-source Friction Intelligence Platform.**  
Named after the axolotl — the only vertebrate that fully regenerates damaged tissue.  
Platforms heal. Scores decay. Drift corrects itself.

[![License](https://img.shields.io/badge/License-Apache_2.0-00B4B4?style=flat-square)](https://opensource.org/licenses/Apache-2.0)
[![Plugin](https://img.shields.io/badge/@backstage--community-plugin--healert-7C3AED?style=flat-square&logo=backstage&logoColor=white)](https://github.com/backstage/community-plugins)
[![Estonia](https://img.shields.io/badge/Healert_OÜ-Registered_in_Estonia-0072CE?style=flat-square)](https://healert.io)
[![Status](https://img.shields.io/badge/status-production_ready-16A34A?style=flat-square)]()

</div>

---

## The Problem

Platform engineering teams invest months building golden paths — IaC pipelines, service templates, GitOps workflows. Then developers find shortcuts.

```
kubectl exec payments-api -- /bin/bash          ← bypasses audit trail
kubectl patch deployment payments --replicas=0  ← drifts from git
kubectl annotate deploy payments bypass=true    ← skips policy gates
```

These bypass events are invisible. No alert fires. No dashboard shows them. The score just silently degrades until an incident — or a compliance audit — reveals months of accumulated drift.

**Healert makes the invisible visible.**

---

## What Healert Does

```
Kubernetes Audit Log
        ↓  one JSON event per line, tailed from EOF
Healert Agent                     Go binary · DaemonSet · zero deps
        ↓  isInternalSystemActor() — human vs controller
        ↓  matchRules()            — 5 detection rules
        ↓  POST /events            — API key auth · 60 req/min limit
Healert Backend                   FastAPI · SQLite · WAL mode
        ↓  calculate_score()       — exponential time decay
        ↓  GET /friction/{ref}     — per-entity friction score
Backstage Plugin                  @backstage-community/plugin-healert
        ↓
FrictionScoreCard  ·  FrictionHeatmap  ·  PDF Export
```

Every friction event is scored with **exponential time decay** — recent bypass events score higher than old ones. As teams fix their behavior, scores heal automatically. A service that had 10 bypass events last month but zero this week trends back to zero on its own.

> No manual resets. No stale scores. Platforms heal — just like the axolotl.

---

## Repositories

| Repository | Description | Language |
|---|---|---|
| [healert-io/agent](https://github.com/healert-io/agent) | Go agent · DaemonSet · 5 detection rules · `healert.sh` management | Go |
| [healert-io/backend](https://github.com/healert-io/backend) | FastAPI backend · exponential decay scoring · SQLite | Python |
| [backstage/community-plugins](https://github.com/backstage/community-plugins) | Backstage plugin · `@backstage-community/plugin-healert` | TypeScript |

---

## Detection Rules

Five rules active in v0.1.1 Coral. All configurable. All production-tested on k3s.

| Rule | Severity | What It Catches |
|---|---|---|
| `kubectl-exec` | 🔴 High | Direct shell access to running pods |
| `pipeline-skip` | 🔴 High | Policy bypass annotation on deployments |
| `config-drift` | 🔴 High | Direct write to workload resources outside GitOps |
| `port-forward` | 🟡 Medium | Direct port-forward — bypasses service mesh |
| `emergency-access` | 🟡 Medium | Direct secret access via kubectl |

10+ optional rules in `rules.yaml` — uncomment to enable: `rbac-change`, `namespace-creation`, `secret-deletion`, `ingress-drift`, `pv-deletion`, and more.

---

## Scoring Formula

```
Score = min(100, round(weighted_total / threshold × 100))

weighted_total = Σ ( points × 0.5 ^ (age_days / half_life) )
```

| Today | 7 days ago | 14 days ago | 30 days ago |
|---|---|---|---|

The axolotl principle — platforms regenerate as behavior improves.
| 100% weight | 50% weight | 25% weight | ~3% weight |

Scores decay automatically. A service that had 5 bypass events last week will score ~50 today and ~0 in two weeks with no new events. **No manual resets needed.**

---

## Quick Start

```bash
# 1. Deploy the backend + agent (local mode)
git clone https://github.com/healert-io/agent.git
cd agent
go build -o healert-agent main.go
./healert.sh init && ./healert.sh setup && ./healert.sh start

# 2. Install the Backstage plugin
# In your Backstage packages/app:
yarn add @backstage-community/plugin-healert

# 3. Add to your entity page
# packages/app/src/components/catalog/EntityPage.tsx:
import { EntityHealertContent } from '@backstage-community/plugin-healert';

# 4. Configure Backstage proxy (app-config.yaml):
# proxy:
#   '/healert':
#     target: 'http://localhost:8000'

# 5. Trigger your first detection
kubectl exec $POD -n default -- echo test
# → Score updates in Backstage within seconds
```

---

## Production Deployment

```bash
# Build and push Docker image
docker build -t ghcr.io/healert-io/agent:0.1.1 .
docker push ghcr.io/healert-io/agent:0.1.1

# Deploy as Kubernetes DaemonSet
./healert.sh start kubernetes

# One agent pod per node — automatic on new nodes
kubectl get pods -n healert-system -o wide
```

The agent runs as **nonroot uid=65532**, with `readOnlyRootFilesystem: true`, `capabilities.drop: ALL`, and `system-node-critical` priority — never evicted under memory pressure.

---

## Key Technical Properties

```
Agent binary:       ~5MB static Go binary · zero external dependencies
Docker image:       ~12MB distroless · no shell · no package manager
DaemonSet:          nonroot · readOnlyRootFilesystem · NetworkPolicy
Auth:               hmac.compare_digest · constant-time · timing-attack safe
Namespace filter:   config.ignore_namespaces + K8S_NAMESPACE Downward API
Entity resolution:  automatic from Kubernetes event namespace · zero config
Scoring:            exponential time decay · configurable threshold + half-life
Retention:          30-day rolling window · automatic cleanup
```

---

## About Healert OÜ

Healert OÜ is a platform intelligence company **registered in Estonia** under the EU e-Residency programme.

### Why Healert — the Axolotl Principle

The name Healert comes from the **axolotl** — the Mexican salamander famous for being the only vertebrate that can fully regenerate damaged tissue. Severed limb, damaged heart, injured spinal cord — the axolotl grows it back perfectly, every time.

```
  ><(((º>    the axolotl regenerates
              what was lost
```

Platform health works the same way. When developers bypass the golden path — through kubectl exec, direct patches, or pipeline skips — the platform is injured. Left undetected, those injuries accumulate into drift, compliance gaps, and incidents.

**Healert detects the injury. Scores heal as teams improve.**

A service with 10 bypass events today that implements proper controls next week will score near zero in two weeks — automatically, without resets, without manual intervention. The platform regenerates.

> *"The axolotl does not fight damage — it absorbs it, measures it, and grows back stronger. That is what a friction intelligence platform should do."*

We build open-source infrastructure that gives platform engineering teams the visibility they need to guide — not police — developer behavior. Friction scores are not punishment. They are a signal. The axolotl does not blame the wound. It heals it.

The Healert agent answers the question platform engineers have always wanted to ask:

> *"Are developers actually using the golden path — or finding ways around it — and are we healing or drifting?"*

---

<div align="center">

**Apache-2.0 · Copyright 2026 Healert OÜ · Tallinn, Estonia**

[healert.io](https://healert.io) · [Backstage Plugin](https://github.com/backstage/community-plugins) · [Agent](https://github.com/healert-io/agent) · [Backend](https://github.com/healert-io/backend) . [Email: Hello@healert.io]

</div>
