# Data Migration Case Study: Monthly Workload Report

## Project Overview

This document captures the logic modernization of the **Monthly Workload Report**. The process involved migrating legacy procedural logic from an Informix DB to a set-based SQL architecture in Snowflake.

### Data Integrity Note

The migration leverages strict frontend validation. The data entry layer ensures:
* **No Missing Dates:** Every report period is accounted for.
* **No Null Values:** All metric columns are populated with a minimum value of 0, allowing for direct arithmetic without `NULL` handling functions.

---

## 1. Reverse Engineering & Logic Analysis

The primary challenge was decoding the Informix ACE syntax, which treats data retrieval and report formatting as a single sequential process.

### Decoding the Legacy Logic:
* **Sequential Flow:** The original script utilized three distinct physical temporary tables (`a`, `b`, and `c`) created in sequence.
* **Procedural Joins:** The final aggregation was performed using non-ANSI "comma-separated" joins, which were modernized into explicit `INNER JOIN` operations.
* **Presentation Decoupling:** I separated the "Pseudo-HTML" generation logic from the data retrieval logic, allowing the SQL to serve any modern BI tool.

---

## 2. Legacy Architecture (Informix ACE)

```sql
/* Informix ACE Report Logic */
DATABASE uidb END

-- Sequential Temp Table Creation
SELECT rptdate, sum(c1) c1, sum(c2) c2... FROM ar5130 
WHERE rptdate >= $bd AND rptdate <= $ed 
GROUP BY rptdate INTO TEMP a;

SELECT rptdate, sum(c1) c1, sum(c2) c2... FROM ae5130 
WHERE rptdate >= $bd AND rptdate <= $ed 
GROUP BY rptdate INTO TEMP b;

SELECT rptdate, sum(c1) c1, sum(c2) c2... FROM ap5130 
WHERE rptdate >= $bd AND rptdate <= $ed 
GROUP BY rptdate INTO TEMP c;

-- Final Aggregation
SELECT a.rptdate,
  SUM(a.c1 + b.c1 + c.c1) uila,
  SUM(a.c2 + b.c2 + c.c2) uiha...
FROM a, b, c
WHERE b.rptdate = a.rptdate AND c.rptdate = a.rptdate
GROUP BY 1 ORDER BY 1
END
