# ADR 0002: SRS as Primary Streaming Engine

**Status:** Accepted

**Date:** January 19, 2026

## Context

A self-hosted VOD platform requires a streaming server capable of:
- HLS and DASH protocol support
- Segment serving from object storage (MinIO)
- Manifest generation and caching
- Low CPU overhead for segment delivery

Key candidates:
1. **SRS (Simple RTMP Server)** - C++, MIT license, lightweight, clustering-ready
2. **OvenMediaEngine** - C++, AGPL-3.0, feature-rich, sub-second latency (live-focused)
3. **NGINX rtmp module** - C, aging, community-maintained, requires custom compilation
4. **Node.js/Express custom server** - JavaScript, full control, more operational overhead

## Decision

**Adopt SRS (Simple RTMP Server) as the primary streaming engine for VOD delivery.**

SRS will handle:
- HLS playlist generation and serving
- DASH manifest serving
- Segment caching and delivery to clients
- Media segment proxy from MinIO

## Rationale

**Why SRS over alternatives:**

| Criterion | SRS | OvenMediaEngine | NGINX rtmp | Node.js Custom |
|-----------|-----|-----------------|-----------|----------------|
| **License** | MIT (✓ good) | AGPL-3.0 (complex) | BSD (✓) | Custom choice |
| **VOD focus** | ✓ First-class | Live-first | General | Depends |
| **Clustering** | ✓ Native | Plugin | Manual | DIY |
| **CPU per Mbps** | ✓ Low | Medium | Medium | High |
| **Docker experience** | ✓ Excellent | Good | OK | Excellent |
| **Configuration** | ✓ Simple YAML/JSON | Complex | C module | Code-based |
| **Community** | ✓ Active Chinese/intl | Medium | Declining | Large |

**Key advantages:**
- **MIT License** - No GPL contamination risk; safe for commercial use
- **VOD-optimized** - SRS 5.0+ explicitly targets HLS/DASH segment serving
- **Lightweight** - Runs on single core with minimal footprint; <100MB memory
- **Battle-tested** - Powers many Asian streaming platforms (Bilibili, Douyin)
- **Docker-native** - Excellent container support, single-binary deployment
- **Clustering ready** - Origin/Edge architecture scales to multi-node later

**Trade-offs:**
- No sub-100ms latency (unlike OvenMediaEngine WebRTC) — acceptable for VOD
- Community smaller than NGINX (but active and responsive)
- Written in C++ (not Node.js) — no code reuse from API service

## Consequences

**Positive:**
- Single lightweight container per SRS instance (can scale with transcoder load)
- Fast segment serving under load (native C++ implementation)
- Simple YAML config file, no compilation required
- Can expand to origin/edge clustering without rearchitecting
- Proven stability in production (Bilibili case studies available)

**Negative:**
- C++ debugging harder than Node.js (smaller team impact)
- OvenMediaEngine might be better if future live-streaming feature added (not current plan)
- Less familiar to Node.js-primary team (training needed)

**Neutral:**
- Requires separate service container (but already designed in C2 model)
- Object storage integration requires MinIO client library (standard practice)

## Alternatives Considered & Rejected

1. **NGINX rtmp module** - Dying community; requires custom compilation; more ops burden
2. **OvenMediaEngine** - Overkill for VOD; AGPL complicates licensing; heavier footprint
3. **Node.js custom** - Reinventing wheel; lower performance; more maintenance burden

## Related Decisions

- **ADR 0001:** Fixed-Bitrate Transcoding – SRS efficiently serves pre-encoded multiple bitrates
- **ADR 0003:** MinIO Object Storage – SRS plugs directly into S3-compatible APIs
- **ADR 0004:** Docker Compose – One SRS container per availability zone

## Future Evolution

If VOD-to-live expansion occurs:
- Consider SRS WebRTC module (added 2024) or pivot to OvenMediaEngine
- Current decision does not preclude this upgrade path

## References

- [SRS Official Documentation](https://ossrs.io/)
- [SRS Clustering Guide](https://ossrs.io/lts/en-us/docs/v5/doc/cluster)
- [SRS vs. OvenMediaEngine Comparison](https://github.com/ossrs/srs/issues/1882)
- [Bilibili Infrastructure Talk](https://www.infoq.com/presentations/bilibili-srs) (Chinese audio, SRS deployment at scale)
