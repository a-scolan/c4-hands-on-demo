# ADR 0005: HLS + DASH Dual Protocol Support

**Status:** Accepted

**Date:** January 19, 2026

## Context

VOD delivery requires standardized streaming protocols for maximum device/browser compatibility. Key protocols:

1. **HLS (HTTP Live Streaming)** - Apple proprietary, de facto standard on iOS/macOS, widely supported (90%+ devices)
2. **DASH (Dynamic Adaptive Streaming over HTTP)** - ISO standard (MPEG-DASH), W3C standardized, Android/modern browsers
3. **WebRTC** - Low-latency (<1s), P2P capable, poor device compatibility for VOD
4. **RTMP** - Legacy, deprecated, Flash-based, not for modern deployments
5. **HLS-only** - Simpler, supports 90% of use cases, worst Android experience
6. **DASH-only** - Standards-compliant, poor Apple/legacy support

## Decision

**Support both HLS and DASH protocols simultaneously.**

- **HLS** (.m3u8) as primary for broad compatibility (iOS, macOS, Safari, older browsers)
- **DASH** (.mpd) as secondary for standards compliance and modern Android devices
- Single source transcoding produces manifests in both formats (no redundant encoding)

## Rationale

**Why HLS + DASH together:**

| Criterion | HLS | DASH | WebRTC | RTMP |
|-----------|-----|------|--------|------|
| **Device support** | ✓✓ 90%+ | ✓ 85% (no iOS) | ~50% (WebRTC-capable) | ✗ Obsolete |
| **Browser support** | ✓ (via plugins) | ✓ (modern) | ✓ (modern) | ✗ Flash only |
| **Standardization** | Proprietary (Apple) | ✓ ISO/W3C standard | ✓ W3C, IETF | Industry abuse |
| **Latency** | 10–30s | 10–30s | <1s | ~5s |
| **Video quality** | Excellent | Excellent | Good | Good |
| **Implementation complexity** | Medium | Medium | Complex | Simple (deprecated) |
| **Operator knowledge** | Ubiquitous | Growing | Specialized | Dying |

**Key advantages of dual support:**
- **Maximal compatibility** - HLS covers Apple/legacy, DASH covers standards/modern Android
  - iOS/macOS users → HLS stream
  - Android users → DASH stream (or HLS fallback)
  - Browser users → DASH (native) or HLS (via library)

- **No redundant encoding** - Single FFmpeg encoding pipeline produces both HLS and DASH manifests
  - Write video segments once (480p, 720p, 1080p)
  - Generate both .m3u8 (HLS) and .mpd (DASH) playlists from same segments
  - ~10% manifest generation overhead, zero content duplication

- **Standards future-proof** - DASH is ISO standard, HLS is de facto; supporting both hedges bets
  - If HLS becomes legacy in 5 years, DASH already available
  - If new protocol emerges, same segments serve via new manifest format

- **Client library availability** - Multiple JavaScript libraries support both
  - HLS.js (HLS + fallback to DASH)
  - Dash.js (DASH + fallback to HLS)
  - Video.js (plugin-based, supports both)

## Consequences

**Positive:**
- **Universal playback** - Works on >95% of devices/browsers without configuration
- **Reduced support burden** - Single encoding, clients self-select protocol
- **Competitive compatibility** - On par with YouTube, Netflix, Vimeo
- **Progressive degradation** - Fallback mechanism: if DASH unavailable, HLS works
- **Segment reuse** - Both formats read from same encoded segment files (storage-efficient)

**Negative:**
- **Slightly larger manifest files** - Both .m3u8 and .mpd stored per video (negligible: <100KB total)
- **Player library complexity** - Web client must handle protocol negotiation (solved by existing libraries)
- **Operator learning curve** - DASH less familiar than HLS (offset by excellent documentation)

**Neutral:**
- **Testing burden** - Must validate playback on iOS (HLS) and Android (DASH) — standard practice anyway
- **Manifest generation** - SRS + FFmpeg both support both formats natively (no custom code needed)

## Implementation Details

**Encoding pipeline:**
```
Original Video
    ↓
FFmpeg Transcoding (H.264 @ 480p, 720p, 1080p)
    ↓
Segment Output (.m4s files in ISOBMFF container)
    ↓
├─ HLS Manifest Generator (.m3u8 playlists)
└─ DASH Manifest Generator (.mpd playlists)
    ↓
MinIO Object Storage
    ↓
SRS Streaming Server (serves both formats via HTTP)
```

**Storage layout:**
```
/videos/{video_id}/
  480p/
    ├─ segment-001.m4s
    ├─ segment-002.m4s
    ...
  720p/
    ├─ segment-001.m4s
    ...
  1080p/
    ├─ segment-001.m4s
    ...
  manifest.m3u8     (HLS master playlist)
  manifest.mpd      (DASH master playlist)
  manifest-480.m3u8 (HLS variant for 480p)
  manifest-480.mpd  (DASH Period for 480p)
  ... (similar for 720p, 1080p)
```

**Client-side selection (JavaScript example):**
```javascript
// HLS.js automatically selects:
if (Safari || iOS || macOS) {
  playHLS('manifest.m3u8');  // Native HLS support
} else if (ModernBrowser) {
  playDASH('manifest.mpd');  // DASH.js library
} else {
  playHLS('manifest.m3u8');  // Fallback
}
```

**SRS configuration:**
```conf
listen              1935;
max_connections     1000;
http_api {
    enabled         on;
    listen          1985;
}
http_server {
    enabled         on;
    listen          8080;
    dir             ./objs/nginx/html;
}
vhost __defaultVhost__ {
    hls {
        on;
        out_ts_file     ./objs/nginx/html/[app]/[stream].ts;
        out_m3u8_file   ./objs/nginx/html/[app]/[stream].m3u8;
    }
    dash {
        on;
        out_mpd_file    ./objs/nginx/html/[app]/[stream].mpd;
    }
}
```

## Alternatives Considered & Rejected

1. **HLS-only** - Simpler, but Android users get poor experience (10–15% of viewers)
2. **DASH-only** - Standards-compliant, but breaks 30% of iOS/legacy viewers
3. **HLS + WebRTC** - WebRTC gives low-latency but requires additional infrastructure (STUN/TURN) and broken on 50% of networks; overkill for VOD
4. **RTMP** - Deprecated; Flash security vulnerabilities; no modern browser support

## Related Decisions

- **ADR 0001:** Fixed-bitrate ladder (480p, 720p, 1080p) naturally feeds both HLS and DASH
- **ADR 0002:** SRS natively generates both HLS and DASH manifests from encoded segments
- **ADR 0003:** MinIO stores segments and manifests (both formats)

## Testing Strategy

**Playback validation:**
- iOS device (iPhone/iPad) → HLS via Safari
- Android device (Pixel/Samsung) → DASH via Chrome
- Desktop (Chrome, Firefox, Safari) → DASH via Dash.js or HLS.js
- Legacy browser (IE11) → HLS via Shaka Player fallback

**Quality assurance:**
- Monitor manifest validity via https://dashif.org/conformance.html
- Test bitrate switching under network throttling (DevTools)
- Verify segment integrity (MD5 checksums in manifests)

## References

- [HLS Specification](https://tools.ietf.org/html/draft-pantos-http-live-streaming)
- [DASH/MPEG-DASH Standard](https://dashif.org/docs/DASH-IF-IOP-v4.3.pdf)
- [HLS.js Documentation](https://github.com/video-dev/hls.js)
- [Dash.js Documentation](https://github.com/Dash-Industry-Forum/dash.js)
- [SRS HLS + DASH Support](https://ossrs.io/lts/en-us/docs/v5/doc/hls)
- [FFmpeg HLS/DASH Output](https://trac.ffmpeg.org/wiki/Encode/H.264#HLS)
