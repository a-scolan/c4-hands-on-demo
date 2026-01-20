# ADR 0001: Fixed-Bitrate Transcoding Strategy

**Status:** Accepted

**Date:** January 19, 2026

## Context

The video streaming platform must support multiple bitrates and resolutions to accommodate diverse device capabilities (mobile phones, tablets, desktops) and network conditions (3G, LTE, WiFi). We face a choice between:

1. **Adaptive Bitrate (ABR)** - Encode videos at many bitrates (500Kbps–8Mbps with 6+ tiers), consuming more storage and encoding time but providing optimal quality per device
2. **Fixed Bitrate Ladder** - Encode at 3 fixed tiers (480p@2Mbps, 720p@5Mbps, 1080p@10Mbps) with simpler orchestration and faster encoding

## Decision

**Adopt a 3-tier fixed-bitrate ladder strategy:**
- **480p @ 2 Mbps** - Mobile/low-bandwidth devices
- **720p @ 5 Mbps** - Tablets, standard WiFi
- **1080p @ 10 Mbps** - Desktops, high-bandwidth connections

This approach balances quality, storage efficiency, and encoding simplicity for a self-hosted VOD platform.

## Rationale

**Advantages:**
- **Simpler encoding orchestration** - Fixed quality tiers eliminate complex bitrate ladder optimization
- **Reduced storage footprint** - 3 versions per video vs. 6–8 with ABR
- **Predictable transcoding time** - Each video takes ~same wall-clock time regardless of content
- **Lower computational cost** - Fewer parallel FFmpeg jobs per video
- **Clear quality segmentation** - Viewers understand "480p", "720p", "1080p" UX

**Trade-offs:**
- Viewers may experience quality mismatch (HD content at 480p, or overkill 1080p encoding for low-res source)
- No true adaptive streaming (player still selects tier, doesn't adapt mid-playback)

**Suitable for VOD because:**
- VOD content is pre-encoded (unlike live which requires real-time decisions)
- Batch processing on schedule reduces peak load
- Quality variance acceptable for self-hosted platform (not high-SLA commercial CDN)

## Consequences

**Positive:**
- Rapid initial deployment—start transcoding immediately without bitrate analysis
- Easy to communicate to users ("watch in 480p, 720p, or 1080p")
- Storage cost predictable: ~3× original file size across all encodings
- Minimal memory/CPU requirements for transcoder service

**Negative:**
- Source videos with varying resolutions may not encode optimally (e.g., 360p source → 1080p encode wastes space)
- No client-side bitrate adaptation (HLS/DASH still serve all tiers; client chooses, doesn't adapt)
- May over-encode low-res sources or under-serve high-res content

**Neutral:**
- Architecture can evolve to per-video analysis if needed (ADR supersession)
- MinIO object storage scales regardless of file count

## Related Decisions

- **ADR 0002:** SRS Streaming Engine – HLS/DASH support pairs well with fixed bitrate output
- **ADR 0004:** Docker Compose Architecture – Handles 3 parallel FFmpeg jobs per video

## References

- [FFmpeg Encoding Guide](https://trac.ffmpeg.org/wiki/Encode/H.264)
- [HLS Specification](https://tools.ietf.org/html/draft-pantos-http-live-streaming) (fixed playlist structure matches fixed ladder)
- [DASH-IF Guidelines](https://dashif.org/) (Period structure)
