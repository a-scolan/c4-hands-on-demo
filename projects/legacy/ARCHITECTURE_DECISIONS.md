# Legacy Vault System - Architecture Decisions

This document outlines the key architectural decisions for the **Legacy (Monolithic) Vault System** and provides rationale for each choice.

## Overview

The Legacy Vault represents a **monolithic, synchronous architecture** optimized for simplicity and operational familiarity. It uses load balancing for availability, shared storage for centralized file management, and traditional database replication for data consistency.

## Key Architecture Decisions

### 1. Monolithic Application Design

**Decision:** Build vault functionality as a single Spring Boot application rather than microservices.

**Rationale:**
- **Operational simplicity:** Single deployment unit, unified logging, easier monitoring
- **Development speed:** All functionality in one codebase, shared libraries
- **Transactional consistency:** ACID transactions across vault operations (file storage, metadata, encryption keys)
- **Team alignment:** Suitable for smaller teams working on a single product

**Trade-off:**
- Limited horizontal scaling (can only scale whole app, not individual services)
- Tight coupling between modules (file upload, encryption, scanning, retrieval)
- All dependencies (Spring Boot, PostgreSQL, NFS) run on same servers

**When this works best:**
- Smaller user bases (< 1M requests/day)
- Relatively stable team size
- Monolithic code repositories are acceptable
- Operational overhead is managed by dedicated DevOps team

---

### 2. Load Balancing with HAProxy

**Decision:** Use HAProxy as the load balancer and reverse proxy.

**Rationale:**
- **Open source:** No licensing costs, widely used in production
- **Reliable:** Proven track record in high-traffic environments
- **Performance:** Efficient layer 4/7 load balancing, low latency
- **Flexibility:** Easy configuration, support for multiple load balancing algorithms
- **Monitoring:** Rich metrics and logs for operations team

**Architecture:**
```
┌─────────────────────────────────────┐
│         Internet / Users            │
└────────────┬────────────────────────┘
             │
    ┌────────▼────────┐
    │   HAProxy LB    │ (Prod.LB, port 443)
    │ (External, DMZ) │
    └────────┬────────┘
             │
    ┌────────┴────────┐
    │                 │
┌───▼───┐         ┌───▼───┐
│Mono-01│         │Mono-02│ (AppTier, internal)
└───────┘         └───────┘
```

**Failure scenarios handled:**
- If Monolith-01 fails → traffic routes to Monolith-02
- If HAProxy fails → single point of failure (mitigation: use a second HAProxy with failover, not implemented in this model)

---

### 3. NFS Shared Storage

**Decision:** Use NFS (Network File System) for shared encrypted file storage.

**Rationale:**
- **Consistency:** All app instances see identical file state immediately
- **Simplicity:** No need for distributed object storage (MinIO, S3)
- **Availability:** Both monoliths can access same files simultaneously
- **Operations:** Standard POSIX file semantics, familiar to ops teams

**Architecture:**
```
┌─────────────┐    ┌─────────────┐
│ Monolith-01 │    │ Monolith-02 │ (AppTier)
└──────┬──────┘    └──────┬──────┘
       │                  │
       └──────────┬───────┘
                  │ NFS (port 2049)
                  │
            ┌─────▼────────┐
            │ NFS Storage  │ (DataTier)
            │ (Prod.Nfs)   │
            └──────────────┘
```

**Trade-offs:**
- Performance: NFS latency higher than local disk
- Single point of failure: NFS server down → no file access
- No built-in geo-distribution (unlike S3/MinIO)
- Scalability: NFS performance degrades with many clients

**Why not S3/MinIO?**
- Architectural simplicity (fewer technologies to manage)
- Operational cost (self-hosted NFS cheaper than object storage cluster)
- Team familiarity (most Linux teams understand NFS)

---

### 4. PostgreSQL with Manual Replication

**Decision:** Use PostgreSQL as primary database with manual replication for HA.

**Rationale:**
- **ACID guarantees:** Critical for vault transaction integrity (file metadata, encryption state)
- **Mature ecosystem:** Wide adoption, extensive tooling, well-documented patterns
- **Complex queries:** Support for sophisticated queries (logs, audit trails, reports)
- **Triggers & views:** Support for sophisticated business logic at database layer

**Replication Strategy:**
```
┌──────────────┐
│ PostgreSQL   │ (Primary, Prod.Database)
│ Port 5432    │ (DataTier, 10.0.3.10)
└──────┬───────┘
       │ Streaming Replication
       │
┌──────▼──────────────┐
│ PostgreSQL Replica  │ (Standby)
│ Port 5432           │ (Optional in model)
└─────────────────────┘
```

**Manual replication implies:**
- Database administrator manually promotes replica to primary if primary fails
- No automatic failover (unlike RDS, Patroni, or Postgres Operator)
- Suitable for lower-traffic systems where operator response time is acceptable

**Failure mode:**
- Primary down → operations team manually promotes replica
- Recovery time objective (RTO): Minutes to hours depending on alerting and response

**Why PostgreSQL over:**
- **MySQL:** Similar capabilities; PostgreSQL has better JSON, geo-spatial support
- **NoSQL (MongoDB, DynamoDB):** Monolithic vault needs ACID, complex queries
- **Managed RDS:** Cost considerations, and desire to keep infrastructure simple

---

### 5. Synchronous Request-Response Pipeline

**Decision:** Use synchronous HTTP request/response for all operations.

**Rationale:**
- **Operational simplicity:** Easy to understand flow, synchronous debugging
- **Error handling:** Clear error feedback to client (success/failure immediately)
- **Consistency:** Client knows operation complete before receiving response
- **Monitoring:** Transaction latency directly observable

**Pipeline for file upload:**
```
Client → HAProxy → Monolith → Validator → Encryptor → NFS → PostgreSQL → Response → Client
         (sync, ~500-1000ms)
```

**Trade-off:**
- Limited concurrency: If scan takes 10 seconds, user waits 10 seconds
- No asynchronous processing: VirusTotal scans block upload completion
- Scalability: Each request ties up application resources for entire duration

**When this works best:**
- Smaller user volumes (< 100 concurrent users)
- Operations are relatively fast (< 5 seconds)
- Operational teams prefer synchronous debugging

---

### 6. VirusTotal Integration

**Decision:** Call VirusTotal API for real-time file scanning.

**Rationale:**
- **Security:** Third-party threat intelligence for zero-day detection
- **Operational responsibility:** Outsource scanning complexity to specialist service
- **No infrastructure:** No need to maintain local scanning engines (ClamAV, etc.)

**Architecture:**
```
Monolith → (HTTPS) → Internet → VirusTotal API → Response → Client
          (port 443)
```

**Implications:**
- Synchronous blocking call (user waits for scan result)
- VirusTotal availability affects vault availability
- Dependency on external service (potential SLA violation)
- Cost: Per-API-call billing

---

### 7. DMZ Network Architecture

**Decision:** Use three-tier network with DMZ, Application, and Data tiers.

**Rationale:**
- **Security:** External traffic only reaches HAProxy in DMZ
- **Isolation:** Database layer isolated from external access
- **Firewall rules:** Clear ingress/egress rules between tiers

**Network layout:**
```
┌──────────────────────────────────────────────┐
│             External Internet                │
└────────────────┬─────────────────────────────┘
                 │ Port 443 (HTTPS)
         ┌───────▼────────┐
         │ DMZ (Zone_DMZ) │ VLAN 99
         │    HAProxy     │ 10.0.0.0/24
         └───────┬────────┘
                 │ Internal Routes
         ┌───────▼──────────┐
         │AppTier (Zone)    │ VLAN 100
         │ Monolith-01,-02  │ 10.0.1.0/24
         └───────┬──────────┘
                 │ Internal Routes
         ┌───────▼──────────┐
         │DataTier (Zone)   │ VLAN 101
         │PostgreSQL, NFS   │ 10.0.2.0/24
         └──────────────────┘
```

**Firewall rules:**
- External → DMZ: Port 443 only (HTTPS)
- DMZ → App: Port 8080 (app), 2049 (NFS)
- App → Data: Port 5432 (PostgreSQL), 2049 (NFS)
- Data → External: None (except VirusTotal outbound)

---

### 8. Deployment to Single Environment

**Decision:** Deploy to single production environment (Prod).

**Rationale:**
- **Simpler operations:** One environment to manage, monitor, patch
- **Cost:** Minimal infrastructure (2 app VMs, 1 storage, 1 DB)
- **Suitable for:** Smaller organizations, early-stage products

**Trade-off:**
- No staging environment (risk of untested changes)
- No dev environment (shared development instance or local dev only)
- Blue-green deployment not possible without infrastructure expansion

---

## Comparison with Refactored Architecture

| Aspect | Legacy | Refactored |
|--------|--------|-----------|
| **App Design** | Monolithic | Microservices |
| **Load Balancing** | HAProxy | Kong API Gateway |
| **File Storage** | NFS | MinIO (S3-compatible) |
| **Database** | PostgreSQL (sync replication) | MongoDB (sharded) |
| **Async Processing** | Synchronous (blocking) | RabbitMQ + Worker services |
| **Scaling** | Vertical only | Horizontal services |
| **Deployment** | 1 environment | 1 environment (same) |
| **Complexity** | Lower | Higher |
| **Operational Overhead** | Lower | Higher (more services) |
| **Concurrency** | Limited | Higher (async workers) |
| **File Scanning** | Synchronous VirusTotal calls | Asynchronous worker queues |

---

## Operational Model

### Deployment Model

The legacy system deployed to a 3-tier network:

1. **DMZ Zone (VLAN 99)**
   - HAProxy load balancer
   - Public internet facing
   - Strictly limited firewall rules

2. **Application Tier (VLAN 100)**
   - Monolith instances (Monolith-01, Monolith-02)
   - Internal only
   - Receives traffic from HAProxy

3. **Data Tier (VLAN 101)**
   - PostgreSQL database
   - NFS storage server
   - No direct external access

### VM Configuration

| VM | Role | Network | CPU | RAM | Storage | OS |
|----|------|---------|-----|-----|---------|-----|
| prod-lb-vm | HAProxy | 10.0.0.10 | 4 cores | 8 GB | 50 GB | Ubuntu 20.04 |
| prod-monolith-vm-01 | Spring Boot App | 10.0.1.10 | 8 cores | 16 GB | 100 GB | Ubuntu 20.04 |
| prod-monolith-vm-02 | Spring Boot App | 10.0.1.11 | 8 cores | 16 GB | 100 GB | Ubuntu 20.04 |
| prod-nfs-vm | NFS + Database | 10.0.2.10 | 8 cores | 16 GB | 500 GB | Ubuntu 20.04 |

### Technology Stack

**Frontend:**
- Web browser (HTTPS)

**Application Server:**
- Spring Boot 2.7 (Java 11+)
- Spring Security (encryption, authentication)
- Spring JPA (database access)

**Storage:**
- NFS server (file storage)
- PostgreSQL 12+ (relational database)

**Networking:**
- HAProxy 2.x (load balancing, reverse proxy)
- OpenSSL (HTTPS/TLS)

**Monitoring & Logging:**
- Custom logging (logs to NFS)
- Manual health checks (health endpoint on each app)
- Infrastructure monitoring (CPU, RAM, disk on each VM)

---

## Key Constraints & Assumptions

1. **Availability:** Single-server failures tolerated by load balancer; multi-zone failures would cause outage
2. **Concurrency:** Peak load assumed < 100 concurrent users
3. **Latency:** User acceptable with synchronous scanning (10+ second upload times)
4. **Compliance:** Data residency requirements met by on-premise deployment
5. **Team:** Small ops team can manage monolithic deployment
6. **Cost:** Infrastructure costs acceptable; no cost optimization required

---

## Future Evolution

If requirements change, consider migrating to refactored architecture:

1. **Scalability:** Need for > 1000 concurrent users → Microservices + async queues
2. **Geographic distribution:** Need for multi-region deployment → Distributed architecture
3. **Resilience:** Need for automatic failover → Managed services (Kubernetes, RDS)
4. **Operational maturity:** Team expertise grows → More complex orchestration

---

## Related Documents

- System Model: [system-model.c4](./system-model.c4)
- System Views: [system-views.c4](./system-views.c4)
- Deployment Model: [deployment.c4](./deployment.c4)
- Deployment Views: [deployment-views.c4](./deployment-views.c4)
- Use Case Flows: [system-sequences.c4](./system-sequences.c4)

---

**Last Updated:** 2025-01-24  
**Architecture Status:** Production
