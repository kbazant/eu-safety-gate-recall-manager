# Architecture

## Overview

The EU Safety Gate Recall Manager uses a simple Power Platform architecture:

**EU Safety Gate → Power Automate → Dataverse ↔ Power Apps → Compliance / Warehouse User**

Microsoft Dataverse is the system of record.

## Components

### EU Safety Gate

**Role:** External source

Provides weekly product-safety alert reports in XML format.

The solution reads data from EU Safety Gate and does not write back to the source.

### Power Automate

**Component:** SG - Ingest Alerts (Daily Delta)

Responsibilities:

- Runs on a daily schedule
- Retrieves EU Safety Gate XML reports over HTTP
- Parses structured safety-alert data
- Checks Dataverse for existing alerts
- Creates new Safety Alert records or updates existing records

The flow also records ingestion activity in the Dataverse **Ingestion Run Log** table.

### Microsoft Dataverse

**Operational tables:**

- Safety Alert
- Inventory Check
- Warehouse

**Supporting table:**

- Ingestion Run Log

Responsibilities:

- Acts as the system of record
- Stores structured recall and review data
- Maintains relationships between alerts, inventory checks, and warehouses
- Supports idempotent ingestion by allowing the flow to identify existing alert records

### Power Apps

**Component:** EU Safety Gate Recall Manager

Responsibilities:

- Safety alert review
- Warehouse inventory checks
- Review outcome capture
- Operational dashboard and views

The model-driven app reads from and writes to Dataverse.

### Compliance / Warehouse User

Responsibilities:

- Reviews safety alerts
- Performs warehouse inventory checks
- Records review outcomes

The final inventory-review outcome is human-owned. No AI or automated compliance decision is used.

## Data Flow

1. The scheduled cloud flow requests EU Safety Gate weekly-report data.
2. Power Automate retrieves report details and parses XML safety-alert records.
3. The flow checks Dataverse for an existing Safety Alert.
4. Existing records are updated and new alerts are created.
5. Compliance and warehouse users review alerts through the model-driven Power App.
6. Users create Inventory Check records, associate them with a Warehouse and Safety Alert, and record the review outcome.

## Design Decisions

- **Dataverse as system of record:** Recall data and warehouse review records remain in one operational data layer.
- **Scheduled XML ingestion:** External safety data is imported automatically through Power Automate.
- **Idempotent create/update logic:** Repeated ingestion runs update existing alerts instead of creating duplicates.
- **Human-owned review outcomes:** The system structures and records the review process but does not automate the final warehouse assessment.
- **Minimal runtime architecture:** No AI, separate approval service, SharePoint, or additional datastore is required by the implemented solution.
