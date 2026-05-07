# Data Migration Case Study: Quarterly Workload Report

## Project Overview

This case study documents the migration of the **Quarterly Workload Report**. This is a high-complexity report that aggregates data from nine different source tables to provide a comprehensive view of covered employers, audits, nonmonetary determinations, and financial recoveries.

### Logic Complexity Note

The original Informix ACE script performed its final calculations in the `FORMAT` section using procedural `let` statements. In the migration, I moved this "business logic" into the SQL layer for better performance and reusability.

---

## 1. Reverse Engineering & Logic Analysis

The reverse-engineering process required mapping a 9-way join and translating procedural "let" statements into optimized SQL expressions.

### Decoding the Legacy Logic:

* **The 9-Table Join:** The legacy script created temp tables `a` through `i` sequentially.
* **Moving Logic from Format to SQL:** In the original report, totals like `Total Fraud Recovered` were calculated line-by-line as the HTML was printed. I identified the specific Informix columns contributing to these totals and consolidated them into the SQL aggregate functions.
* **Translation Strategy:** I used a nested subquery approach to perform the heavy lifting of joining the nine tables and summing raw columns, while the outer query handles the final derived metrics.

---

## 2. Legacy Architecture (Informix ACE)

```ace
database uidb end

define
variable bd date
variable ed date
variable det decimal
variable frec decimal
variable nfrec decimal
variable rec decimal
end

input
prompt for bd using "Enter the Starting Date of the Quarter: "
prompt for ed using "Enter the Ending Date of the Quarter: "
end

output 
left margin 0
right margin 0
top margin 0
report to "/home/jp/public_html/WorkloadReportQ.html"
end

select rptdate,
sum (c3) emps,
sum (c25b) audits
from ar581
where
rptdate >= $bd and rptdate <= $ed
group by rptdate
order by 1
into temp a;

select rptdate,
sum (c1) c1det,
sum (c13) c13det,
sum (c15) c15det
from ar207
where
rptdate >= $bd and rptdate <= $ed
group by rptdate
order by 1
into temp b;

select rptdate,
sum (c1) c1det,
sum (c3) c3det,
sum (c5) c5det
from ae207
where
rptdate >= $bd and rptdate <= $ed
group by rptdate
order by 1
into temp c;

select rptdate,
sum (c1) c1det,
sum (c3) c3det,
sum (c5) c5det
from ap207
where
rptdate >= $bd and rptdate <= $ed
group by rptdate
order by 1
into temp d;

select rptdate,
sum (c206) c206,
sum (c207) c207,
sum (c278) c278,
sum (c208) c208,
sum (c209) c209,
sum (c279) c279
from ar227
where
rptdate >= $bd and rptdate <= $ed
group by rptdate
order by 1
into temp e;

select rptdate,
sum (c206) c206,
sum (c207) c207,
sum (c208) c208,
sum (c209) c209
from au227
where
rptdate >= $bd and rptdate <= $ed
group by rptdate
order by 1
into temp f;

select rptdate,
sum (c206) c206,
sum (c207) c207,
sum (c208) c208,
sum (c209) c209
from ap227
where
rptdate >= $bd and rptdate <= $ed
group by rptdate
order by 1
into temp g;

select rptdate,
sum (c206) c206,
sum (c207) c207,
sum (c278) c278,
sum (c370) c370,
sum (c372) c372,
sum (c373) c373,
sum (c208) c208,
sum (c209) c209,
sum (c279) c279,
sum (c374) c374,
sum (c376) c376,
sum (c377) c377
from am227
where
rptdate >= $bd and rptdate <= $ed
group by rptdate
order by 1
into temp h;

select rptdate,
sum (c206) c206,
sum (c207) c207,
sum (c278) c278,
sum (c370) c370,
sum (c371) c371,
sum (c372) c372,
sum (c373) c373,
sum (c208) c208,
sum (c209) c209,
sum (c279) c279,
sum (c374) c374,
sum (c375) c375,
sum (c376) c376,
sum (c377) c377
from af227
where
rptdate >= $bd and rptdate <= $ed
group by rptdate
order by 1
into temp i;

select a.rptdate,
sum (a.emps) emps,
sum (a.audits) audits,
sum (b.c1det+c.c1det+d.c1det) uidet,
sum (b.c13det+c.c3det+d.c3det) ufdet,
sum (b.c15det+c.c5det+d.c5det) uxdet,
sum (e.c206+f.c206+g.c206+h.c206+i.c206) uif,
sum (e.c207+f.c207+g.c207+h.c207+i.c207) ucf,
sum (e.c278+h.c278+i.c278) ebf,
sum (h.c370+i.c370) peucf,
sum (i.c371) puaf,
sum (h.c372+i.c372) traf,
sum (h.c373+i.c373) duaf,
sum (e.c208+h.c208+f.c208+g.c208+i.c208) uinf,
sum (e.c209+f.c209+g.c209+h.c209+i.c209) ucnf,
sum (e.c279+h.c279+i.c279) ebnf,
sum (h.c374+i.c374) peucnf,
sum (i.c375) puanf,
sum (h.c376+i.c376) tranf,
sum (h.c377+i.c377) duanf
from a, b, c, d, e, f, g, h, i
where b.rptdate = a.rptdate
and c.rptdate = a.rptdate
and d.rptdate = a.rptdate
and e.rptdate = a.rptdate
and f.rptdate = a.rptdate
and g.rptdate = a.rptdate
and h.rptdate = a.rptdate
and i.rptdate = a.rptdate
group by 1
order by 1
end

format

first page header

print "<HTML><HEAD><TITLE>Quarterly Information for the Workload Report</TITLE>"
print "</HEAD>"


print "<BODY>"
print "<H1 align=center>Quarterly Information for the Workload Report</H1>"

print "<TABLE border = ", ascii 34, "1", ascii 34, "align=", ascii 34, "center", ascii 34, ">"
print "<TR bgcolor =", ascii 34, "EDDDCC" , ascii 34, ">"
print "<TH>Quarter Ending</TH>"
print "<TH>Covered Employers</TH>"
print "<TH>Completed Audits</TH>"
print "<TH>Total Nonmonetary Determinations</TH>"
print "<TH>Total Fraud Recovered</TH>"
print "<TH>Total NonFraud Recovered</TH>"
print "<TH>Total Recovered Overpayments</TH></TR>"

print

on every row

let det = uidet + ufdet + uxdet
let frec = uif + ucf + ebf + peucf + puaf + traf + duaf
let nfrec = uinf + ucnf + ebnf + peucnf + puanf + tranf + duanf
let rec = frec + nfrec

print "<TR align = right bgcolor = ", ascii 34, "D7D7D7", ascii 34, ">"
print "<TD align=left><strong>", rptdate,"</strong></TD>"
print "<TD align=center><strong>", emps using "###,##&","</strong></TD>"
print "<TD align=center><strong>", audits using "###,##&","</strong></TD>"
print "<TD align=center><strong>", det using "###,###,##&","</strong></TD>"
print "<TD align=center><strong>", frec using "$###,###,##&","</strong></TD>"
print "<TD align=center><strong>", nfrec using "$###,###,##&","</strong></TD>"
print "<TD align=center><strong>", rec using "$###,###,##&","</strong></TD></TR>"

print 
on last row 

print "</TABLE></BODY></HTML>"

end
```

## 3. Modern Architecture (Snowflake SQL)

```sql
-- Quarterly Information for the Workload Report
with Quarterly_Metrics AS (
    select 
        a.rptdate AS Quarter_Ending,
        SUM(a.c3) AS Covered_Employers,
        SUM(a.c25b) AS Completed_Audits,
        SUM(ba.c1 + ba.c13 + ba.c15 + bb.c1 + bb.c3 + bb.c5 + bc.c1 + bc.c3 + bc.c5) AS Total_NonMon_Determinations,
        SUM(ca.c206 + cb.c206 + cc.c206 + cd.c206 + ce.c206 + ca.c207 + cb.c207 + cc.c207 + cd.c207 + ce.c207 + ca.c278 + cd.c278 + ce.c278 + cd.c370 + ce.c370 + ce.c371 + cd.c372 + ce.c372 +          cd.c373 + ce.c373) AS Total_Fraud_Recov,
        SUM(ca.c208 + cb.c208 + cc.c208 + cd.c208 + ce.c208 + ca.c209 + cb.c209 + cc.c209 + cd.c209 + ce.c209 + ca.c279 + cd.c279 + ce.c279 + cd.c374 + ce.c374 + ce.c375 + cd.c376 + ce.c376 +          cd.c377 + ce.c377) AS Total_NonFraud_Recov
    from ar581 a
    inner join ar207 ba on a.rptdate = ba.rptdate
    inner join ae207 bb on a.rptdate = bb.rptdate
    inner join ap207 bc on a.rptdate = bc.rptdate
    inner join ar227 ca on a.rptdate = ca.rptdate
    inner join au227 cb on a.rptdate = cb.rptdate
    inner join ap227 cc on a.rptdate = cc.rptdate
    inner join am227 cd on a.rptdate = cd.rptdate
    inner join af227 ce on a.rptdate = ce.rptdate
    where a.rptdate between '2024-12-01' and '2025-12-31'
    group by a.rptdate
)
select 
    *, (Total_Fraud_Recov + Total_NonFraud_Recov) AS Total_Recovered_OP
from Quarterly_Metrics
order by Quarter_Ending;
```
