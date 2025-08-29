# readme[README_OSS_Genio_Public.md](https://github.com/user-attachments/files/22052647/README_OSS_Genio_Public.md)

# Open Data Platform (OSS) + Genio — for Oil & Gas

**One line:** A modular, open-source data platform (Kafka → Iceberg → Trino → dbt → GE → OpenLineage) with **Genio** on top for memory-first analytics, summarization, QA, and retrieval—built to reduce license spend, remove lock-in, and respect OT/SCADA safety.

---

## Why this exists (grounded)

- **Cut run‑rate:** Replace big proprietary SKUs with OSS where it’s mature; keep a few commercial tools only where they still earn their keep.  
- **Open formats:** Iceberg tables + Kafka APIs mean portability across clouds/engines.  
- **Ops reality:** Jobs-as-code (Airflow/dbt), clear SLOs, and rollback criteria—no big-bang migrations.  
- **OT safety:** Analytics mirrors are **read-only**; we do **not** touch control paths.

---

## High‑level architecture (how the pieces fit)

```
[ DBs ]  [ Files/SFTP ]  [ SCADA Mirrors ]             (read-only taps)
   |            |                 |
   |            |                 v
   |        NiFi/SFTPGo       Telegraf/OPC-UA  →  Kafka/Redpanda  →  Iceberg Tables on MinIO (Nessie catalog)
   |                                 |                                ^
   |                                 v                                |
   +-------- Debezium CDC ---------->+--------- Kafka Connect ---------+
                                              (Iceberg Sink)

           Trino SQL  ← dbt (ELT)  ← Airflow (or Dagster)  → Great Expectations (DQ gates)
                               \→ OpenLineage (Marquez) for lineage & run history

                      Superset / APIs / Notebooks
                                  ^
                                  |
                               Genio Heads/Loops (NOW→EXPRESS→INTERPRET→REFLECT→VISUALIZE→EMBED→REPLAY)

  Observability: Prometheus + Grafana + Loki + Tempo       Security: Keycloak (SSO) + Vault (secrets) + OPA (policy)
```

---

## What’s included (pick & choose modules)

> Each module is its own repo so teams can adopt selectively.

1. **Core (lakehouse)** — MinIO + Nessie + Trino + Iceberg  
   https://github.com/genioCE/g-oss-core

2. **CDC** — Redpanda/Kafka + Connect (Debezium + Iceberg sink) + Apicurio  
   https://github.com/genioCE/g-oss-cdc

3. **Batch/ELT** — Airflow + dbt + Great Expectations  
   https://github.com/genioCE/g-oss-batch

4. **Governance/Lineage** — OpenLineage (Marquez)  
   https://github.com/genioCE/g-oss-governance

5. **Observability** — Prometheus + Grafana + Loki + Tempo + OTel  
   https://github.com/genioCE/g-oss-observability

6. **Search** — OpenSearch + Dashboards  
   https://github.com/genioCE/g-oss-search

7. **BI** — Apache Superset (wired for Trino)  
   https://github.com/genioCE/g-oss-bi

8. **Security** — Vault + Keycloak + OPA  
   https://github.com/genioCE/g-oss-security

9. **MFT** — SFTPGo + NiFi (file ingress → S3/MinIO)  
   https://github.com/genioCE/g-oss-mft

10. **Streaming compute** — Apache Flink (+ Kafka)  
    https://github.com/genioCE/g-oss-stream

> A **meta‑repo** (`g-oss-platform`) can pin compatible versions, and a **client overlay** can lock environment values.

---

## Where **Genio** fits

**Genio** is our memory‑first layer that runs on the open data plane:

- **Heads/Loops:** entity extraction, summarization, QA, retrieval—aligned to the loop  
  `NOW → EXPRESS → INTERPRET (↻ PRUNE) → REFLECT (↻ TRUTH) → VISUALIZE → EMBED → REPLAY`.
- **Value:** With the plumbing open and auditable (Iceberg, Trino, Kafka, dbt), Genio focuses on cognition and automation—not proprietary data movement.
- **Industry packs:** EnergyPack for O&G; others later.

---

## Three on‑ramps (choose your speed)

- **POC (Docker Compose):** Run **Core + CDC + Batch** locally to prove SLOs and lineage.  
- **Kubernetes modules:** Deploy selected modules to AKS/EKS/GKE with Helm; GitOps with Argo CD/Flux.  
- **Azure Marketplace (preview):** CNAB‑packaged **Kubernetes apps** (AKS Cluster Extensions) so enterprises can click‑to‑install.

---

## SLOs & KPIs (defaults you can adopt)

| Metric                     | Phase 1         | Phase 2         | Notes                                                   |
|---                         |---:             |---:             |---                                                     |
| Freshness (batch)          | ≤ 2h            | ≤ 30m           | Source close → availability                            |
| Stream latency (P95)       | ≤ 60s           | 10–30s          | CDC/stream end‑to‑end                                  |
| Pipeline success rate      | ≥ 99.5%         | ≥ 99.9%         | Excluding planned maintenance                          |
| MTTR (failed run)          | < 30m           | < 15m           | Runbooks + on‑call                                     |
| Data completeness          | ≥ 99.7%         | ≥ 99.9%         | Counts/continuity                                      |
| Data accuracy              | ≥ 99.5%         | ≥ 99.8%         | Ranges, referential integrity                           |
| RPO / RTO                  | ≤ 5m / ≤ 30m    | ≤ 1m / ≤ 10m    | Kafka retention + automation                           |
| Value KPIs                 | Licenses removed, $/TB, time‑to‑provision, adoption (MAU) |  | Track at exec level |

---

## Security & compliance (baseline we hold)

- **SSO everywhere** (Keycloak/OIDC), **secrets in Vault**, **policy as code** (OPA/Rego).  
- **No secrets in repos**; SBOMs + image signing recommended.  
- **OT safety:** strictly **read‑only mirrors** from SCADA/historians; no control writes.  
- **Auditability:** OpenLineage + Git + Nessie table commits = traceable data state.

---

## Migration playbook (no big‑bangs)

1) **Mirror** (shadow) 3–5 flows beside current tools; prove SLOs.  
2) **Parallel run** with DQ/variance checks; publish lineage.  
3) **Cutover** by domain; keep a rollback path for each.  
4) **Retire** licenses with sign‑offs; capture savings → **fund Genio**.

---

## Azure Marketplace (for procurement)

We provide **CNAB skeletons** for all modules (Porter + umbrella Helm, ARM + createUiDefinition for AKS Cluster Extensions). Image digests, unpacked subcharts, and billing labels are already accounted for. Use Partner Center to create **Kubernetes app** offers and attach the CNAB from your ACR.

---

## Roadmap (rolling 12 months)

- **Now:** Core/CDC/Batch/Observability GA; Governance & Security hardening.  
- **Next:** TSDB sidecar patterns (VictoriaMetrics/QuestDB), OpenMetadata integration, more OPC‑UA/MQTT blueprints.  
- **Later:** Deeper Genio ↔ Observability hooks, managed offerings, additional industry packs.

---

## How to get started

- **Evaluate:** clone the repos, run the Compose POC (**core + cdc + batch**).  
- **Select:** pick 1–2 domains for a 90‑day pilot; adopt the SLO/KPI table above.  
- **Plan:** define rollback criteria, access model (SSO/Vault), and budget the parallel run.  
- **Execute:** ship small cutovers, show savings, and expand.

---

## Support & licensing

- **License:** Apache‑2.0 across modules unless stated otherwise (some dependencies may use permissive non‑Apache licenses—see each repo).  
- **Support options** (typical):  
  - **Community:** best‑effort in GitHub issues.  
  - **Commercial:** Bronze (business‑hours), Silver (24×5), Gold (24×7) with defined response SLAs.

---

## Contributing

PRs welcome. Please include:
- A test plan (how you validated changes),  
- Docs updates (README/EXEC_SUMMARY),  
- Security notes (if secrets/permissions changed).

---

## Contact

- General inquiries / partnership: brian@hewesguyen.com  
- Security: brian@hewesguyen.com  
- Enterprise pilots: jerry@hewesguyen.com

---

### TL;DR for execs

Start with **Core + CDC + Batch**. Mirror a handful of pipelines, show **freshness + lineage**, and **cut licenses**. Use those savings to fund **Genio**—the part that actually improves decision speed and quality.
