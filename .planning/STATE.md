# Project State: MTGO Replay Recorder

**Core Value:** Capture accurate, shareable replays of MTGO games

---

## Project Reference

This is a Windows desktop application that runs alongside Magic: The Gathering Online to capture, store, and replay games. Uses computer vision to read board state and game logs in real-time, then stores them as compact, shareable replay files with event-based reconstruction.

**Key constraint:** MTGO window must remain on screen during v1 capture (background capture deferred to v2).

---

## Current Position

**Phase:** 0 - Initialization Complete
**Plan:** None - awaiting phase 1 planning
**Status:** Ready to begin Phase 1 implementation

### Progress Bar

```
Phase 1: █████████░░░░░░░░░ 0% (0/3 requirements)
Phase 2: ░░░░░░░░░░░░░░░░░ 0% (0/4 requirements)
Phase 3: ░░░░░░░░░░░░░░░░░ 0% (0/2 requirements)
Phase 4: ░░░░░░░░░░░░░░░░░ 0% (0/3 requirements)

Overall: ░░░░░░░░░░░░░░░░░ 0% (0/10 requirements)
```

---

## Performance Metrics

No metrics yet - implementation has not begun.

**Key metrics to track during development:**
- Capture FPS (target: 10+ FPS minimum)
- Screenshot latency (target: <30ms per frame)
- OCR accuracy (target: 90%+ for card names)
- Replay file size (target: <1MB for 10-minute game)
- Card recognition accuracy (target: 85%+ with hybrid approach)
- Memory usage during capture (target: stable, no leaks over 1+ hour)

---

## Accumulated Context

### Key Decisions Made

| Decision | Date | Rationale | Status |
|----------|------|-----------|--------|
| Computer vision approach | 2025-01-30 | MTGO logs lack board state data | Implementation pending |
| Event-list format | 2025-01-30 | More compact than state snapshots (10-100x smaller than video) | Implementation pending |
| Hybrid card recognition | 2025-01-30 | Image matching alone unreliable with card art variations | Implementation pending |
| Accept v1 limitations | 2025-01-30 | Prioritize working replays over perfect capture | Confirmed constraint |

### Technology Stack

**Confirmed from research:**
- Python 3.12+ (core runtime)
- PyQt6 6.10.1 (Windows desktop GUI)
- MSS 10.1.0 (screen capture - ultra-fast)
- OpenCV 4.13.0.90 (computer vision / image recognition)
- Tesseract OCR (via pytesseract 0.3.13)
- Pillow 12.1.0 (image processing)
- NumPy 2.4.1 (array computing)
- pywin32 311 (Windows API access)
- Python multiprocessing (bypass GIL for parallelism)

### Known Technical Constraints

- **Platform:** Windows only
- **Capture requirement:** MTGO window must remain visible during v1
- **Real-time processing:** Must keep pace with live gameplay
- **Multi-process architecture:** Required to bypass Python GIL for CPU-bound CV work

### Critical Pitfalls to Avoid

1. **Screenshot bottleneck** - Use MSS or BitBlt from day one, benchmark <30ms before any CV code
2. **OCR hallucinations** - Always request confidence scores, set 70-80% threshold, reject low-confidence reads
3. **Card recognition false positives** - Hybrid approach requiring both image matching and OCR agreement
4. **Event timeline fragmentation** - Store both event stream AND state snapshots
5. **Window desynchronization** - Use checksum-based change detection

---

## Session Continuity

### Last Action
Initialized project with requirements, research, and roadmap documents.

### Next Action
Run `/gsd-plan-phase 1` to create implementation plan for Phase 1 (Capture Infrastructure).

### Blockers
None - ready to proceed.

### Outstanding Questions
None at this time.

---

## Notes

### Research Summary
Research completed on 2025-01-30 with HIGH confidence on stack and pitfalls, MEDIUM confidence on features and architecture. Critical gaps remain in MTGO-specific details (log format, card database scope, recognition accuracy with real captures).

### Requirements Status
v1: 12 requirements defined (10 planned for MVP)
- Capture Infrastructure: 3 requirements
- Storage Format: 3 requirements (plus CARD-02)
- Card Recognition: 2 requirements
- Visualization: 3 requirements

v1.1: Deferred after v1 validation (playback controls)
v1.x: Deferred to future release (multi-game, sideboarding, deck extraction)
v2: Deferred until product-market fit established (gap handling, analysis, web viewer, background capture)

### Out of Scope Features
- Real-time opponent hand detection (violates competitive integrity)
- Live overlay during gameplay (may violate TOS)
- Perfect 100% accuracy (computer vision limitation)
- All card sets supported (20+ years of sets = maintenance burden)
- Automatic deck building suggestions (complex, requires ML)
- Video-only replays (huge file sizes, no interactivity)
- Auto-upload to cloud (privacy concerns, costs)
- Background capture in v1 (MTGO doesn't render when minimized)

---

*State initialized: 2025-01-30*
