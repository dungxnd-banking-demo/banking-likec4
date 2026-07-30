# banking-likec4 — Architecture Documentation

LikeC4 architecture docs for the Banking Demo ecosystem — 4 repos, event-driven banking on K8s (AWS EC2), documented in 14 C4 diagrams.

> **Companion repos:** [banking-demo-revamp](https://github.com/dungxnd-banking-demo/banking-demo-revamp) (source) · [banking-demo-gitops](https://github.com/dungxnd-banking-demo/banking-demo-gitops) (Helm + ArgoCD) · [bankind-demo-infra](https://github.com/dungxnd-banking-demo/bankind-demo-infra) (Terraform + Ansible)

## Quick start

```bash
bun install
bun dev
```

Opens a local dev server at http://localhost:5173 with hot reload. Edit any `.c4` file and diagrams update in real time.

## LikeC4 features used

| Feature | Where |
|---|---|
| **Relationship kinds** (visual differentiation) | 10 kinds — `https`(solid), `nat_rpc`(solid+diamond), `jetstream_evt`(dashed+amber), `websocket_cn`(solid+sky), `redis_cmd`(dashed+red), `sql_tx`(solid+gray), `gitops`(solid+diamond), `provision`(dotted), `push_image`(dotted), `oltp_grpc`(dotted+gray) |
| **Element metadata** | Every service: language, port, endpoints, pool size, frameworks, storage size |
| **Tags with colors** | `#cqrs`(green), `#event_driven`(blue), `#stateful`, `#stateless`, `#realtime` |
| **Global styles** | `styleGroup theme_banking` — shared actor/external coloring across all views |
| **View groups** | All views use `group { include ... }` for logical boundary boxes |
| **Auto-layout** | `LeftRight` on landscape/containers/pipeline, `TopBottom` on flow/data/o11y |
| **Rank constraints** | `rank same` on container view — forces 5 clean rows (entry, gateway, bus, services, data) |
| **Deployment model** | `cloud` → `vpc` → `ec2` hierarchy with `instanceOf` for all 17 containers |
| **Deployment relationships** | Cross-node instances with metadata |
| **Deployment views** | 4 views with `includeAncestors` for physical topology |
| **Custom colors** | `bank_green`, `bank_indigo`, `bank_amber`, `bank_red` — semantic banking theme |
| **Element notations** | 12 element kinds with shapes (person, browser, storage) and tech icons |
| **Markdown descriptions** | All elements and views use rich markdown with bold, tables, code blocks |

## Project structure

```
banking-likec4/
├── specification.c4      12 element kinds, 10 relationship kinds, 5 tags, 3 deployment nodes, 4 colors
├── model.c4              Logical model (actors, external systems, Banking System × 17 containers, ArgoCD)
├── deployment.c4         AWS → VPC → 4 EC2 nodes, all containers deployed via instanceOf
├── views.c4              9 logical views + 2 dynamic sequence views
├── deployment-views.c4   4 deployment views (Infra, App Pods, Security, Audit)
├── likec4.config.json    Project config (redirect to index view)
├── package.json          Bun scripts: dev / build / preview / export / validate / format
└── .github/workflows/    CI: validate → build → deploy to GitHub Pages
```

## Views (14 total)

### Logical

| View | What it shows |
|---|---|
| `index` | C4 Level 1 — System Landscape. Actors, external systems, Banking System, infra repo |
| `banking-system` | C4 Level 2 — Containers. All 17 deployable units with groups and auto-layout |
| `api-request-flow` | How an HTTP REST call flows: Browser → Caddy → Kong → api-producer → NATS → service |
| `nats-subjects` | NATS subject hierarchy — RPC subjects and JetStream event subjects |
| `transfer-flow` | Money transfer with CQRS: DB tx → Redis pipeline → JetStream → WebSocket push |
| `data-layer` | PostgreSQL tables + Redis key space (session, cache, balance hash, pub/sub) |
| `observability-stack` | Structured logs + Prometheus metrics + OTel traces → self-hosted or Instana |
| `gitops-pipeline` | CI → GitHub → GHCR → ArgoCD → Kubernetes (3 repos, 1 pipeline) |
| `redis-keyspace` | CQRS read model: session, user_cache, balance hash, notify pub/sub |

### Deployment

| View | What it shows |
|---|---|
| `deployment-infra` | AWS EC2 cluster: master + 2 workers + warpgate node in a VPC |
| `deployment-app-pods` | App pod distribution across worker nodes |
| `deployment-security` | Post-lockdown security posture — only 80/443/2222 open |
| `deployment-audit` | Audit logging pipeline: kube-apiserver → Vector → Parseable → SeaweedFS |

## Scripts

| Command | Does |
|---|---|
| `bun dev` | Start dev server with hot reload |
| `bun run build` | Build static site to `dist/` (for GitHub Pages) |
| `bun run preview` | Preview the built site locally |
| `bun run export` | Export all views to PNG |
| `bun run validate` | Check syntax, references, layout (CI-friendly) |
| `bun run format` | Format `.c4` files in place |
| `bun run format:check` | Check formatting without writing (CI-friendly) |

## Deploy to GitHub Pages

Push to `main` and the workflow handles it — validates the model, builds the static site, deploys to GitHub Pages. Requires Pages enabled in repo Settings (source: GitHub Actions).

To use a different repo name, update `--base` in `package.json` → `build` script.

## Relationship kinds

The model uses semantic relationship kinds instead of generic labels:

| Kind | Used for |
|---|---|
| `http` | HTTPS / HTTP between services (solid, secondary color) |
| `nats-rpc` | NATS Core request/reply via `nats/micro` (solid, green) |
| `nats-jetstream` | JetStream durable event stream (dashed, amber) |
| `ws` | WebSocket connections (solid, sky) |
| `redis-pubsub` | Redis publish/subscribe (dashed, red) |
| `sql` | PostgreSQL queries (solid, gray) |
| `deploy` | CI/CD deployments (dotted, muted) |
| `sync-gitops` | ArgoCD GitOps sync (solid, sky) |
| `provisions` | Terraform/Ansible infrastructure provisioning (dotted, muted) |

## Tech stack

- **Bun** — JavaScript runtime and package manager
- **LikeC4** 1.59.2 — DSL for C4 architecture modeling
- **Graphviz** (WASM) — diagram layout engine, bundled with LikeC4
- **GitHub Actions** — CI/CD: validate → build → deploy to Pages
- **GitHub Pages** — hosting the static site
