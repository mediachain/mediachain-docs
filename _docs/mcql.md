---
title: MCQL
layout: page
category: arch
order: 10
---

MCQL is a query language for retrieving statements from the node's statement db.
It supports `SELECT` (and `DELETE`) statements with a syntax very similar to SQL, where
namespaces play the role of tables.

Some basic MCQL statements:

```sql
-- count statements in the database
SELECT COUNT(*) FROM *

-- see all namespaces in the database
SELECT namespace FROM *

-- count statements in the namespace images.dpla
SELECT COUNT(*) FROM images.dpla

-- lookup statements by media WKI
SELECT * FROM images.dpla WHERE wki = dpla_871570744a860166dba198ca95e13590

-- retrieve a sample of 5 statements from namespace
SELECT * FROM images.dpla LIMIT 5

-- retrieve the last 5 statements merged in the db
SELECT * FROM images.dpla ORDER BY counter DESC LIMIT 5

-- retrieve statement id, insertion counter tuples
SELECT (id, counter) FROM images.dpla

-- see all publishers in the namespace
SELECT publisher FROM images.dpla

-- retrieve all statements by a publisher
SELECT * FROM images.dpla WHERE publisher = 4XTTM4K8sqTb7xYviJJcRDJ5W6TpQxMoJ7GtBstTALgh5wzGm

```

The full grammar for MCQL is defined as a PEG in [the concat project](https://github.com/concat/master/mc/query/query.peg)
