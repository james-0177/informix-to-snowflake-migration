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

```ace
database uidb end

define
variable bd date
variable ed date
variable benefits decimal
end

input
prompt for bd using "Enter the Starting Date of the Month: "
prompt for ed using "Enter the Ending Date of the Month: "
end

output 
left margin 0
right margin 0
top margin 0
report to "/home/jp/public_html/WorkloadM.html"
end

select rptdate,
sum (c1) c1,
sum (c2) c2,
sum (c3) c3,
sum (c4) c4,
sum (c5) c5,
sum (c6) c6
from ar5130
where
rptdate >= $bd and rptdate <= $ed
group by rptdate
order by 1
into temp a;

select rptdate,
sum (c1) c1,
sum (c2) c2,
sum (c3) c3,
sum (c4) c4,
sum (c5) c5,
sum (c6) c6
from ae5130
where
rptdate >= $bd and rptdate <= $ed
group by rptdate
order by 1
into temp b;

select rptdate,
sum (c1) c1,
sum (c2) c2,
sum (c3) c3,
sum (c4) c4,
sum (c5) c5,
sum (c6) c6
from ap5130
where
rptdate >= $bd and rptdate <= $ed
group by rptdate
order by 1
into temp c;

select a.rptdate,
sum (a.c1+b.c1+c.c1) uila,
sum (a.c2+b.c2+c.c2) uiha,
sum (a.c3+b.c3+c.c3) ufla,
sum (a.c4+b.c4+c.c4) ufha,
sum (a.c5+b.c5+c.c5) uxla,
sum (a.c6+b.c6+c.c6) uxha
from a, b, c
where b.rptdate = a.rptdate
and c.rptdate = a.rptdate
group by 1
order by 1
end

format

first page header

print "<HTML><HEAD><TITLE>Monthly Benefit Appeals for the Workload Report</TITLE>"
print "</HEAD>"


print "<BODY style='font-family ,: Garamond; color:#000099;'>"
print "<H1 align=center>Monthly Benefit Appeals for the Workload Report</H1>"
print "<br />"
print "<br />"

print "<TABLE border = ", ascii 34, "1", ascii 34, "align=", ascii 34, "center", ascii 34, ">"
print "<TR bgcolor =", ascii 34, "EDDDCC" , ascii 34, " Border=" , ascii 34, "1", ascii 34, "align=", ascii 34, "center", ascii 34, "style='color:#006633;'>"
print "<TH>Month Ending</TH>"
print "<TH>UI-LA</TH>"
print "<TH>UI-HA</TH>"
print "<TH>UCFE-LA</TH>"
print "<TH>UCFE-HA</TH>"
print "<TH>UCX-LA</TH>"
print "<TH>UCX-HA</TH>"
print "<TH>Total Benefit Appeals Decisions</TH></TR>"

print

on every row

let benefits = uila + uiha + ufla + ufha + uxla + uxha

print "<TR bgcolor = ", ascii 34, "D7D7D7", ascii 34, " Border=" , ascii 34, "1", ascii 34, "align=", ascii 34, "center", ascii 34,"style='color:#006633;'>"
print "<TD align=left>", rptdate,"</TD>"
print "<TD align=center>", uila using "###,##&","</TD>"
print "<TD align=center>", uiha using "###,##&","</TD>"
print "<TD align=center>", ufla using "###,##&","</TD>"
print "<TD align=center>", ufha using "###,##&","</TD>"
print "<TD align=center>", uxla using "###,##&","</TD>"
print "<TD align=center>", uxha using "###,##&","</TD>"
print "<TD align=center><b>", benefits using "###,##&","</b></TD></TR>"

print 
on last row 

print "</TABLE></BODY></HTML>"

end
```

## 3. Modern Architecture (Snowflake SQL)

```sql
--Monthly Benefit Appeals for the Workload Report--
select Month_Ending, sub.UI_LA, sub.UI_HA, sub.UCFE_LA, sub.UCFE_HA, sub.UCX_LA, sub.UCX_HA, (sub.UI_LA+sub.UI_HA+sub.UCFE_LA+sub.UCFE_HA+sub.UCX_LA+sub.UCX_HA) as Total_Benefit_Appeals_Decisions from (select ar.rptdate as Month_Ending, sum(ar.c1+ae.c1+ap.c1) as UI_LA, sum(ar.c2+ae.c2+ap.c2) as UI_HA, sum(ar.c3+ae.c3+ap.c3) as UCFE_LA, sum(ar.c4+ae.c4+ap.c4) as UCFE_HA, sum(ar.c5+ae.c5+ap.c5) as UCX_LA, sum(ar.c6+ae.c6+ap.c6) as UCX_HA
from ar5130 ar
inner join ae5130 ae on ar.rptdate = ae.rptdate
inner join ap5130 ap on ar.rptdate = ap.rptdate
where ar.rptdate between '2025-03-01' and '2026-03-31'
group by ar.rptdate
order by ar.rptdate) as sub;
```
