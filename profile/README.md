<div align="center">

# Segmento

**Open-source customer intelligence platform for SMEs**

RFM segmentation · Churn prediction · Multi-channel automation · Self-hosted · GDPR-compliant

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)
[![Python](https://img.shields.io/badge/Python-3.13-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.x-3178C6?logo=typescript&logoColor=white)](https://www.typescriptlang.org/)
[![Next.js](https://img.shields.io/badge/Next.js-16-black?logo=next.js)](https://nextjs.org/)
[![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?logo=docker&logoColor=white)](https://docs.docker.com/compose/)

</div>

---

## What is Segmento?

Segmento is a **self-hosted, privacy-first customer intelligence platform** built for small and mid-sized businesses. Upload your transaction data, and Segmento automatically segments your customers using RFM analysis and K-Means clustering, predicts churn risk with XGBoost, performs ABC revenue analysis, and lets you act on these insights through email, SMS, webhooks, or Teams — all from a single dashboard.

No customer data ever leaves your infrastructure. PII is anonymized at ingestion time (SHA-256 hashing, auto-drop of sensitive fields), making Segmento compliant with GDPR and KVKK out of the box. It is a self-hosted alternative to expensive SaaS tools like Klaviyo or Segment.com, designed to run on a single VPS.

---

## Repositories

| Repository | Description | Stack |
|-----------|-------------|-------|
| [**engine**](https://github.com/segmentoapp/engine) | ML/AI backend — data ingestion, segmentation, churn prediction, action dispatch | Python · FastAPI · Celery · scikit-learn · XGBoost |
| [**gateway**](https://github.com/segmentoapp/gateway) | API gateway — auth, billing, project management, engine proxy | TypeScript · Bun · Hono · Drizzle ORM |
| [**web**](https://github.com/segmentoapp/web) | Analytics dashboard — segments, customers, campaigns, automations | Next.js · React · TailwindCSS · shadcn/ui |
| [**infra**](https://github.com/segmentoapp/infra) | Infrastructure — Docker Compose, observability, CI/CD | Docker · Prometheus · Grafana · ELK · Redis Sentinel |

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         Browser / Client                         │
└────────────────────────────┬────────────────────────────────────┘
                             │
                    ┌────────▼────────┐
                    │   Web (Next.js) │  :3000
                    └────────┬────────┘
                             │ REST
                    ┌────────▼────────┐
                    │    Gateway      │  :3001
                    │  (Bun + Hono)   │
                    └──┬──────────┬───┘
                       │          │
           ┌───────────▼─┐    ┌───▼──────────┐
           │  Engine API  │    │  PostgreSQL   │
           │  (FastAPI)   │    │  + pgvector   │
           │    :8000     │    └──────────────┘
           └──────┬───────┘
                  │ Celery tasks
         ┌────────▼────────┐
         │  Celery Workers  │
         │                  │
         │ ┌──────────────┐ │    ┌─────────┐
         │ │  ingestion   │ │    │  Redis  │
         │ │ segmentation │◄├────┤ Sentinel│
         │ │  predictive  │ │    │  (HA)   │
         │ │    action    │ │    └─────────┘
         │ │ maintenance  │ │
         │ └──────────────┘ │    ┌─────────┐
         └──────────────────┘    │  MinIO  │
                                 │ (S3 API)│
                                 └─────────┘
```

---

## Key Features

**Intelligent Data Ingestion**
- Lazy CSV/XLSX header scanning via Polars — no full-file load required
- Semantic column mapping with sentence-transformers (`all-MiniLM-L6-v2`) — automatically identifies customer ID, transaction date, revenue columns
- PII anonymization at ingest: SHA-256 hashing, auto-drop of detected sensitive fields
- Data quality scoring with actionable improvement suggestions

**Dynamic Customer Segmentation**
- RFM (Recency, Frequency, Monetary) feature engineering
- K-Means clustering with dual optimization (Elbow + Silhouette, k=2..10)
- Auto-labeling: Champions, Loyal Customers, At-Risk, Lost, and more
- Drift detection with automatic model retraining when customer distribution shifts >15%
- Vector similarity search via pgvector for "similar customers" lookup

**Predictive Analytics**
- Churn probability scoring (XGBoost) → low / medium / high risk
- ABC revenue analysis (cumulative revenue cutoffs: 80% / 95% / 100%)
- AI-generated cluster personas and names via Google Gemini

**Multi-Channel Action Engine**
- Boolean rule engine (AND/OR/nested conditions) layered on top of ML segments
- Dispatch via: Email (SendGrid), SMS (Twilio), Webhook (HTTP POST with retry), Microsoft Teams
- Campaign wizard with scheduling and audience preview
- Automation builder with scheduled, event-based, and manual triggers

**Production-Ready Infrastructure**
- PgBouncer connection pooling (transaction mode, 200 max clients)
- Redis Sentinel for HA task queue (1 master + 2 replicas + 3 sentinels)
- Prometheus + Grafana metrics, ELK log aggregation, OpenTelemetry tracing
- GitHub Actions CI/CD with self-hosted runner (no inbound SSH required)

---

## Quick Start

```bash
# 1. Clone the infrastructure repo
git clone https://github.com/segmentoapp/infra
cd infra

# 2. Copy and configure environment variables
cp .env.example .env.local
# Edit .env.local with your credentials (DB password, MinIO keys, API keys)

# 3. Start all services
docker compose -f docker-compose.local.yml up -d
```

Services will be available at:
- **Dashboard** → `http://localhost:3000`
- **Engine API** → `http://localhost:8000`
- **Gateway API** → `http://localhost:3001`
- **MinIO Console** → `http://localhost:9001`

See each repository's `README.md` for development setup instructions.

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| ML / Data | scikit-learn, XGBoost, Polars, Pandas, sentence-transformers, pgvector |
| AI | Google Gemini API (GenAI personas) |
| Task Queue | Celery 5.4, Redis Sentinel |
| Backend | Python 3.13, FastAPI, SQLAlchemy |
| API Gateway | TypeScript, Bun, Hono.js, Drizzle ORM |
| Frontend | Next.js 16, React 19, TailwindCSS 4, shadcn/ui, Recharts |
| Auth & Billing | better-auth, Stripe |
| Storage | PostgreSQL 18 + pgvector, MinIO (S3-compatible) |
| Observability | Prometheus, Grafana, Elasticsearch + Filebeat, OpenTelemetry / Tempo |
| Containerization | Docker, Docker Compose |

---

## Roadmap

### v0.2 — In Progress
- [x] Project-scoped RBAC (read / write / none per user per project)
- [ ] Cursor-based pagination for large customer datasets
- [ ] Uplift modeling (S-Learner via sklift) — persuadability scoring per customer

### v0.3 — Planned
- [ ] Shopify / WooCommerce connector (native plugin for transaction sync)
- [ ] Real-time event ingestion via webhook listener (Kafka / Redpanda)
- [ ] A/B test framework for campaign message variants
- [ ] SDK / embeddable segment widget for e-commerce storefronts
- [ ] Multi-language AI persona descriptions

### v1.0 — Vision
- [ ] LLM-powered natural language segment builder — *"find customers who haven't bought in 60 days and spent over $200 historically"*
- [ ] GPU-accelerated segmentation via RAPIDS cuML
- [ ] Federated learning for privacy-preserving cross-tenant model improvement
- [ ] White-label / OEM packaging for resellers

---

## Contributing

Segmento is early-stage and contributions are very welcome. Good starting points:

- Browse [open issues](https://github.com/segmentoapp/engine/issues) tagged `good first issue`
- Add a new channel integration (Slack, WhatsApp, etc.) in `engine/src/infrastructure/action/`
- Improve test coverage — unit tests live in `engine/tests/`
- Translate the web dashboard (i18n files in `web/messages/`)

Please open an issue before starting a significant PR so we can align on approach.

---

## License

MIT — see [LICENSE](LICENSE).

---

<div align="center">
Built with care for SMEs who deserve enterprise-grade customer intelligence without enterprise pricing.
</div>
