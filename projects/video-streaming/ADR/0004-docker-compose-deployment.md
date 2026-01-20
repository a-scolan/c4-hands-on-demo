# ADR 0004: Docker Compose for Single-Node Deployment

**Status:** Accepted

**Date:** January 19, 2026

## Context

The video streaming platform must run on a single bare-metal or virtual machine for self-hosting. Deployment orchestration options:

1. **Docker Compose** - Lightweight, local-focused, single-YAML, minimal overhead
2. **Kubernetes (K8s)** - Powerful, cluster-ready, steep learning curve, overkill for single node
3. **systemd + Docker CLI** - Raw Linux services; manual process management; fragile
4. **Docker Swarm** - Simplified K8s; deprecated by Docker Inc.; limited community support
5. **Podman + systemd** - Rootless containers; CentOS/RHEL focused; smaller ecosystem

## Decision

**Use Docker Compose as the primary deployment and orchestration mechanism for single-node self-hosted deployment.**

Docker Compose will define:
- All service containers (NGINX, API, SRS, Transcoder, Redis, PostgreSQL, MinIO)
- Inter-service networking (docker-compose network, port mapping)
- Persistent volumes for state (database, cache, video files)
- Environment variable injection and secrets management
- Health checks and restart policies

## Rationale

**Why Docker Compose for this project:**

| Criterion | Docker Compose | Kubernetes | systemd | Docker Swarm |
|-----------|---|---|---|---|
| **Single-node support** | ✓✓ Native | ✓ Works but overkill | ✓ Works | ✓ Works |
| **Learning curve** | ✓ Easy (YAML) | Steep (50+ CRDs) | Medium (Linux) | Medium |
| **File count** | ✓ 1 (docker-compose.yml) | 5–10 (YAML per service) | 7+ (systemd units) | 1 (compose) |
| **Production-ready** | ✓ Yes (small scale) | ✓✓ (enterprise) | ✓ For simple setups | Declining |
| **Community examples** | ✓✓ Vast (every app) | ✓✓ Vast (K8s-native) | Limited | Declining |
| **Scaling to multi-node** | Manual migration | ✓ Native clustering | Manual setup | Limited |
| **Operator burden** | ✓ Minimal | High (needs training) | Medium | Low |

**Key advantages:**
- **Minimal learning curve** - Team with Docker experience productive immediately
- **Single configuration file** - Entire stack in `docker-compose.yml` (readable, version-controllable)
- **Native volume management** - Persistent data via Docker volumes (automatic creation, backup-friendly)
- **Networking out-of-box** - Internal service discovery via container names (no external registry)
- **Health checks & restart** - Built-in `healthcheck` and `restart: always` policies
- **Secrets management** - `.env` files for secrets (evolve to Docker secrets or external manager later)
- **Development parity** - Dev team runs same compose file locally (not just staging/prod)
- **Zero subscription cost** - Open-source, no licensing or metered fees

**Well-suited because:**
- **Single bare-metal host** - No need for K8s orchestration across nodes
- **Fixed service count** - 7 containers; K8s shine with 50+ services
- **Predictable load** - VOD transcoding on schedule, not bursty (no auto-scaling needed)
- **Team size** - Small team, fewer operators (not 50-person platform team)

## Consequences

**Positive:**
- **Fast feedback loop** - Deploy from `docker-compose up` in seconds
- **Understandable to newcomers** - Developers familiar with Docker get full stack immediately
- **Easy backups** - Stop containers, copy volumes, restart (no etcd, cluster state complexity)
- **Local development** - Same compose file runs on laptop (docker-desktop) and production
- **Portable** - Migrate to new hardware by copying compose file + volumes
- **Cost-free** - Docker/Docker Compose open-source, no licensing

**Negative:**
- **Single point of failure** - All services on one machine (mitigated: multi-node migration later)
- **Limited scaling** - Can't add transcoder replicas across multiple hosts without major refactor
- **No built-in high availability** - No automatic failover (OK for VOD; not for real-time)
- **Storage bottleneck** - Single machine's disk limits growth (mitigated: external storage attachment)

**Neutral:**
- **Operator responsibility** - Not managed (good for learning, bad for ops-lite teams)
- **Future migration needed** - If scale demands multi-node, must migrate to K8s or Swarm (not breaking)

## Migration Path (Future)

**When to upgrade to Kubernetes:**
- Video catalog grows to >50TB (storage scaling needs)
- Concurrent viewers exceed 10,000 (streaming bandwidth scaling)
- Team expands to dedicated platform/ops team

**Upgrade strategy:**
- Kubernetes Helm chart mirrors docker-compose.yml structure (almost 1:1 translation)
- Same images, volumes, env vars work in K8s (no app code changes)
- Gradual: K8s deployment in parallel, traffic cut over incrementally

## Technology Stack

**Compose file structure:**
```yaml
version: '3.9'
services:
  nginx:        # Reverse proxy
  api:          # Node.js API
  srs:          # Streaming server
  transcoder:   # FFmpeg worker(s)
  redis:        # Queue + cache
  postgres:     # Metadata DB
  minio:        # Object storage

volumes:
  postgres_data:
  redis_data:
  minio_data:

networks:
  default:      # Internal service network
```

**Deployment:**
```bash
docker-compose up -d              # Start all services
docker-compose logs -f            # Follow logs
docker-compose scale transcoder=3 # Scale workers
docker-compose down -v            # Stop + remove volumes
```

## Related Decisions

- **ADR 0001–0003** - SRS, MinIO, FFmpeg all Docker-compatible, no custom compilation
- **ADR 0005** - HLS/DASH served from containers within compose network

## Backup & Disaster Recovery

**Strategy:**
- Daily snapshot of PostgreSQL volume (mysqldump / pg_dump)
- Incremental backup of MinIO data (Duplicacy / Restic to external NAS)
- Compose file + environment in Git for quick redeploy

**RTO (Recovery Time):** ~15 min (restore volumes, `docker-compose up`)  
**RPO (Recovery Point):** ~1 day (daily backup cadence)

## References

- [Docker Compose Official Documentation](https://docs.docker.com/compose/)
- [Compose File Schema](https://github.com/compose-spec/compose-spec)
- [Best Practices for Compose](https://docs.docker.com/compose/compose-file/compose-file-v3/#best-practices)
- [When to use K8s vs. Compose](https://www.youtube.com/watch?v=E0LhVjQ0kCI) (Docker YouTube)
