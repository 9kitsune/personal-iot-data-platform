# Personal IoT & Telemetry Analytics Platform

This repository houses the blueprint and implementation for a unified, local-first IoT Data Platform. The system is designed to ingest, model, and analyse real-time streaming and batch telemetry from household sensors, vehicle diagnostics, and media consumption.

## Platform Architecture Roadmap

To manage delivery, the platform is structured into three modular phases. 

### Phase 1: TeslaMate Advanced Telemetry & Financial Pipeline (Status: 🟢 PRODUCTION)
* **Description:** A dbt-driven analytical pipeline transforming raw Postgres time-series Tesla telemetry into structured financial and battery-degradation models.
* **Key Deliverables:** Fully implemented dbt models, dynamic energy tariff API integrations, and Grafana dashboard configurations.
* **[Link to Code & Deep Dive](./teslamate-pipeline)**

### Phase 2: The Personal IoT Lakehouse (Status: 🟡 ARCHITECTURE & DESIGN ONLY)
* **Description:** An end-to-end local IoT Data Lakehouse ingestive architecture tracking real-time residential environmental telemetry using Raspberry Pi and Home Assistant.
* **Current Progress:** System architecture blueprint, schema designs, and Docker Compose configurations are finalised. Code implementation is currently paused to prioritise Phase 1 production stability.
* **[Link to Architecture RFC & Schemas](./iot-lakehouse-design)**

### Phase 3: Jellyfin Media Intelligence Platform (Status: 🔴 BACKLOG)
* **Description:** A secure ETL pipeline extracting, cleaning, and modeling unstructured media server metadata to analyse consumption habits.
* **Current Progress:** Scoping phase. Defined ingestion protocols for Jellyfin Webhooks and target dimensional models (`Fact_Playback_Session`).
