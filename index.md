---
generated: true
---
# Reef Index — sellflow-reef

> Auto-generated catalog. Do not edit manually.

## Systems
- [[SYS-DELIVERY]] — Delivery Lookup BFF (draft)
- [[SYS-INVENTORY]] — Inventory API Service (draft)
- [[SYS-ORDER]] — Order Service (draft)
- [[SYS-SETTLEMENT]] — Partner Settlement Batch Service (draft)
- [[SYS-SETTLEMENT-ANOMALY]] — Settlement Anomaly Detection Service (draft)

## Schemas
- [[SCH-DELIVERY]] — Delivery BFF Transient Data Shapes (draft)
- [[SCH-INVENTORY]] — Inventory Data Model (draft)
- [[SCH-ORDER]] — Order Data Model (draft)
- [[SCH-ORDER-MIGRATION-HISTORY]] — Order Schema Migration History (V1–V24) (draft)
- [[SCH-SETTLEMENT-ANOMALY]] — Settlement Anomaly Data Model (draft)
- [[SCH-SETTLEMENT-BATCH]] — Settlement Batch Data Model (draft)
- [[SCH-SETTLEMENT-FIELD-LINEAGE-SETTLEMENT-DTL]] — SETTLEMENT_DTL Field Lineage — from ORDER_MST/ORDER_DTL to the Payout (draft)

## APIs
- [[API-DELIVERY]] — Delivery BFF API Surface (draft)
- [[API-INVENTORY]] — Inventory API Surface (draft)
- [[API-ORDER]] — Order Service API (draft)
- [[API-SETTLEMENT-ANOMALY]] — Settlement Anomaly Detection API (draft)
- [[API-SETTLEMENT-BATCH]] — Settlement Batch Interface Surface (draft)

## Processes
- [[PROC-DELIVERY-AUTH]] — Delivery BFF Authentication and Authorization (draft)
- [[PROC-DELIVERY-ERROR-HANDLING]] — Delivery BFF Error Handling and Degraded Responses (draft)
- [[PROC-DELIVERY-STATUS-SYNC]] — Delivery Status Sync and Retry (draft)
- [[PROC-INVENTORY-AUTH]] — Inventory API Authentication and Authorization (draft)
- [[PROC-INVENTORY-ERROR-HANDLING]] — Inventory API Error Handling and Failure Modes (draft)
- [[PROC-INVENTORY-RESTOCK]] — Stock Restoration on Order Cancellation (draft)
- [[PROC-INVENTORY-RESTORE-LOG-LIFECYCLE]] — RESTORE_LOG Row Lifecycle (the table that has none) (draft)
- [[PROC-INVENTORY-STOCK-ITEM-LIFECYCLE]] — stock_item Row Lifecycle (draft)
- [[PROC-ORDER-AUTH]] — Order Service Authentication and Access Control (draft)
- [[PROC-ORDER-CANCEL]] — Order Cancellation Lifecycle (draft)
- [[PROC-ORDER-ERROR-HANDLING]] — Order Service Error Handling and Failure Behaviour (draft)
- [[PROC-ORDER-EVENT-OUTBOX-LIFECYCLE]] — ORDER_EVENT_OUTBOX Row Lifecycle (draft)
- [[PROC-ORDER-ORDER-CANCEL-LIFECYCLE]] — ORDER_CANCEL Row Lifecycle (draft)
- [[PROC-ORDER-ORDER-MST-LIFECYCLE]] — ORDER_MST Row Lifecycle (draft)
- [[PROC-SELLFLOW-CANCEL-MONEY-PATH]] — Cancellation Money Path End to End (active)
- [[PROC-SELLFLOW-OWNERSHIP]] — Service Ownership and Its Four Disagreeing Records (draft)
- [[PROC-SELLFLOW-RUNTIME]] — Sellflow Runtime Topology (draft)
- [[PROC-SETTLEMENT-ANOMALY-LIFECYCLE]] — SETTLEMENT_ANOMALY Row Lifecycle (draft)
- [[PROC-SETTLEMENT-ANOMALY-VS-BATCH]] — settlement-batch vs settlement-anomaly — Side-by-Side (draft)
- [[PROC-SETTLEMENT-AUTH]] — Settlement Authentication and Access Control (draft)
- [[PROC-SETTLEMENT-CANCEL-RECON-QUEUE-LIFECYCLE]] — CANCEL_RECON_QUEUE Row Lifecycle (draft)
- [[PROC-SETTLEMENT-CORRECTION]] — Settlement Correction Workflow (draft)
- [[PROC-SETTLEMENT-CORRECTION-PROCEDURE-V03-VS-V11]] — 정산 정정 업무절차 — v0.3 vs v1.1, a Documentary Diff (draft)
- [[PROC-SETTLEMENT-DAILY-BATCH]] — Daily Partner Settlement Batch (draft)
- [[PROC-SETTLEMENT-ERROR-HANDLING]] — Settlement Batch Error Handling and Recovery (draft)
- [[PROC-SETTLEMENT-FLOW-CATALOG]] — Settlement Flow Catalog (draft)
- [[PROC-SETTLEMENT-RUN-LIFECYCLE]] — SETTLEMENT_RUN and SETTLEMENT_DTL Lifecycle (draft)
- [[PROC-SETTLEMENT-RUN-LOG-LIFECYCLE]] — SETTLEMENT_RUN_LOG Lifecycle — Created by Migration, Never Populated (draft)

## Decisions
- [[DEC-DELIVERY-GENERATED-CLIENT]] — Committing a Generated Order Client (draft)
- [[DEC-INVENTORY-RESTOCK-BY-REASON]] — Restock Conditional on the Cancel Reason Code (draft)
- [[DEC-ORDER-OUTBOX-RELAY]] — Database Outbox for Settlement Notification (draft)
- [[DEC-ORDER-SF2287-CANCEL-BLOCK-REMOVAL]] — SF-2287 — Removing the Settled-Order Cancel Block (draft)
- [[DEC-SELLFLOW-2026-AUTOMATION-SIZING]] — Sizing the 2026 Correction-Automation Programme on a 2023 Forecast (draft)
- [[DEC-SELLFLOW-MONEY-ROUNDING]] — Divergent Money Rounding Between Order and Settlement (draft)
- [[DEC-SELLFLOW-SHARED-DB]] — Shared MySQL Instance Across Services (draft)
- [[DEC-SETTLEMENT-ANOMALY-STANDALONE]] — Anomaly Detection as a Standalone Service on Another Team's Tables (draft)
- [[DEC-SETTLEMENT-CANCEL-CLAWBACK]] — Next-Month Clawback as Compensation for Post-Settlement Cancels (draft)

## Glossary
- [[GLOSSARY-DELIVERY]] — Delivery Domain Glossary (draft)
- [[GLOSSARY-INVENTORY]] — Inventory Domain Glossary (draft)
- [[GLOSSARY-ORDER]] — Order Domain Glossary (draft)
- [[GLOSSARY-SELLFLOW]] — Sellflow Unified Glossary (draft)
- [[GLOSSARY-SETTLEMENT]] — Settlement Domain Glossary (draft)
- [[GLOSSARY-SOURCE-INDEX]] — Sellflow Term to Source Index (draft)

## Contracts
- [[CON-ORDER-DELIVERY]] — Order ↔ Delivery Client Contract (draft)
- [[CON-ORDER-INVENTORY]] — Order ↔ Inventory Restock Contract (draft)
- [[CON-ORDER-SETTLEMENT]] — Order ↔ Settlement Cancellation Contract (draft)
- [[CON-SELLFLOW-CANCEL-ENTITY-COMPARISON]] — "A Cancellation" Across Five Services — Entity Comparison (draft)
- [[CON-SETTLEMENT-ANOMALY-SETTLEMENT-BATCH]] — Settlement-Anomaly ↔ Settlement-Batch Table-Read Contract (draft)

## Risks
- [[RISK-DELIVERY]] — Delivery BFF Known Risks (draft)
- [[RISK-INVENTORY]] — Inventory API Known Issues (draft)
- [[RISK-ORDER]] — Order Service Known Issues (draft)
- [[RISK-ORDER-DISABLED-TESTS]] — Order Service Disabled and Non-Executing Tests (draft)
- [[RISK-SELLFLOW-DOC-DRIFT]] — Documentation Drift Across Sellflow (draft)
- [[RISK-SETTLEMENT]] — Settlement Service Risk Themes (draft)
- [[RISK-SETTLEMENT-ANOMALY]] — Settlement Anomaly Service Risks (draft)
- [[RISK-SETTLEMENT-RECON-BACKLOG]] — Cancel Reconciliation Queue Backlog (draft)

## Patterns
- [[PAT-SELLFLOW-CROSS-SERVICE-AUTH]] — Four Services, Four Postures, No Service-to-Service Authentication (draft)
- [[PAT-SELLFLOW-CROSS-SERVICE-DATA-FLOW]] — The Real Integration Surface Is the Table, Not the API (draft)
- [[PAT-SELLFLOW-DB-AS-QUEUE]] — The Database Is the Queue — Polled Tables Instead of a Broker (draft)
- [[PAT-SELLFLOW-DOC-CODE-DRIFT]] — How Documents and Code Drift Apart (draft)
- [[PAT-SELLFLOW-ORDER-CANCEL-DIVERGENCE]] — One Concept, Five Models — How "An Order Cancellation" Fragmented (draft)
- [[PAT-SELLFLOW-ORPHANED-COMPONENTS]] — Orphaned Components — Code and Schema With No Caller, Writer or Scheduler (draft)
- [[PAT-SELLFLOW-ROMANISED-NAMING]] — Romanised Korean Naming in Schemas and Code (draft)
