# MTGO Replay Recorder

## What This Is

A Windows desktop application that runs alongside Magic: The Gathering Online to capture, store, and replay games. Uses computer vision to read board state and game logs in real-time, then stores them as compact, shareable replay files with event-based reconstruction.

## Core Value

Capture accurate, shareable replays of MTGO games.

## Requirements

### Validated

(None yet — ship to validate)

### Active

- [ ] Real-time capture of MTGO game state (board + logs) while window is visible
- [ ] Auto-detection of game start and end
- [ ] Hybrid card recognition (image matching + text OCR)
- [ ] Event-list storage format for compact replay files
- [ ] Interactive replayer for viewing captured games
- [ ] Sideboarding detection between games
- [ ] Shareable replay format for community use

### Out of Scope

- Multi-game sessions in v1 — focus on single game capture
- Background capture in v1 — MTGO window must stay visible
- Perfect gap handling in v1 — accepts data drops if MTGO is minimized
- Deck editor screen capture — game-focused only

## Context

MTGO's log files don't capture full board state, making computer vision necessary. The tool monitors the MTGO window in real-time, using OCR to read the log panel and image recognition to identify cards on the board. Logs are scrollable, allowing full history retrieval when window is re-opened, but v1 prioritizes continuous capture over gap reconstruction.

Targeted at the MTGO community for sharing games, analyzing matches, and preserving gameplay for study.

## Constraints

- **Platform**: Windows only — MTGO runs on Windows
- **Capture**: MTGO must remain on screen during v1 — background capture deferred
- **Real-time**: Processing must keep pace with live gameplay
- **Tech stack**: No predetermined preference — choose best tool for the job

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Computer vision approach | MTGO logs lack board state data | — Pending |
| Event-list format | More compact than state snapshots | — Pending |
| Hybrid card recognition | Image matching alone unreliable with card art variations | — Pending |
| Accept v1 limitations | Prioritize working replays over perfect capture | — Pending |

---
*Last updated: 2025-01-30 after initialization*
