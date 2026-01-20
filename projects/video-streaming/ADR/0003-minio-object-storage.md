# ADR 0003: MinIO for S3-Compatible Object Storage

**Status:** Accepted

**Date:** January 19, 2026

## Context

VOD platform requires persistent object storage for:
- Original uploaded video files (pre-encode)
- Encoded video segments (480p, 720p, 1080p HLS/DASH)
- Manifests (.m3u8, .mpd files)
- Metadata and thumbnails

Options evaluated:
1. **MinIO** - S3-compatible API, Docker container, self-hosted, Apache 2.0 license
2. **AWS S3** - Fully managed, proven, expensive long-term ($25–100+/month for typical VOD load)
3. **IPFS/Filecoin** - Decentralized, immutable; high latency, not suitable for streaming
4. **Local filesystem + NFS** - Simple, but no versioning, replication, or disaster recovery
5. **Ceph** - Powerful, overkill for single-node; complex operations
6. **Seaweed FS** - Lightweight, newer, less battle-tested

## Decision

**Adopt MinIO (self-hosted S3-compatible) as the primary object store for video files and segments.**

MinIO will serve as the centralized artifact repository for all video content, manifests, and static assets.

## Rationale

**Why MinIO:**

| Criterion | MinIO | AWS S3 | Seaweed FS | Local NFS |
|-----------|-------|--------|-----------|-----------|
| **Cost** | ✓ Free (Docker) | $$$ (pay-per-GB) | Free | Free (1 machine) |
| **S3 API compatibility** | ✓ 100% | Native | ✓ Good | ✗ No |
| **Deployment** | ✓ Single container | Managed | Container | Sysadmin |
| **Scaling** | ✓ Cluster-ready | Automatic | ✓ FS-based | Manual (NFS bottleneck) |
| **Versioning** | ✓ Built-in | ✓ Built-in | ✗ Weak | ✗ No |
| **Replication** | ✓ Erasure coding | ✓ Automatic | ✗ Limited | ✓ Manual |
| **Maturity** | ✓ Production (2015+) | ✓✓ (AWS-backed) | Emerging (2019+) | Stable but aging |

**Key advantages:**
- **Zero cost** for self-hosted—Docker container, no licensing
- **100% S3 API compatibility** - Works with SRS, FFmpeg, any S3-client library without code changes
- **Instant scale to multi-node** - Distributed erasure coding built-in (no RAID complexity)
- **Built-in monitoring** - Prometheus metrics, web console, lifecycle policies
- **Easy migration path** - Seamless transition to AWS S3 if needed (same API, swap endpoint)
- **Operator-friendly** - Stateless containers, persistent data on Docker volumes

**For this specific use case:**
- VOD files are immutable post-transcode (MinIO versioning not critical, but nice-to-have)
- Segment lifetime is bounded (auto-delete old segments via lifecycle policies)
- Self-hosted avoids AWS storage bills ($0.023/GB = $23/month per 1TB stored)

## Consequences

**Positive:**
- **No vendor lock-in** - Same API as S3; migrate to AWS/DigitalOcean/Wasabi without code changes
- **Cost-effective at scale** - Bare-metal storage costs <$1/TB vs. AWS cloud pricing
- **Operationally simple** - Single container, standard Docker tooling, health checks built-in
- **Content delivery partnership** - Future optional: MinIO Gateway mode in front of AWS S3 for hybrid cloud
- **Industry standard** - Used by Shopify, DuckDB, Databricks for ML pipelines; proven reliability

**Negative:**
- Operational responsibility for backup/disaster recovery (AWS S3 handles this)
- Single-node MinIO limited to machine storage capacity (mitigated: multi-node expansion later)
- Bandwidth egress not metered (good for internal networks; bad if exposing to WAN)
- Community support smaller than AWS (but active, responsive Slack #minio)

**Neutral:**
- Requires Docker volume management (same as other stateful containers)
- Learning curve minimal (S3 API is de facto standard)

## Alternatives Considered & Rejected

1. **AWS S3 direct** - Cloud SaaS, zero ops, but $200+/year storage + egress costs; defeats "self-hosted" goal
2. **Seaweed FS** - Interesting project, but less battle-tested in streaming; smaller ecosystem
3. **Pure NFS** - No versioning, replication, or object metadata; doesn't scale past single network
4. **Ceph** - Powerful but 10× operational complexity for single-machine deployment

## Deployment Strategy

**Docker Compose:**
```yaml
services:
  minio:
    image: minio/minio:latest
    volumes:
      - minio_data:/data
    ports:
      - "9000:9000"  # S3 API
      - "9001:9001"  # Web console
    environment:
      MINIO_ROOT_USER: minioadmin
      MINIO_ROOT_PASSWORD: <strong-password>
```

**Lifecycle policies:**
- Delete original uploads after successful transcode (30-day retention)
- Keep encoded segments indefinitely (immutable, serves VOD forever)
- Auto-archive old manifests (daily refresh cycle)

## Related Decisions

- **ADR 0002:** SRS directly reads from MinIO segments (no proxying)
- **ADR 0004:** Docker Compose volumes persist MinIO data
- **ADR 0005:** HLS/DASH Protocol – MinIO serves manifests and segments

## Migration Path

If future scaling requires cloud:
1. Enable MinIO-to-S3 replication (native feature)
2. Switch streaming endpoint from localhost:9000 to AWS S3 endpoint
3. Retire MinIO container (data safely in AWS)

## References

- [MinIO Quickstart](https://min.io/docs/minio/container/index.html)
- [MinIO Erasure Coding](https://min.io/docs/minio/container/replication/server-side-replication.html)
- [S3 API Documentation](https://docs.aws.amazon.com/AmazonS3/latest/API/Welcome.html)
- [MinIO vs. AWS S3 Feature Comparison](https://min.io/product/feature-matrix)
