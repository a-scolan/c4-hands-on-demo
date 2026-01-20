# ADR 0006: Bare-Metal Datacenter Deployment with Scaling Architecture

**Status:** Accepted

**Date:** January 20, 2026

## Context

The video streaming platform needs deployment infrastructure that supports:
- **Initial deployment**: Single-node Docker Compose for MVP (ADR 0004)
- **Future scaling**: Multi-node expansion without complete rewrite
- **Production-grade**: Redundancy, monitoring, security boundaries
- **Cost efficiency**: Self-hosted on bare-metal to avoid cloud costs

**Scaling challenges:**
- Storage: Video files grow unbounded (10TB → 100TB+)
- Compute: Transcoding workload scales with upload volume
- Traffic: Streaming bandwidth grows with concurrent viewers
- Availability: Single point of failure unacceptable at scale

**Deployment options:**
1. **Tiered datacenter zones** - DMZ, App, Processing, Data tiers with network isolation
2. **Flat single-VLAN** - All services on same network (simpler, less secure)
3. **Cloud-native** - AWS/GCP (expensive at video scale, vendor lock-in)
4. **Hybrid cloud** - Transcoding in cloud, storage on-prem (complex data transfer)

## Decision

**Adopt a tiered bare-metal datacenter architecture with clear network zones and scaling-ready service separation.**

### Infrastructure Tiers

**Production Environment → Network Zones:**

```
Production Environment
├── DMZ Zone (VLAN 10, 10.0.10.0/24)
│   └── Edge services: NGINX reverse proxy, SSL termination
├── Application Tier (VLAN 20, 10.0.20.0/24)
│   ├── Web UI service (Nginx static file server)
│   └── API service (Node.js + Express)
├── Processing Tier (VLAN 30, 10.0.30.0/24)
│   ├── Transcoder workers (FFmpeg, horizontally scalable)
│   └── Streaming engine (SRS, segment delivery)
└── Data Tier (VLAN 40, 10.0.40.0/24)
    ├── PostgreSQL (metadata + user data)
    ├── Redis (cache + job queue)
    └── MinIO (object storage cluster)
```

### Key Architectural Decisions

**1. Network Segmentation**
- Each tier isolated in separate VLANs
- Firewall rules enforce service-to-service communication policies
- DMZ only exposed to internet (ports 80/443)
- Data tier never directly accessible from DMZ

**2. Horizontal Scaling Strategy**
- **Transcoder tier**: Add worker nodes as upload volume grows (scale-out)
- **Storage tier**: MinIO cluster mode for distributed object storage
- **API tier**: Load-balance API servers (future: add nodes behind reverse proxy)
- **Database**: PostgreSQL read replicas for query scaling

**3. Initial Deployment (Single Node)**
- Start with all tiers on one bare-metal server (Docker Compose per ADR 0004)
- Logical tier separation via Docker networks (maps to future VLANs)
- Zero downtime migration: Move tiers to dedicated hardware incrementally

**4. Storage Architecture**
- **MinIO in distributed mode** - Start single-node, expand to 4-node cluster (erasure coding)
- **NFS mount for volumes** - PostgreSQL and Redis persistence on SAN/NAS
- **Separate video storage pool** - MinIO on dedicated high-capacity disks

## Rationale

### Why Tiered Architecture?

| Criterion | Tiered Zones | Flat Network | Cloud-Native |
|-----------|--------------|--------------|--------------|
| **Security** | ✓✓ Defense in depth | Limited isolation | ✓ VPCs available |
| **Scaling clarity** | ✓✓ Tier independence | Monolithic | ✓ Auto-scaling |
| **Cost (1TB/month)** | ✓✓ $50 (bare-metal) | $50 | $200+ (S3 + compute) |
| **Migration path** | ✓ Gradual tier-by-tier | All at once | Vendor lock-in |
| **Operator control** | ✓✓ Full control | ✓ Full control | Limited (managed) |
| **Initial complexity** | Medium | Low | High |

### Scaling Progression

**Phase 1: Single Node (Months 1-6)**
- Docker Compose on single bare-metal server
- Logical network separation via Docker bridge networks
- All tiers co-located
- **Hardware**: 16-core CPU, 64GB RAM, 2TB NVMe + 10TB HDD

**Phase 2: Storage Separation (Months 6-12)**
- Move MinIO to dedicated storage nodes (4x servers with 20TB each)
- MinIO distributed mode with erasure coding (4-node cluster)
- Shared NFS for PostgreSQL + Redis volumes
- **New hardware**: 4x storage nodes (8-core, 32GB RAM, 20TB HDD each)

**Phase 3: Processing Tier Expansion (Months 12-18)**
- Add dedicated transcoder workers (3-5 nodes)
- Load-balanced behind Redis job queue
- Parallel transcoding for multiple uploads
- **New hardware**: 3x transcoder nodes (16-core, 32GB RAM, 2TB SSD each)

**Phase 4: API and Streaming Scaling (Months 18-24+)**
- Load-balanced API servers (2-3 nodes behind NGINX)
- SRS streaming cluster (origin + edge nodes)
- PostgreSQL read replicas
- **New hardware**: 2x API nodes, 2x SRS nodes, 1x DB replica

### Why Not Cloud?

**Cost comparison (10TB storage, 100 hours transcoding/month, 50TB egress):**
- **AWS**: $500/month (S3 $230 + EC2 $150 + Egress $120)
- **Bare-metal**: $150/month (hardware amortized over 3 years + electricity)
- **Savings**: $350/month = $4,200/year

**Strategic advantages:**
- No vendor lock-in (can migrate between datacenters)
- Full control over hardware specs (GPU transcoding, high-bandwidth NICs)
- No surprise egress bills (streaming traffic not metered)
- Data sovereignty (content stays in owned infrastructure)

## Consequences

### Positive Consequences

- **Scaling-ready from day 1** - Architecture accommodates growth without redesign
- **Security in depth** - Network zones enforce least-privilege access
- **Cost-effective scaling** - Add bare-metal nodes at fixed cost (no cloud markup)
- **Operational clarity** - Each tier has clear responsibility boundary
- **Migration without downtime** - Move tiers to dedicated hardware incrementally
- **Hardware flexibility** - Choose optimal specs per tier (GPU for transcoding, NVMe for DB)

### Negative Consequences

- **Initial over-engineering** - Single-node MVP runs in tiered structure (added complexity)
- **Hardware procurement lead time** - Buying servers takes weeks (cloud is instant)
- **Operator responsibility** - Team manages networking, firewalls, hardware failures
- **Datacenter dependency** - Requires reliable facility (power, cooling, connectivity)

### Neutral Consequences

- **Hybrid deployment option** - Can use cloud for transcoding bursts (keep data on-prem)
- **Complexity vs. scale** - Tiers add overhead early but pay off at scale
- **Team learning curve** - Network engineering knowledge required (not just Docker)

## Implementation Details

### Network Configuration

**VLAN and Subnet Allocation:**
```
VLAN 10 (DMZ):        10.0.10.0/24   - NGINX edge (10.0.10.10)
VLAN 20 (AppTier):    10.0.20.0/24   - WebUI (10.0.20.10), API (10.0.20.20)
VLAN 30 (ProcTier):   10.0.30.0/24   - Transcoder (10.0.30.10-50), SRS (10.0.30.60)
VLAN 40 (DataTier):   10.0.40.0/24   - Postgres (10.0.40.10), Redis (10.0.40.20), MinIO (10.0.40.30-33)
```

**Firewall Rules (Allow only necessary traffic):**
```
Internet → DMZ:       HTTPS (443), HTTP (80) - public access
DMZ → AppTier:        HTTP (8000) - API proxy
AppTier → DataTier:   PostgreSQL (5432), Redis (6379), MinIO (9000)
ProcTier → DataTier:  Redis (6379), MinIO (9000)
ProcTier → AppTier:   None (one-way: API enqueues, transcoder pulls from Redis)
```

### Docker Compose to Physical Mapping

**Phase 1 (Single Node):**
```yaml
# docker-compose.yml
networks:
  dmz:      # Maps to VLAN 10
  app:      # Maps to VLAN 20
  proc:     # Maps to VLAN 30
  data:     # Maps to VLAN 40

services:
  nginx:        networks: [dmz, app]
  api:          networks: [app, data]
  transcoder:   networks: [proc, data]
  postgres:     networks: [data]
  redis:        networks: [data]
  minio:        networks: [data]
```

**Phase 2+ (Multi-Node):**
- Replace Docker networks with physical VLANs
- Each service on dedicated VM/bare-metal in appropriate VLAN
- Inter-VLAN routing enforced by datacenter switches/firewall

## Related Decisions

- **ADR 0001**: Fixed-Bitrate Transcoding - Transcoding tier designed for predictable load
- **ADR 0002**: SRS Streaming Engine - SRS cluster mode supports edge nodes in ProcTier
- **ADR 0003**: MinIO Object Storage - MinIO distributed mode for DataTier scaling
- **ADR 0004**: Docker Compose - Phase 1 deployment uses Compose with logical tiers
- **ADR 0005**: HLS + DASH Protocols - Streaming bandwidth scales with concurrent viewers

## Future Considerations

**GPU Acceleration (Phase 5+):**
- Add GPU nodes to Processing Tier for hardware transcoding (NVENC)
- 10x transcoding throughput increase with minimal CPU overhead

**Geographic Distribution (Phase 6+):**
- Deploy edge SRS nodes in multiple locations for CDN-like delivery
- MinIO replication between datacenters for redundancy

**Kubernetes Migration (Phase 7+):**
- When managing >10 nodes, migrate to Kubernetes for orchestration
- Keep tier-based networking (maps to K8s namespaces + network policies)

## References

- [MinIO Distributed Setup](https://min.io/docs/minio/linux/operations/install-deploy-manage/deploy-minio-multi-node-multi-drive.html)
- [VLAN Design Best Practices](https://www.cisco.com/c/en/us/support/docs/lan-switching/vlan/10023-1.html)
- [SRS Cluster Configuration](https://ossrs.io/lts/en-us/docs/v5/doc/cluster)
- [PostgreSQL Replication](https://www.postgresql.org/docs/current/high-availability.html)
