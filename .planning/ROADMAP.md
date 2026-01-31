# Roadmap: MTGO Replay Recorder

**Created:** 2025-01-30
**Depth:** Standard
**Coverage:** 10/10 v1 requirements mapped

## Overview

This roadmap delivers an MVP for capturing, storing, and replaying MTGO games using computer vision and OCR. The phases are ordered by dependency: capture infrastructure must work reliably before card recognition can be tested, storage format must be designed before events can be recorded, and visualization depends on having clean data to display.

The structure prioritizes avoiding critical pitfalls identified in research: screenshot bottleneck in Phase 1, OCR hallucinations in Phase 2, card recognition false positives in Phase 3, and event timeline fragmentation in Phase 4.

---

## Phase 1: Capture Infrastructure

**Goal:** Capture MTGO window content reliably and automatically detect game state changes.

**Dependencies:** None (foundation phase)

**Requirements:**
- CAPT-01: Capture MTGO window at minimum 10 FPS while visible
- CAPT-02: Detect game start and end automatically
- CAPT-03: Monitor specific regions of MTGO window independently

**Success Criteria:**
1. User can start and stop screen capture from MTGO window at minimum 10 FPS
2. System automatically detects when an MTGO game begins and ends
3. System captures specific regions (log panel, battlefield) independently of full window
4. Frame rate remains at or above 10 FPS during active gameplay

---

## Phase 2: Storage Format & Log OCR

**Goal:** Store game events in a compact replay format and extract game log text with confidence validation.

**Dependencies:** Phase 1 (must have working capture pipeline)

**Requirements:**
- STOR-01: Store game events in compact event-list format (JSON with delta compression)
- STOR-02: Include periodic state snapshots in replay file for recovery
- STOR-03: Replay files smaller than 1MB for 10-minute game
- CARD-02: Recognize card names from MTGO game log panel with 90%+ accuracy

**Success Criteria:**
1. System stores game events in compact event-list format
2. Replay files for 10-minute games are smaller than 1MB
3. System includes periodic full-state snapshots for error recovery
4. System extracts card names from game log with 90%+ accuracy and validates confidence scores

---

## Phase 3: Card Recognition & State Tracking

**Goal:** Identify cards on the battlefield and track their positions across zones using hybrid recognition.

**Dependencies:** Phase 1 (capture), Phase 2 (OCR infrastructure)

**Requirements:**
- CARD-01: Identify cards on battlefield using hybrid recognition (image matching + OCR fallback)
- CARD-03: Track card positions and zones (battlefield, hand, graveyard, library)

**Success Criteria:**
1. System identifies cards on battlefield using hybrid recognition (both image matching and OCR must agree)
2. System tracks card positions and associates them with zones (battlefield, hand, graveyard, library)
3. Card recognition uses confidence thresholds and requires cross-validation between image matching and OCR
4. System handles card movement between zones (e.g., from hand to battlefield, battlefield to graveyard)

---

## Phase 4: Replay Visualization & Metadata

**Goal:** Display captured game state visually and render card images with metadata context.

**Dependencies:** Phase 1 (capture), Phase 2 (storage format), Phase 3 (card recognition data)

**Requirements:**
- VIS-01: Display board state visualization showing cards in each zone
- VIS-02: Render card images in visualization using local card art cache
- VIS-03: Display game metadata (date/time, turn count, player names)

**Success Criteria:**
1. System displays board state visualization showing all cards in each zone (hand, battlefield, graveyard, library)
2. System renders card images using local art cache (downloads if needed)
3. System displays game metadata including date/time, turn count, and player names
4. User can navigate between different points in the captured replay

---

## Progress

| Phase | Name | Status | Requirements |
|-------|------|--------|--------------|
| 1 | Capture Infrastructure | Pending | 3/3 |
| 2 | Storage Format & Log OCR | Pending | 4/4 |
| 3 | Card Recognition & State Tracking | Pending | 2/2 |
| 4 | Replay Visualization & Metadata | Pending | 3/3 |

**Overall Progress:** 0/10 requirements complete

---

## Coverage Map

| Requirement | Phase | Status |
|-------------|-------|--------|
| CAPT-01 | 1 | Pending |
| CAPT-02 | 1 | Pending |
| CAPT-03 | 1 | Pending |
| STOR-01 | 2 | Pending |
| STOR-02 | 2 | Pending |
| STOR-03 | 2 | Pending |
| CARD-02 | 2 | Pending |
| CARD-01 | 3 | Pending |
| CARD-03 | 3 | Pending |
| VIS-01 | 4 | Pending |
| VIS-02 | 4 | Pending |
| VIS-03 | 4 | Pending |

**Coverage:** 12/12 v1 requirements mapped ✓

---

## Phase Dependencies

```
Phase 1 (Capture)
    ↓
Phase 2 (Storage & OCR) ← requires Phase 1
    ↓
Phase 3 (Card Recognition) ← requires Phase 1, Phase 2
    ↓
Phase 4 (Visualization) ← requires Phase 1, Phase 2, Phase 3
```

---

## Pitfall Prevention by Phase

| Critical Pitfall | Prevention Phase | Mitigation |
|-----------------|------------------|------------|
| Screenshot bottleneck | Phase 1 | Benchmark MSS capture speed before any CV code; implement parallel processing |
| OCR hallucinations | Phase 2 | Implement confidence thresholds (70-80%); reject low-confidence reads with retry |
| Card recognition false positives | Phase 3 | Hybrid approach requiring image matching AND OCR agreement |
| Event timeline fragmentation | Phase 2 | Store both event stream AND periodic state snapshots |
| Window desynchronization | Phase 1 | Implement checksum-based frame change detection |

---

*Roadmap created: 2025-01-30*
