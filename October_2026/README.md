# October 2026

Two hands-on demo projects from DevChangeMakers' October 2026 events. Each folder is self-contained and has its own detailed README with step-by-step instructions.

| Project | What it is | Main stacks |
| --- | --- | --- |
| [notion-task-board](./notion-task-board) | Flask task board that reads a Notion database, runs on Kubernetes, and is monitored live with Prometheus + Grafana | Notion · Python/Flask · Docker · Kubernetes (kind) · Helm · Prometheus · Grafana · Devin Desktop · GitHub Copilot CLI |
| [mentor-match](./mentor-match) | AI mentor marketplace for developers in Bangladesh: chat to find a mentor, then book and pay through Stripe Connect | Node.js/Express · Qwen on Cerebras · Stripe Connect · Figma Make · Claude Code · Amazon Bedrock · Grok (Cursor) · GitHub Copilot |

---

## 1. Notion Task Board on Kubernetes

A 3-hour workshop. You build a small Flask app that reads tasks from a Notion database, deploy it to a local Kubernetes cluster, collect its metrics with Prometheus, and chart them live in Grafana.

**The demo moment:** move a task to **Done** in Notion, and within about 30 seconds the Grafana dashboard updates.

```
Notion (Tasks DB) ──API, every 30 s──▶ Flask app (Pod) ──/metrics──▶ Prometheus ──▶ Grafana
                                       └──────────── kind cluster (Kubernetes) ────────────┘
```

### Tech stack

| Layer | Technology | Role |
| --- | --- | --- |
| Data source | **Notion API** | Tasks database, the source of truth |
| App | **Python 3.12, Flask**, `requests`, `prometheus_client` | Shows tasks and exposes `/metrics` |
| Container | **Docker** (`python:3.12-slim`) | Packages the app as `task-board:v1` |
| Orchestration | **Kubernetes** via **kind**, **Helm** | Runs the app and the monitoring stack |
| Monitoring | **Prometheus** (kube-prometheus-stack, ServiceMonitor) | Scrapes and stores task counts |
| Dashboards | **Grafana** | Visualizes the counts |
| AI tooling | **Devin Desktop**, **GitHub Copilot CLI** | Generates the app; runs and debugs cluster commands |

### What's inside

```
notion-task-board/
├── app.py              Flask app + Prometheus metrics
├── Dockerfile          Container image build
├── requirements.txt    Python dependencies
├── .env.example        NOTION_TOKEN, NOTION_DATABASE_ID
├── k8s/                deployment.yaml, service.yaml, servicemonitor.yaml
├── grafana/panels.md   Dashboard panel queries
└── docs/               Devin + Copilot prompts, build report
```

### Getting started

You need Docker (or another container runtime), `kind`, `kubectl`, `helm`, and a Notion integration token with access to your Tasks database.

1. Copy `.env.example` to `.env` and fill in `NOTION_TOKEN` and `NOTION_DATABASE_ID`.
2. Build the image and load it into kind: `docker build -t task-board:v1 .` then `kind load docker-image task-board:v1 --name workshop`.
3. Install kube-prometheus-stack with Helm, apply the manifests in `k8s/`, and port-forward to the app and Grafana.

The full walkthrough, including beginner explanations of Kubernetes, Pods, Prometheus and Grafana, is in [notion-task-board/README.md](./notion-task-board/README.md).

---

## 2. MentorMatch: AI Mentor Marketplace

A live demo of seven tools working together. A learner types what they want to learn, AI extracts the skills and finds matching mentors, and the learner books and pays a mentor through **Stripe Connect** (test mode). The platform keeps a 10% fee.

```
"I want to learn Terraform on AWS"
  → Qwen on Cerebras extracts skills (terraform, aws)
  → match against src/mentors.json
  → Qwen writes the reply, mentor card appears
  → Book session → Stripe Checkout ($15)
  → $13.50 to the mentor, $1.50 platform fee
```

### Tech stack

**In the app**

| Layer | Technology | Role |
| --- | --- | --- |
| Backend | **Node.js 20.6+, Express** (ES modules) | API server (`server.js`) |
| AI | **Qwen on Cerebras** (`@cerebras/cerebras_cloud_sdk`) | Skill extraction (~500 ms) and chat replies |
| Matching | Plain JS over `src/mentors.json` | Whole-word, case-insensitive skill matching |
| Payments | **Stripe Connect** (Express accounts, destination charges) | Mentor onboarding, payments, platform fee, payouts |
| Frontend | Plain **HTML/CSS/JS** in `public/` | Chat UI and mentor onboarding pages |
| Tests | Node's built-in test runner (`node --test`) | 8 tests for skill matching |

**Used to build it**

| Tool | Job |
| --- | --- |
| **Figma Make** | Generated the whole UI from one prompt |
| **Claude Code** (+ Figma MCP) | Main builder: design to code, backend changes, fixes |
| **Amazon Bedrock** | Optional way to run Claude Code on AWS |
| **Grok** (Cursor + Grok Bot) | Quick UI edits in Cursor; audited the repo and opened fix branches |
| **GitHub Copilot** | Reviews every pull request before merge |

### What's inside

```
mentor-match/
├── server.js             Express server and API routes
├── src/                  cerebras.js, reply.js, match.js, skills.js, stripe.js, mentors.json
├── public/               index.html, app.js, onboard.html, styles.css
├── test/                 Skill-matching tests
├── scripts/check-keys.js Verifies the Cerebras and Stripe keys
├── design/figma-brief.md Prompt used for Figma Make
└── AGENTS.md             Instructions shared by AI coding agents
```

### Getting started

You need Node.js 20.6+, a Cerebras API key (free tier at [cloud.cerebras.ai](https://cloud.cerebras.ai)), and later a Stripe account in test mode. Commands are for Windows PowerShell:

```powershell
cd October_2026\mentor-match
npm install
Copy-Item .env.example .env          # fill in CEREBRAS_API_KEY, CEREBRAS_MODEL, STRIPE_SECRET_KEY
New-Item -ItemType Directory -Force data | Out-Null
npm test        # skill-matching tests, no keys needed
npm run check   # verifies your API keys
npm run dev     # http://localhost:3000
```

The demo script, lessons learned and pull request history are in [mentor-match/README.md](./mentor-match/README.md).

---

## Notes

- **Never commit `.env` files.** Both projects ship only `.env.example`.
- Stripe in MentorMatch must use **test-mode** keys (`sk_test_...`); the server refuses live keys.
- The original repositories are [mitul3737/notion-task-board](https://github.com/mitul3737/notion-task-board) and [mitul3737/mentor-match](https://github.com/mitul3737/mentor-match). The copies here won't pick up later changes made there.
