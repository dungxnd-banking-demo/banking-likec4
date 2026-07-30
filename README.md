# banking-likec4 — Architecture Documentation

LikeC4 architecture docs for the Banking Demo ecosystem — event-driven banking on K8s (AWS EC2), documented in 16 C4 diagrams.

> **Companion repos:** [banking-demo-revamp](https://github.com/dungxnd-banking-demo/banking-demo-revamp) (source) · [banking-demo-gitops](https://github.com/dungxnd-banking-demo/banking-demo-gitops) (Helm + ArgoCD) · [bankind-demo-infra](https://github.com/dungxnd-banking-demo/banking-demo-infra) (Terraform + Ansible)

## Quick start

```bash
bun install
bun dev
```

Opens a local dev server at http://localhost:5173 with hot reload. Edit any `.c4` file and diagrams update in real time.

## Views (16 total)

### Architecture

| View | Type | What it shows |
|---|---|---|
| `Landscape / C4 Level 1` | System Landscape | All actors, external systems, Banking System, GitOps, infra |
| `Containers / C4 Level 2` | Container Diagram | 17 deployable units with rank constraints (6 clean rows) |
| `Requests / API Flow` | Zoom-in | Caddy → Kong (3 routes: SPA, API, WebSocket) → NATS RPC → services |
| `Messaging / NATS Subjects` | Zoom-in | NATS request/reply subjects + JetStream stream |
| `Features / Transfer Flow` | CQRS Detail | Three-tier: SerializableTx → Redis pipeline → JetStream → WebSocket |
| `Data / PostgreSQL + Redis` | Data Layer | 4 services sharing PG + Redis, showing per-service connections |
| `Operations / Observability` | Cross-cutting | Prometheus metrics, Instana traces, Grafana dashboard |

### CI/CD

| View | Type | What it shows |
|---|---|---|
| `Pipeline / GitOps CI/CD` | Pipeline | CI build → GHCR → gitops tag bump → ArgoCD auto-sync |

### Dynamic (Sequence Diagrams)

| View | Type | What it shows |
|---|---|---|
| `Use Cases / 12.1 Transfer / Send Money` | Sequence | End-to-end transfer with `try/catch`, `alt`, parallel Redis+JetStream |
| `Use Cases / 03.1 Authentication / Login & Register` | Sequence | Register (bcrypt) / Login (cache-then-DB) with `alt`/`else` branches |
| `Use Cases / 05.1 Balance / Check Balance` | Sequence | CQRS read path: Redis primary, PostgreSQL fallback |
| `Use Cases / 15.1 Notifications / Real-time Push` | Sequence | WebSocket lifecycle with `loop`, `opt` heartbeat, Redis pub/sub |

### Deployment

| View | Type | What it shows |
|---|---|---|
| `Infra / AWS EC2 Cluster` | Deployment | 4-node K8s cluster: master + 2 workers + warpgate in default VPC |
| `Infra / Pod Distribution` | Deployment | Worker-1 (entry+messaging) vs Worker-2 (services+data) |
| `Security / Post-Lockdown` | Deployment | Only 80/443/2222 open; SSH+kube-api closed |
| `Security / Audit Pipeline` | Deployment | kube-apiserver → Vector → Parseable → SeaweedFS |

## LikeC4 features used

| Feature | Where |
|---|---|
| **Relationship kinds** (visual differentiation) | 10 kinds — `https`(solid), `nat_rpc`(solid+diamond), `jetstream_evt`(dashed+amber), `websocket_cn`(solid+sky), `redis_cmd`(dashed+red), `sql_tx`(solid+gray), `gitops`(solid+diamond), `provision`(dotted), `push_image`(dotted), `oltp_grpc`(dotted+gray) |
| **Relationship metadata** | Key relationships carry metadata — `isolation`, `retries`, `dedup`, `commands`, `load_balanced` |
| **Element metadata** | Every service: language, port, endpoints, pool size, replicas, storage size |
| **Tags with colors** | `#cqrs`(green), `#event_driven`(blue), `#stateful`, `#stateless`, `#realtime` — assigned on specification to element kinds |
| **Global style groups** | `theme_banking` — shared service/data coloring across views |
| **View folder organization** | 4 folders: `Architecture/`, `CI/CD/`, `Dynamic/`, `Deployment/` with sub-folders |
| **View groups** | Boundary boxes in landscape, container, transfer, and observability views |
| **Auto-layout** | `LeftRight` on landscape/containers/pipeline/transfer; `TopBottom` on flow/data/o11y/NATS |
| **Rank constraints** | `rank same` on container view — forces 6 clean rows |
| **`navigateTo` links** | Landscape → Container view; Transfer → Transfer Sequence |
| **Dynamic views (sequence)** | 4 sequence diagrams with `alt`/`else`, `try`/`catch`/`finally`, `loop`, `opt`, `parallel` flow control |
| **Deployment model** | `cloud` → `vpc` → `ec2` hierarchy with `instanceOf` for all containers; deployment metadata on nodes |
| **Deployment views** | 4 views with `includeAncestors`, organized under `Deployment/` folder |
| **Custom colors** | `bank_green`, `bank_indigo`, `bank_amber`, `bank_red` — semantic banking theme |
| **Element notations** | 12 element kinds with shapes (`person`, `browser`, `storage`), tech icons, and `description` on specification |
| **Tech icons** | `tech:github`, `tech:docker`, `tech:terraform`, `tech:argocd`, `tech:grafana`, `tech:kubernetes`, `tech:kong`, `tech:go`, `tech:nats-icon`, `tech:postgresql`, `tech:redis` |
| **Markdown descriptions** | All elements and views use rich markdown with bold, lists, code blocks |

## Project structure

```
banking-likec4/
├── specification.c4       12 element kinds, 10 relationship kinds, 5 tags, 4 deployment nodes, 4 colors
├── model.c4               Logical model (actors, 5 external systems, Banking System × 17 containers, ArgoCD)
├── deployment.c4          AWS → VPC → 4 EC2 nodes, all containers deployed via instanceOf
├── views.c4               8 logical/pipeline views with folder organization + shared styles
├── dynamic-views.c4       4 sequence diagrams with flow control (alt, loop, try/catch, opt)
├── deployment-views.c4    4 deployment views with folder organization
├── likec4.config.json     Project config (redirect to index view)
├── package.json           Bun scripts: dev / build / preview / export / validate / format
└── .github/workflows/     CI: validate → build → deploy to GitHub Pages
```

## Relationship kinds

The model uses semantic relationship kinds instead of generic labels:

| Kind | Visual | Used for |
|---|---|---|
| `https` | solid · secondary | HTTPS between services |
| `nat_rpc` | solid · green · diamond head | NATS Core request/reply via `nats/micro` |
| `jetstream_evt` | dashed · amber | JetStream durable event stream |
| `websocket_cn` | solid · sky | WebSocket connections |
| `redis_cmd` | dashed · red | Redis commands + pub/sub |
| `sql_tx` | solid · gray | PostgreSQL queries + transactions |
| `gitops` | solid · sky · diamond head | ArgoCD GitOps sync |
| `provision` | dotted · muted | Terraform/Ansible infrastructure provisioning |
| `push_image` | dotted · muted | Docker push → GitHub Container Registry |
| `oltp_grpc` | dotted · gray | OpenTelemetry OTLP/gRPC traces |

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

## Tech stack

- **Bun** — JavaScript runtime and package manager
- **LikeC4** 1.59.2 — DSL for C4 architecture modeling
- **Graphviz** (WASM) — diagram layout engine, bundled with LikeC4
- **GitHub Actions** — CI/CD: validate → build → deploy to Pages
- **GitHub Pages** — hosting the static site
