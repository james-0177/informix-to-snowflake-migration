# Legacy Modernization: Informix to Snowflake Migration

## Project Overview

This repository showcases the end-to-end migration of a mission-critical reporting suite from a legacy **Informix DB** to a modern **Snowflake Cloud Data Warehouse**.

The project involved:

1. **Reverse Engineering:** Decoding procedural Informix ACE scripts and pseudo-HTML output logic.

2. **Data Logic Consolidation:** Translating sequential temporary table creation into high-performance, set-based SQL.

3. **Architecture Modernization:** Moving from manual, menu-driven legacy execution to an automated, modern cloud data pipeline.

## The "Workload Report" Series

The primary focus of this migration was the **Workload Report**, which is executed at different granularities (Weekly, Monthly, and Quarterly). As the reporting period expands, the complexity of the data aggregation increases significantly.

### 1. [Monthly Workload Report](./migration_case_study_monthly_workload.md)

* **Complexity:** Moderate.

* **Key Challenge:** Migrating from three sequential passes over the database into a single join logic.

* **Modernization:** Introduction of on-the-fly calculations for "Total Benefit Appeals Decisions."

### 2. [Quarterly Workload Report](./migration_case_study_quarterly_workload.md)

* **Complexity:** High.

* **Key Challenge:** Consolidating **nine distinct source tables** and moving complex financial arithmetic (Fraud vs. Non-Fraud recoveries) from the presentation layer (ACE Format) to the data layer (Snowflake SQL).

* **Modernization:** Total removal of procedural "let" variables in favor of declarative SQL expressions.

## Technical Skills Demonstrated

### Reverse Engineering

* **Procedural Decoding:** Ability to read Informix ACE `DEFINE`, `INPUT`, and `FORMAT` blocks to extract hidden business rules.

* **Legacy Schema Mapping:** Mapping cryptic column names (e.g., `c25b`, `c370`) back to their functional business definitions (e.g., `Completed Audits`, `PEUC Fraud`).

### SQL Engineering (Snowflake)

* **Set-Based Logic:** Replacing physical temporary tables with Common Table Expressions (CTEs) and optimized subqueries.

* **Performance Tuning:** Leveraging the Snowflake Optimizer through explicit `INNER JOIN` syntax and consolidated aggregation.

### Data Integrity

* **Frontend-Backplane Coordination:** Designing SQL logic that relies on strict frontend validation (preventing `NULL` values and date gaps) to ensure report accuracy and performance.

## Legacy Environment vs. Modern Stack

* **Old World:** PuTTY Terminal, WinSCP, Informix 4GL/ACE, Static HTML output to local `/public_html` directories.

* **New World:** Snowflake, ISO-Compliant SQL, Cloud Storage, and BI Tool integration.
