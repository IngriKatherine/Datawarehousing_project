# Group 1 project: Apache CouchDB Implementation

**DATS 6102 – Data Warehousing | Fall 2026**
**Group 1 – Apache CouchDB**

A final group project that models, loads, queries, and analyzes a [scenario, e.g., "retail orders"] dataset in **Apache CouchDB**, a JSON document database accessed over HTTP/REST. The project contrasts CouchDB's document-oriented design with a normalized relational design in **MySQL**, and compares CouchDB with another NoSQL system covered in class (**MongoDB** / **Neo4j**).

## Team

| Member | Primary Focus |
|---|---|
| Damian Mendoza | TBD [e.g., Architecture/research + technical documentation] |
| Ingri Quevedo | TBD [e.g., Installation/configuration + data loading] |
| Stephanie Armstrong | TBD [e.g., Data modeling + queries/testing] |
| Hassan Saleh H Alshamrani | TBD [e.g., MySQL comparison + integration/presentation] |

All members understand the full project and contributed to every section.

## Table of Contents

1. [Project Overview](#project-overview)
2. [Why CouchDB?](#why-couchdb)
3. [Repository Structure](#repository-structure)
4. [Setup and Installation](#setup-and-installation)
5. [Data Model](#data-model)
6. [Loading the Data](#loading-the-data)
7. [CRUD Operations](#crud-operations)
8. [Indexes and Queries](#indexes-and-queries)
9. [CouchDB-Specific Feature Demo](#couchdb-specific-feature-demo)
10. [MySQL Comparison](#mysql-comparison)
11. [NoSQL Comparison](#nosql-comparison)
12. [Strengths, Limitations, and Use Cases](#strengths-limitations-and-use-cases)
13. [Security, Deployment, and Licensing](#security-deployment-and-licensing)
14. [Demo Checklist](#demo-checklist)
15. [References](#references)

## Project Overview

- **Database:** Apache CouchDB [version]
- **Scenario:** TBD [1–2 sentences describing the business scenario]
- **Dataset:** TBD [name, source, size, number of documents]
- **Goal:** Show how a document-oriented, HTTP-based, revision-aware database handles TBD [scenario] differently than a normalized relational design.

## Why CouchDB?

CouchDB stores data as self-contained JSON documents and exposes everything through a REST API. Key concepts covered in this project:

- **Document model:** databases, documents, `_id`, and JSON structure
- **HTTP/REST interface:** all operations via `GET`, `PUT`, `POST`, `DELETE`
- **MVCC and revisions:** `_rev`-based optimistic concurrency, with no locking
- **Replication:** incremental, multi-directional, supports offline-first design
- **Querying:** Mango selectors/indexes and MapReduce views
- **Security:** authentication, admin/user roles, deployment options

## Repository Structure

```
.
├── README.md
├── docs/
│   ├── architecture.md          # CouchDB architecture and research notes
│   ├── data-model.md            # Document design and embed-vs-reference rationale
│   ├── mysql-comparison.md      # Relational mapping and comparison
│   ├── nosql-comparison.md      # CouchDB vs MongoDB/Neo4j
│   └── final-report.pdf
├── data/
│   ├── raw/                     # Original dataset
│   └── couchdb/                 # Transformed JSON documents (bulk-load format)
├── scripts/
│   ├── load_data.sh|py          # Bulk load into CouchDB
│   ├── crud_examples.sh
│   ├── queries/                 # Mango query JSON files
│   ├── indexes/                 # Index definitions
│   └── replication_demo.sh
├── mysql/
│   ├── schema.sql               # Normalized MySQL design
│   └── queries.sql              # Equivalent SQL queries
├── presentation/
│   └── slides.pdf
└── screenshots/
```

*(Adjust to match your actual layout.)*

## Setup and Installation

### Prerequisites
- Apache CouchDB [version] ([install guide](https://docs.couchdb.org/en/stable/install/index.html)) or Docker
- `curl` (and optionally `jq`, Python 3.x)

### Option A: Docker
```bash
docker run -d --name couchdb \
  -p 5984:5984 \
  -e COUCHDB_USER=admin \
  -e COUCHDB_PASSWORD=<your-password> \
  couchdb:latest
```

### Option B: Native Install
Follow the OS-specific instructions in the CouchDB docs, then start the service.

### Verify
```bash
curl http://admin:<password>@localhost:5984/
```
Fauxton (web UI): http://localhost:5984/_utils

> **Note:** Never commit real credentials. Use environment variables or a `.env` file (git-ignored).

## Data Model

**Scenario entities:** [e.g., customers, orders, products]

**Document design (example):**
```json
{
  "_id": "order:1001",
  "type": "order",
  "customer": { "id": "cust:42", "name": "Jane Doe" },
  "items": [
    { "product_id": "prod:7", "name": "Widget", "qty": 2, "price": 9.99 }
  ],
  "total": 19.98,
  "status": "shipped",
  "created_at": "2026-10-01T14:30:00Z"
}
```

**Design decisions:**
- **Embedded:** [what and why]
- **Referenced:** [what and why]
- **`_id` strategy:** [e.g., type-prefixed natural keys]

See [`docs/data-model.md`](docs/data-model.md) for details.

## Loading the Data

```bash
# Create the database
curl -X PUT http://admin:<password>@localhost:5984/<dbname>

# Bulk load
curl -X POST http://admin:<password>@localhost:5984/<dbname>/_bulk_docs \
  -H "Content-Type: application/json" \
  -d @data/couchdb/documents.json
```
Or run `scripts/load_data.*`.

## CRUD Operations

| Operation | Example |
|---|---|
| **Create** | `PUT /<db>/<docid>` or `POST /<db>` |
| **Read** | `GET /<db>/<docid>` |
| **Update** | `PUT /<db>/<docid>` with the current `_rev` |
| **Delete** | `DELETE /<db>/<docid>?rev=<rev>` |

Full scripts and outputs are in `scripts/crud_examples.*`.

## Indexes and Queries

### Index
```bash
curl -X POST http://localhost:5984/<db>/_index \
  -H "Content-Type: application/json" \
  -d '{"index":{"fields":["type","status","created_at"]},"name":"type-status-date-idx","type":"json"}'
```
**Why:** [explain the access pattern the index supports and the cost of a full scan without it]

### Queries

| # | Description | Type |
|---|---|---|
| 1 | [e.g., Find a single order by ID] | Direct document read |
| 2 | [e.g., All shipped orders after a date] | Mango, multi-field |
| 3 | [e.g., Orders by customer] | Mango / view |
| 4 | [e.g., Total sales per product] | MapReduce view |

Query definitions are in `scripts/queries/`.

## CouchDB-Specific Feature Demo

**Feature demonstrated:** [Revisions/MVCC conflict and/or Replication]

- **Revisions/MVCC:** Update a document with a stale `_rev` and show the `409 Conflict`, then resolve it.
- **Replication:** Replicate `<db>` to a second database/instance, modify data on both sides, and show sync and conflict handling.

Steps and output: `scripts/replication_demo.*` and `screenshots/`.

## MySQL Comparison

| Aspect | CouchDB | MySQL |
|---|---|---|
| Structure | Self-contained JSON documents | Normalized tables |
| Identity | `_id` / references | Primary / foreign keys |
| Relationships | Embedding or manual references | Joins with enforced FK constraints |
| Retrieval | Single document read, Mango, views | SQL `JOIN`, `GROUP BY` |
| Concurrency | MVCC via `_rev`, optimistic, per-document atomicity | ACID transactions, row locking, multi-row transactions |
| Schema | Flexible, schema-less | Fixed, enforced schema |

**Mapping:** [Describe how your documents map to tables; ERD in `mysql/` and `docs/`.]

**Conclusion:** For [scenario], [CouchDB/MySQL] is better suited because [reasons].

Details: [`docs/mysql-comparison.md`](docs/mysql-comparison.md)

## NoSQL Comparison

CouchDB vs **[MongoDB / Neo4j]** across data model, querying, relationships, consistency/transactions, scaling, and use-case fit. See [`docs/nosql-comparison.md`](docs/nosql-comparison.md).

## Strengths, Limitations, and Use Cases

**Strengths:** HTTP/JSON API, multi-master replication, offline-first sync, built-in MVCC, simple operation
**Limitations:** no multi-document transactions, no joins, limited ad hoc aggregation, conflict resolution is the application's job
**Common use cases:** offline-first mobile/web apps, distributed data sync, content/catalog stores, edge deployments

## Security, Deployment, and Licensing

- **Authentication:** [basic auth / cookie / JWT, admin vs. member roles]
- **Deployment options:** single node, cluster, Docker, cloud-hosted
- **License:** Apache License 2.0

## Demo Checklist

- [ ] Show the database and loaded JSON documents
- [ ] Show CRUD operations
- [ ] Show at least four meaningful queries/operations (incl. one multi-field retrieval)
- [ ] Show the index/query structure
- [ ] Show a CouchDB-specific capability (revisions or replication)

## References

- Apache CouchDB Documentation: https://docs.couchdb.org
- [Dataset source]
- [Other references]

## License

Course project for DATS 6102. [Add license if desired, e.g., MIT.]
