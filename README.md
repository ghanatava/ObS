# Project Onboarding: Custom Object Store

Welcome to the Custom Object Store project! This document provides an overview of the system, its architecture, core concepts, and the steps to get started.

---

## 1. Overview
We’re building a distributed, S3‑like object storage system that stores objects in chunks across multiple nodes, ensuring durability and redundancy without relying on AWS S3.

### Goals
- **Durability & Redundancy:** Ensure data survives hardware failures.
- **Scalability:** Handle petabytes of data by scaling out storage nodes.
- **Simplicity:** Provide a straightforward RESTful API for PUT/GET/DELETE operations.

---

## 2. High-Level Architecture

```
Client ⇄ API Gateway ⇄ Metadata Service ⇄ Chunk Storage Nodes
                             ↕
                       Replication & EC
```

1. **Client API**: Receives object operations (PUT/GET/DELETE).
2. **Metadata Service**: Tracks object IDs, versions, chunk IDs, and their locations.
3. **Chunk Storage Nodes**: Store and serve fixed-size chunks.
4. **Replication & Erasure Coding (EC)**: Provides redundancy.

---

## 3. Data Chunking & Storage

- **Chunk Size:** 64 MiB (configurable).
- **Chunking:** Split objects into sequential chunks.
- **Storage:** Each chunk gets a unique ID, stored on multiple nodes.

---

## 4. Metadata & Indexing

- **Persistent Store:** Use etcd or Cassandra for high availability.
- **Mappings:** `{ objectID, version } → [chunkID1, chunkID2, …]`.
- **Lookup:** On GET, metadata service returns chunk locations.

---

## 5. Redundancy & Durability

- **Replication:** Synchronous replication to N nodes (default 3 replicas).
- **Erasure Coding:** Reed‑Solomon (k data + m parity) to tolerate up to m failures.
- **Health Checks & Repair:** Periodic heartbeat; self‑healing background jobs to re‑replicate lost chunks.

---

## 6. Core Concepts & Involved Technologies

- **Consistent Hashing:** Evenly distribute chunks across nodes.
- **Quorum Reads/Writes:** Ensure strong consistency on critical operations.
- **Versioning:** Immutable object versions for snapshots and rollbacks.
- **Concurrency Control:** Leverage distributed locks (e.g., using etcd leases).
- **Monitoring & Metrics:** Instrument with Prometheus for request counts, latencies, error rates.
- **Security:** TLS for client/server communication, token‑based auth.
- **Data Placement Strategy:** Rack-awareness and failure domains.

---

## 7. Getting Started

### Prerequisites
- Go ≥ 1.20 or Java/Node.js (choose your stack)
- Docker & Docker Compose or Kubernetes (minikube)
- etcd cluster (local or hosted)

### Setup
1. **Clone the repository**
   ```bash
   git clone https://github.com/your-org/custom-object-store.git
   cd custom-object-store
   ```
2. **Configure environment** in `config/` (chunk size, replication factor, EC parameters).
3. **Start dependencies**
   ```bash
   docker-compose up -d etcd
   ```
4. **Deploy services**
   ```bash
   docker-compose up -d metadata-node storage-node
   ```
5. **Run tests**
   ```bash
   make test-integration
   ```

---

## 8. Developer Workflow

1. Create a feature branch: `git checkout -b feature/<epic>-<task>`
2. Follow coding standards in `CONTRIBUTING.md`.
3. Write unit tests for every new component.
4. Open a pull request; ensure CI passes.

---

## 9. Next Steps & Other Concepts

- **Caching Layer:** Implement edge caches via CDN.
- **Lifecycle Policies:** Auto‑expire or tier objects.
- **Object Locking:** Retention and legal holds.
- **Backup & Archive:** Integrate with cold storage backends.
- **API Versioning & Compatibility:** Maintain backward compatibility.

We’re excited to have you on board! Feel free to reach out on Slack or open an issue for questions.
