# Architecture Decision Records (ADR)

## Current State: Refactored Microservices Architecture

The Vault system evolved from a **synchronous monolithic** architecture to an **async microservices** architecture with distributed storage and fault tolerance.

---

## Key Design Decisions

### 1. API Gateway: Kong (vs. HAProxy Load Balancer)

**Decision:** Upgraded from HAProxy load balancer to Kong API Gateway

**Rationale:**
- **API-aware routing:** Kong routes based on paths, headers, JWT claims (vs. HAProxy's L4/L7 load balancing only)
- **Built-in auth:** JWT validation, OAuth2, API keys out-of-the-box (vs. HAProxy passthrough)
- **Rate limiting & circuit breakers:** Production-grade policies without custom code
- **Plugin ecosystem:** Extensible via Lua plugins for custom logic
- **Microservices gateway pattern:** Routes to multiple backend services (vs. HAProxy's single backend pool)

**Trade-off:** More complex than simple load balancing, but essential for microservices architecture

---

### 2. Storage: MinIO 3-Node Replication (Removed AWS S3)

**Decision:** MinIO as **primary** encrypted object storage with 3-node distributed replication

**Why Remove AWS S3?**
- **Cost:** S3 egress charges ($0.02/GB) prohibitive for frequent downloads
- **Complexity:** Rare disaster scenarios don't justify dual-storage architecture
- **Redundancy:** MinIO 3-node HA already provides 99.9% availability via self-healing replication
- **Recovery:** On-premises backup infrastructure (Bacula/Veeam) handles disaster recovery

**High Availability Strategy:**
- MinIO 3-node cluster with distributed erasure coding
- Self-healing: automatic re-replication on node failure
- No single point of failure

**Trade-off:** No geo-redundancy (acceptable per SLA requirements and on-premises deployment model)

---

### 3. Database: MongoDB 3-Node Replica Set

**Decision:** MongoDB instead of PostgreSQL (legacy used PostgreSQL)

**Rationale:**
- **Flexible schema** for evolving document metadata
- **Native replication** with automatic failover
- **Horizontal scaling** via sharding (future-proofing)
- Better alignment with microservices (each service owns data)

**Trade-off:** No ACID transactions across collections (mitigated by careful error handling and idempotency)

---

### 4. Async Processing: RabbitMQ Message Queue

**Decision:** Decouple upload from processing using async job queue

**Rationale:**
- **Non-blocking uploads:** Immediate response to users (improved UX)
- **Backpressure handling:** Queue absorbs traffic spikes
- **Parallel processing:** Multiple workers scale horizontally
- **Reliability:** Dead letter queue for failed jobs
- **Fail-fast validation:** Upload service validates BEFORE queuing (instant error feedback)

**Legacy Problem:** Synchronous upload → validate → scan → encrypt → store blocked user for 5-15 seconds

**Refactored Flow:**
1. Upload → validate instantly (< 100ms) → queue → immediate response (< 500ms)
2. Worker processes asynchronously: scan → encrypt → store (10-30s)
3. User notified when ready (polling or webhook)

**Why Validate at Upload Service?**
- **Fail-fast:** Invalid files rejected in < 100ms (before queuing)
- **Queue protection:** Only valid files consume worker resources
- **Better UX:** Immediate error vs. waiting 30s for async failure notification
- **Cost savings:** No virus scanning for invalid files

---

### 5. Service Consolidation: Retrieval Service (merged Document + Download)

**Decision:** Merge Document Service and Download Service into unified Retrieval Service

**Rationale:**
- **Similar responsibilities:** Both read-only operations on documents/files
- **Shared caching:** Metadata and decrypted files cached together (Redis)
- **Reduced complexity:** One service instead of two (fewer VMs, simpler networking)
- **Cache optimization:** Single cache pool for both operations (better hit rate)
- **Technology alignment:** Both use GoLang for performance

**Before (2 services):**
- Document Service (Node.js): Metadata CRUD → MongoDB
- Download Service (GoLang): File retrieval → MinIO + decrypt

**After (1 service):**
- Retrieval Service (GoLang): Metadata + files + Redis caching
  - Retrieval Handler: Unified HTTP handler for metadata queries and file retrieval
  - Cache Manager: Redis client caching frequently accessed metadata and decrypted files
  - Decryptor: AES-256 decryption service

**Benefits:**
- **50% fewer VMs** in app tier (3 instead of 4)
- **Faster response:** Shared cache for metadata queries and downloads
- **Simpler deployment:** One less service to monitor/scale
- **Better resource utilization:** Unified memory pool for caching

---

### 6. Architecture Evolution: Monolith → Microservices

**Why Refactor?**

| Legacy (Monolith) | Refactored (Microservices) |
|-------------------|---------------------------|
| Single point of failure | Isolated failures |
| Synchronous blocking operations | Async, non-blocking |
| Validation during processing | Fail-fast validation at upload |
| Difficult to scale (entire JVM) | Granular scaling per service |
| Tight coupling | Loose coupling via API/queue |
| Local disk (NFS) single point of failure | Distributed object storage (MinIO 3-node) |
| PostgreSQL + NFS | MongoDB + MinIO (better alignment) |
| HAProxy + Monolith | Kong API Gateway + 4 services |
| 1 Load Balancer (HAProxy) | 1 API Gateway (Kong) - shared infrastructure |

**Service Breakdown (5 vault-specific services + 1 shared infrastructure):**
- **Frontend:** React SPA (Node.js, static assets)
- **Upload Service:** File reception, fail-fast validation, job queuing (Node.js + Multer)
- **Retrieval Service:** Unified metadata + file downloads with Redis caching (GoLang)
- **Processing Worker:** Async scanning, encryption, storage to MinIO (GoLang)
- **Queue:** RabbitMQ message broker (job distribution)
- **Shared Infrastructure (not vault-specific):**
  - **API Gateway:** Kong (routing, auth, rate limiting - can route to multiple systems)
  - **Database:** MongoDB 3-node replica set (metadata)
  - **Storage:** MinIO 3-node cluster (encrypted files, S3-compatible)

---

## Network Architecture

**7 Network Zones (VLANs):**
1. **DMZ (VLAN 100):** Edge API gateway with TLS termination
2. **App Tier (VLAN 101):** Microservices (frontend, upload, retrieval)
3. **Processing Tier (VLAN 102):** Worker + RabbitMQ
4. **Data Tier (VLAN 103):** MongoDB + MinIO storage
5. **Security Zone (VLAN 104):** Monitoring (Prometheus, Grafana, ELK)
6. **Backup Zone (VLAN 105):** Disaster recovery (Bacula/Veeam)
7. **CI/CD Zone (VLAN 200):** GitLab, Docker Registry, Artifact Repository (with DB schema migrations)

**Firewall Strategy:** Zone-based segmentation with explicit inter-zone rules derived from service-level flows

---

## Technology Choices Summary

| Component | Technology | Why |
|-----------|-----------|-----|
| **API Gateway** | Kong | API-aware routing, auth, rate limiting (shared infra) |
| **Frontend** | React SPA | Modern UX, decoupled from backend |
| **Upload Service** | Node.js + Multer | Fast fail-fast validation before queuing |
| **Retrieval Service** | GoLang + Redis | High-performance metadata/file retrieval with caching |
| **Worker** | GoLang | CPU-intensive scanning, encryption, MinIO storage |
| **Queue** | RabbitMQ | Industry-standard, reliable, dead letter queues |
| **Database** | MongoDB 3-node | Document metadata, flexible schema, native HA |
| **Storage** | MinIO 3-node | Primary encrypted object storage (no S3), self-healing |
| **Monitoring** | Prometheus + Grafana + ELK | Open-source observability stack |
| **Encryption** | AES-256-GCM | Industry standard, authenticated encryption |
| **External Scanning** | VirusTotal API | Best-in-class virus detection (70+ engines) |
| **CI/CD** | GitLab CI + Harbor + Nexus | Complete DevOps pipeline with schema migrations |

---

## Trade-offs & Constraints

### What We Gained:
✅ **Scalability:** Independent service scaling  
✅ **Resilience:** Fault isolation, no single point of failure  
✅ **Performance:** Async processing, non-blocking uploads, caching  
✅ **Cost:** Removed expensive S3 egress fees, reduced VMs (3 instead of 4 in app tier)  
✅ **UX:** Fail-fast validation (instant error feedback)  

### What We Accepted:
⚠️ **Complexity:** More moving parts (4 services vs. 1 monolith)  
⚠️ **No Geo-Redundancy:** Single datacenter (acceptable per SLA)  
⚠️ **Eventual Consistency:** Async processing (10-30s delay)  
⚠️ **Monitoring Overhead:** Distributed tracing required  
⚠️ **Cache Invalidation:** Redis cache management complexity  

---

## Future Considerations

- **Service Mesh (Istio/Linkerd):** If inter-service communication becomes complex
- **Kubernetes:** If VM management overhead increases
- **Event Sourcing:** If audit trail requirements expand
- **Multi-Region:** If geo-redundancy becomes requirement

---

**Last Updated:** January 2026  
**Status:** Production-ready refactored architecture
