# Project Research Summary

**Project:** MTGO Replay Recorder
**Domain:** Game replay capture system with computer vision and OCR
**Researched:** 2026-01-30
**Confidence:** MEDIUM

## Executive Summary

This project is a Windows desktop application that captures Magic: The Gathering Online gameplay using computer vision and OCR, then stores replays in a compact event-list format for interactive playback. Experts build this as a multi-process pipeline: capture runs in one process (MSS for screenshots), OCR and CV in parallel worker processes (bypassing Python's GIL), with a PyQt6 GUI for control. Event sourcing enables dramatically smaller replay files (10-100x vs video) while supporting scrubbing and timeline navigation.

The recommended approach prioritizes performance from day one: use MSS for ultra-fast capture (<30ms per frame), implement producer-consumer queues to decouple capture from processing, and design a hybrid card recognition system (image matching + OCR with cross-validation). Multi-process architecture is essential - Python's GIL would cripple performance if everything runs in one process. The stack (Python 3.12+, PyQt6, MSS, OpenCV, Tesseract) is mature and well-documented, with HIGH confidence on all major components.

Critical risks center on data integrity and performance: slow screenshots cause missed events, OCR hallucinations corrupt replay files silently, and window desynchronization leads to fragmented timelines. Mitigation requires benchmarking capture speed before any CV code, implementing OCR confidence thresholds with rejection logging, and using checksum-based frame change detection. The architecture must balance real-time performance with accuracy - frame skipping is acceptable if designed explicitly, but skipping OCR confidence checking is never acceptable.

## Key Findings

### Recommended Stack

Python 3.12+ with PyQt6 for desktop GUI, MSS for ultra-fast screen capture, OpenCV for computer vision, and Tesseract OCR form the core. This combination was chosen because Python's ecosystem excels at CV/OCR work, PyQt6 provides native-feeling Windows UI with lower resource overhead than Electron, MSS delivers sub-30ms capture (vs 100ms+ for alternatives), and OpenCV/Tesseract are industry standards with Python bindings. Multi-process architecture (capture + worker pool + storage) bypasses the GIL for true parallelism.

**Core technologies:**
- **Python 3.12+**: Core runtime — Mature ecosystem for computer vision, excellent library availability for screen capture, OCR, and image processing
- **PyQt6 6.10.1**: Windows desktop GUI — Native-feeling application with proper window management and lower resource overhead than Electron
- **MSS 10.1.0**: Screen capture — Ultra-fast cross-platform screenshots using ctypes, actively maintained, thread-safe
- **OpenCV 4.13.0.90**: Computer vision — Industry-standard library for template matching, feature detection, and deep learning
- **Tesseract OCR (pytesseract 0.3.13)**: OCR engine — Open-source text recognition with Python wrapper, offline capable
- **Python multiprocessing**: Parallel processing — Bypasses GIL for true parallelism, essential for CPU-bound CV work

### Expected Features

**Must have (table stakes):**
- Play/Pause/Rewind controls — Users expect timeline scrubbing in all media tools
- Basic board state visualization — Cannot replay what you cannot see (cards, hands, battlefield)
- Game timeline display — Users need context of what happened when (turns, phases, events)
- Replay file export/import — Without sharing, replays have limited utility
- Card tooltips/hover info — MTGO has thousands of cards, users need reference
- Search/filter within replay — Long games require finding specific moments
- Game metadata display — Match context is critical for study (decks, players, format)

**Should have (competitive):**
- Computer vision card recognition — MTGO doesn't expose board state via API, hybrid approach (image + OCR) for reliability
- Compact event-list format — 10-100x smaller than video, shareable via Discord/Reddit
- Sideboarding detection — No MTGO tool currently detects deck changes between games
- Multi-game match support — MTGO matches are typically best-of-3
- Interactive replayer — Not just playback, but exploration (click cards to see history)
- Deck list extraction — Users want to know what opponents played

**Defer (v2+):**
- Gap handling for minimized windows — Engineering complexity, solve after core validation
- Turn-by-turn analysis — Nice to have, not core value
- Replay sharing platform — Requires user base first
- Background capture via API or mods — High complexity, explore after validation

### Architecture Approach

Producer-consumer pipeline with event sourcing separates capture from processing, enabling backpressure handling and parallel execution. Capture runs at its own pace filling a bounded queue; worker processes consume frames for OCR/CV independently. Event sourcing stores only what changed (deltas), not full state snapshots, achieving dramatic file size reduction. Hybrid card recognition chains image matching (fast but brittle) with OCR text extraction (slower but reliable), requiring both methods to agree for high confidence.

**Major components:**
1. **Capture Layer** (Window Monitor, Screen Capture, Frame Buffer) — Detect MTGO window, grab frames via MSS, queue for processing
2. **Processing Pipeline** (Log OCR, Card Match, State Update) — Extract text from logs, identify cards via CV, merge into game state
3. **Event Stream** — Record game events in compact timestamped format
4. **Storage Layer** (Replay File, Card DB, State Cache) — Persistent event storage, Scryfall card reference, in-memory state
5. **Replay Viewer** — Reconstruct and display games from event stream

### Critical Pitfalls

1. **Screenshot bottleneck** — Use MSS or BitBlt from day one, benchmark capture speed (<30ms) before any CV code, implement parallel processing, avoid PyScreeze/PIL.ImageGrab (100ms+ per capture)
2. **OCR hallucinations** — Always request confidence scores from Tesseract, set minimum threshold (70-80%), reject low-confidence reads as "[OCR FAILED]" with retry count, never trust OCR without validation
3. **Window/frame desynchronization** — Use checksum-based change detection to capture only when frames change, implement frame buffering for events spanning multiple frames, add frame continuity checks to detect missed frames
4. **Card recognition false positives** — Use hybrid approach (image matching + OCR) requiring both methods to agree, implement confidence thresholds for both, never rely solely on grayscale matching (color is key for MTG)
5. **Event timeline fragmentation** — Capture both event stream AND state snapshots (not just events), include full board state at game start and after each phase change, store card references (ID, name, type) not just names

## Implications for Roadmap

Based on research, suggested phase structure:

### Phase 1: Foundation & Capture Pipeline

**Rationale:** Screenshot performance is the foundation everything depends on. The research identifies "screenshot bottleneck" as a critical pitfall that causes performance death spirals. Capture pipeline must be verified before adding any processing logic.

**Delivers:** Working MTGO window detection, high-speed frame capture via MSS, bounded frame buffer queue, frame change detection via checksums, basic frame rate benchmarking

**Addresses:** Real-time capture (MVP feature), single game capture

**Avoids:** Screenshot bottleneck (Critical Pitfall #1), Window desynchronization (Critical Pitfall #5)

**Features from FEATURES.md:**
- Real-time capture of visible MTGO window
- Single game capture

**Architecture components:**
- Capture Layer (Window Monitor, Screen Capture, Frame Buffer)

### Phase 2: Replay Format & Log OCR

**Rationale:** Replay format design must come before event capture. The research warns that "event timeline fragmentation" creates unreplayable state. With a solid format, log OCR can be implemented with confidence thresholds to prevent silent data corruption.

**Delivers:** Compact event-list storage format with state snapshots, Tesseract OCR integration with confidence checking, log text extraction and parsing, replay file export/import, basic metadata display

**Uses:** Tesseract OCR via pytesseract

**Implements:** Storage Layer (Replay File), Processing Pipeline (Log OCR)

**Features from FEATURES.md:**
- Event-list storage format for replays
- Replay file export/import
- Game metadata display
- Basic statistics

**Avoids:** OCR hallucinations (Critical Pitfall #2), Event timeline fragmentation (Critical Pitfall #4)

### Phase 3: Card Recognition & State Machine

**Rationale:** Hybrid card recognition is the most complex technical challenge and requires both image matching and OCR working together. This phase depends on solid capture (Phase 1) and reliable OCR (Phase 2).

**Delivers:** Hybrid card matching system (image features + OCR text), card database integration (Scryfall), state machine for tracking game state, card detection on battlefield/graveyard/hand, basic board state visualization

**Uses:** OpenCV for feature matching, Tesseract for card name OCR, Scryfall API for card database

**Implements:** Processing Pipeline (Card Match, State Update), Storage Layer (Card DB, State Cache)

**Features from FEATURES.md:**
- Computer vision card recognition (hybrid approach)
- Basic board state visualization
- Card tooltips/hover info
- Deck list extraction

**Avoids:** Card recognition false positives (Critical Pitfall #3)

### Phase 4: Replay Viewer & Interactive Playback

**Rationale:** With capture and recognition working, the viewer brings everything together. This phase depends on all previous phases delivering reliable data.

**Delivers:** Event playback engine, visual game reconstruction, play/pause/rewind controls, timeline scrubbing, turn/phase markers, search/filter within replay

**Implements:** Replay Viewer (Player, Renderer)

**Features from FEATURES.md:**
- Basic interactive replayer (play/pause, timeline scrub)
- Game timeline display
- Search/filter within replay

### Phase 5: Multi-game & Differentiators

**Rationale:** After core v1 is validated, add competitive differentiators. These are "nice to have" but not required for launch.

**Delivers:** Multi-game match support, sideboarding detection, gap handling for minimized windows (partial), export to other formats (JSON, CSV)

**Features from FEATURES.md:**
- Multi-game match support
- Sideboarding detection
- Gap handling for minimized windows
- Export to other formats

### Phase 6: Polish & Performance

**Rationale:** Final phase focuses on performance optimization, error handling, and UX polish. The research identifies several performance traps and UX pitfalls.

**Delivers:** Long-duration stability (1+ hour capture), error recovery in replay playback, user training and documentation, performance profiling and optimization

**Avoids:** Performance traps, UX pitfalls, technical debt

### Phase Ordering Rationale

- **Foundation first:** Capture pipeline (Phase 1) must be verified before any processing is added. Research identifies screenshot performance as the critical bottleneck that can't be fixed later without major redesign.
- **Format before content:** Replay format design (Phase 2) before event capture because the format determines what data can be stored. Event sourcing vs snapshots decision affects everything downstream.
- **Dependencies chain:** Card recognition (Phase 3) depends on capture working reliably and OCR providing text for cross-validation. Viewer (Phase 4) depends on all previous phases producing clean data.
- **Core before polish:** MVP features (Phases 1-4) are prioritized before differentiators (Phase 5-6). Research emphasizes that v1 should validate the core concept before adding complexity.
- **Avoid pitfalls proactively:** Each phase is designed to prevent specific critical pitfalls identified in research (screenshot bottleneck, OCR hallucinations, false positives, fragmentation).

### Research Flags

Phases likely needing deeper research during planning:
- **Phase 2 (Replay Format & Log OCR):** MTGO log format specifics are not well-documented publicly. Need to analyze actual MTGO log files to understand structure, character encoding, and event types. OCR confidence thresholds need empirical testing with MTGO-specific fonts.
- **Phase 3 (Card Recognition & State Machine):** Card art variations across sets/foil effects create massive search space. Need to determine which card database subsets (modern formats only? all sets?) are practical for v1. OpenCV feature matching parameters (ORB vs SIFT, FLANN vs BF matcher) require testing with real MTGO captures.
- **Phase 5 (Multi-game & Differentiators):** Sideboarding detection algorithm is novel (no existing MTGO tool does this). Need to prototype detection logic using deck list comparison between games.

Phases with standard patterns (skip research-phase):
- **Phase 1 (Foundation & Capture):** Screen capture patterns are well-documented. MSS, PyQt6 window management, and producer-consumer queues have extensive examples.
- **Phase 4 (Replay Viewer):** Event-driven playback engines are a standard pattern in game development. UI controls for scrubbing, pausing, seeking are straightforward with PyQt6.

## Confidence Assessment

| Area | Confidence | Notes |
|------|------------|-------|
| Stack | HIGH | All major technologies verified with official docs/PyPI pages (MSS 10.1.0, OpenCV 4.13.0.90, Tesseract, PyQt6) |
| Features | MEDIUM | Based on competitive analysis (MORT, video recorders) and MTGO constraints, but some differentiators (sideboarding detection) are untested |
| Architecture | MEDIUM | Producer-consumer pipeline and event sourcing are well-established patterns, but multi-process coordination needs prototype validation |
| Pitfalls | HIGH | Five critical pitfalls identified with specific prevention strategies, backed by documentation and performance measurements |

**Overall confidence:** MEDIUM

Stack and pitfalls are HIGH confidence (verified sources). Features and architecture are MEDIUM (established patterns applied to novel domain). Critical gaps remain in MTGO-specific details (log format, card database scope, recognition accuracy with real captures).

### Gaps to Address

- **MTGO log file structure:** Not publicly documented. Need to analyze actual MTGO logs during Phase 2 to understand encoding, event format, and what information is available.
- **Card database scope:** MTGO has 20+ years of card sets. Unclear if v1 should support all sets or start with recent formats. Scryfall API provides bulk data, but performance depends on subset size.
- **Recognition accuracy in practice:** OpenCV template matching and OCR confidence thresholds need empirical testing with real MTGO captures. Lab benchmarks may not reflect real-world conditions (animations, overlays, varying lighting).
- **Window capture edge cases:** Multi-monitor setups, DPI scaling, window overlapping, MTGO window resizing/moving need testing. Research indicates these integration points are prone to errors.
- **State machine complexity:** MTGO rules engine is complex (zones, phases, stack, priority). Need to determine what game state is essential to capture vs what can be inferred.

## Sources

### Primary (HIGH confidence)
- Python MSS documentation — PyPI package page (version 10.1.0, released Aug 16, 2025)
- pytesseract documentation — PyPI package page (version 0.3.13, released Aug 16, 2024)
- opencv-python documentation — PyPI package page (version 4.13.0.90, released Jan 18, 2026)
- Pillow documentation — PyPI package page (version 12.1.0, released Jan 2, 2026)
- NumPy documentation — PyPI package page (version 2.4.1, released Jan 10, 2026)
- PyQt6 documentation — Official docs (version 6.10.1)
- pywin32 documentation — PyPI package page (version 311, released Jul 14, 2025)
- PyScreeze documentation — Performance measurement: "On a 1920 x 1080 screen, the screenshot() function takes roughly 100 milliseconds"
- Windows BitBlt API — Microsoft Learn documentation (CAPTUREBLT flag, layered windows)
- OpenCV documentation — Template matching, ORB features, FLANN matching
- Tesseract OCR documentation — GitHub repo, API for confidence scores
- Scryfall API documentation — Official docs for card database bulk data

### Secondary (MEDIUM confidence)
- MTGO constraints (PROJECT.md) — Logs don't capture full board state, computer vision required
- MORT (Magic Online Replay Tool) — Existing MTGO replay tool that processes log files post-game
- RePlays (game recording software) — Video-based game recorder for comparison
- PyAutoGUI documentation — PAUSE setting, failsafe corner behavior
- Windows Desktop Duplication API — Microsoft Learn (last updated Jan 2021, older docs but stable API)

### Tertiary (LOW confidence)
- General game replay patterns (Unreal/Unity, esports) — Training data, requires validation for MTGO domain
- Card art variations across sets — MTGO community knowledge, not systematically documented
- CNN captcha recognition (nickliqian/cnn_captcha) — OCR preprocessing reference, not MTGO-specific

---
*Research completed: 2026-01-30*
*Ready for roadmap: yes*
