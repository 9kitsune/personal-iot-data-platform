# Personal IoT & Telemetry Analytics Platform

This repository houses the end-to-end architecture blueprints, infrastructure configurations, and transformation code for a unified, local-first IoT Data Platform. The system is designed to ingest, clean, and model high-frequency transactional data and real-time streaming telemetry from household sensors, connected vehicles, and local applications.

---

## 🛠️ Platform Architecture Roadmap

To manage delivery and simulate enterprise product lifecycles, the platform's features are categorized into modular deployment phases.

### Phase 1: Baby Health & Milestone Analytics (Status: 🟢 PRODUCTION)
* **Description:** A self-hosted event-driven pipeline running on a local server. It captures real-time infant health telemetry (feeds, sleep windows, diaper logs) via the Baby Buddy Django REST API, backed by a transactional PostgreSQL database. It handles secure remote edge ingestion via a WireGuard overlay network.
* **Key Deliverables:** 
  * Production Docker Compose orchestration stacks.
  * Complex dbt Core SQL models handling **sleep sessionization** (calculating wake windows across midnight boundaries) and **metric normalization** (volumetric data handling).
  * Real-time user interface deployment optimized for zero-friction mobile logging.
* **[Link to Production Code & dbt Models](./baby-analytics-platform)**

### Phase 2: TeslaMate Advanced Telemetry & Financial Pipeline (Status: 🟡 ARCHITECTURE & DESIGN ONLY)
* **Description:** A dbt-driven analytical pipeline designed to transform raw PostgreSQL time-series vehicle telemetry into structured battery-degradation models and dynamic charging cost analytics.
* **Current Progress:** Schema designs and external API mapping profiles are finalized. Code execution is paused to prioritize Phase 1 production stability.
* **[Link to Architecture RFC & Schemas](./teslamate-pipeline-design)**

### Phase 3: The Personal IoT Environmental Lakehouse (Status: 🔴 BACKLOG)
* **Description:** An end-to-end local data lakehouse architecture tracking real-time residential environmental telemetry using a Raspberry Pi grid and Home Assistant.
* **Planned Stack:** MQTT / DuckDB / Prefect Orchestration / Evidence.dev.
* **[Link to Design Scope](./iot-lakehouse-backlog)**

### Phase 4: Jellyfin Media Intelligence Platform (Status: 🔴 BACKLOG)
* **Description:** A Python-based ETL pipeline extracting, cleaning, and modeling unstructured media server metadata to analyze domestic consumption habits and content demand profiles.
* **Planned Stack:** Python (Polars) / JSON parser webhooks / Dimensional modeling (`Fact_Playback_Session`).
* **[Link to Scoping Document](./jellyfin-intelligence-backlog)**

---

## 🔒 Infrastructure, Security & Network Topology

A core constraint of this data platform is total local privacy and data sovereignty. Direct exposure to the public internet is eliminated. Instead, remote edge ingestion (e.g., logging infant data outside the home network) is achieved via an **encrypted WireGuard overlay mesh network**. 

All data streams are ingested through secure split-tunneling routing profiles, ensuring zero-latency handshakes and zero battery overhead on edge mobile devices.
