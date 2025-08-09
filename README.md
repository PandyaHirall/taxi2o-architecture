# Taxi2o – Azure Lakehouse Architecture

![Architecture Diagram](images/architecture.png)

## Table of Contents
1. [Mission](#mission)
2. [Objectives](#objectives)
3. [Cloud Architecture (Final)](#cloud-architecture-final)
4. [Why These Azure Services (and Trade-offs)](#why-these-azure-services-and-trade-offs)
5. [Pipelines & Orchestration](#pipelines--orchestration)
6. [Access Patterns & BI Modes](#access-patterns--bi-modes)
7. [Security & Governance](#security--governance)
8. [Data Quality Playbook (Silver)](#data-quality-playbook-silver)
9. [Cost Management](#cost-management)
10. [Improvements Across Phases](#improvements-across-phases)
11. [Example KPIs & Dashboards](#example-kpis--dashboards)
12. [DevOps & Folder Structure](#devops--folder-structure)
13. [Implementation Steps (Walkthrough)](#implementation-steps-walkthrough)



## Mission
Design and implement a secure, scalable cloud-based data platform that unifies Booking, Driver, Vehicle, GPS, and Customer Feedback data to deliver near real-time operational insights and reliable curated reporting using a modern Azure lakehouse.

---

## Objectives
- Unify core Taxi2o data sources in a single governed platform.
- Ingest both streaming and batch: GPS/app events in real time; systems of record on schedule.
- Adopt Medallion (Bronze → Silver → Gold) with Delta for ACID, time travel, and upserts.
- Serve analytics with Synapse SQL and Power BI using clear access patterns.
- Operational excellence: monitoring, alerts, retries, and fault tolerance by design.
- Security & cost control: private endpoints, Key Vault, RBAC, and right-sized compute.

---

## Cloud Architecture (Final)
### Source Systems
| Domain         | Examples                                  | Cadence       | Key Fields                   | Notes                   |
|----------------|-------------------------------------------|---------------|------------------------------|-------------------------|
| Customer Info  | profiles, segments, contact prefs         | Daily/Hourly  | customer_id, segment, status | From CRM/SaaS or SQL    |
| Driver Info    | onboarding, license, payouts              | Daily/Hourly  | driver_id, license_status, rating | HR/operations system |
| Vehicle Info   | VIN/plate, insurance, capacity            | Weekly/Daily  | vehicle_id, make/model, capacity | Registry/ERP          |
| Booking (events)| requested/accepted/pickup/dropoff/cancel | Streaming     | trip_id, state, timestamps   | App telemetry           |
| GPS Tracking   | lat/lon/speed/heading                     | Streaming     | device_id, trip_id, event_ts, lat, lon | High volume, low latency |

---

## Why These Azure Services (and Trade-offs)

### Ingestion Layer
- **Azure Data Factory**: Batch ingestion with rich connectors, retries, and parameters. Cost-efficient for non-real-time workloads.
- **Azure Event Hubs**: Handles real-time booking & GPS telemetry with high throughput.

### Storage & File Format
- **ADLS Gen2 + Delta Lake**:
  - Bronze: Raw, append-only, quarantine bad data.
  - Silver: Cleaned, validated, deduped.
  - Gold: BI-ready marts.

### Transform & Serve
- **Synapse Spark** for streaming & batch transformations.
- **Synapse SQL** for serving to BI tools.

---

## Pipelines & Orchestration
- **Master Pipeline** in ADF for end-to-end orchestration.
- **Sub-Pipelines** for ingestion, curation, and serving.
- Retry policy: exponential backoff (1m → 5m → 15m) up to 3 attempts, then alert.

---

## Access Patterns & BI Modes
- **Direct Lake**: For near real-time analytics.
- **Import Mode**: For small, curated datasets with sub-second visuals.
- **DirectQuery**: For live source querying when duplication is not desired.

---

## Security & Governance
- Private endpoints.
- RBAC with Azure AD.
- Secrets in Key Vault.
- Data classification & lineage via Purview.

---

## Data Quality Playbook (Silver)
- Schema & partition validation.
- Deduplication rules.
- Geo-sanity checks.
- Quarantine for bad rows.

---

## Cost Management
- Autoscale compute.
- Optimize & vacuum Delta tables.
- Use Serverless SQL for ad-hoc queries.

---

## Improvements Across Phases
1. Phase 1: Direct BI on raw data.
2. Phase 2: Partial separation of processing & storage.
3. Final: Full medallion architecture with clear serving layers.

---

## Example KPIs & Dashboards
- On-time Pickup %
- Driver Utilization
- Average Wait Time
- Revenue per KM
- Heatmaps for demand hotspots

---



## Implementation Steps (Walkthrough)
1. Provision Azure resources.
2. Secure network & data.
3. Implement batch & streaming ingestion.
4. Curate data in Silver layer.
5. Build Gold marts.
6. Serve to Power BI.
7. Set up monitoring & governance.

---

**📌 Note:** Replace `images/architecture.png` with your actual diagram image in the `images/` folder.
