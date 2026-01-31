# Requirements: MTGO Replay Recorder

**Defined:** 2025-01-30
**Core Value:** Capture accurate, shareable replays of MTGO games

## v1 Requirements

Requirements for initial release. Each maps to roadmap phases.

**Total v1 requirements: 12**

### Capture Infrastructure

- [ ] **CAPT-01**: System can capture MTGO window screen content at minimum 10 FPS while window is visible
- [ ] **CAPT-02**: System can detect when MTGO game starts and ends automatically
- [ ] **CAPT-03**: System can monitor specific regions of MTGO window (log panel, board zones) independently

### Card Recognition

- [ ] **CARD-01**: System can identify cards on the battlefield using hybrid recognition (image matching + OCR fallback)
- [ ] **CARD-02**: System can recognize card names from MTGO game log panel with 90%+ accuracy
- [ ] **CARD-03**: System can track card positions and zones (battlefield, hand, graveyard, library)

### Storage Format

- [ ] **STOR-01**: System stores game events in compact event-list format (JSON with delta compression)
- [ ] **STOR-02**: System includes periodic state snapshots in replay file for recovery and gap reconstruction
- [ ] **STOR-03**: System stores replay files smaller than 1MB for 10-minute game (target: 10-100x smaller than video)

### Visualization

- [ ] **VIS-01**: System can display board state visualization showing cards in each zone (hand, battlefield, graveyard, library)
- [ ] **VIS-02**: System can render card images in visualization using local card art cache
- [ ] **VIS-03**: System can display game metadata (date/time, turn count, player names)

**v1 requirements total: 12**

## v1.1 Requirements

Deferred after v1 validation.

### Playback Controls

- [ ] **PLAY-01**: User can play captured replay from start to end
- [ ] **PLAY-02**: User can pause and resume replay playback
- [ ] **PLAY-03**: User can seek to any point in replay timeline (scrubbing)

## v1.x Requirements

Deferred to future release after v1 validation.

- [ ] **MULT-01**: System can capture and link multiple games in a single match (best-of-3)
- [ ] **SIDE-01**: System can detect sideboarding changes between games in a match
- [ ] **DECK-01**: System can extract and display deck lists from captured gameplay
- [ ] **STAT-01**: System can calculate and display basic statistics (turn count, cards drawn, mana spent)
- [ ] **EXP-01**: User can export replay files and share them with other users
- [ ] **IMP-02**: User can import and view replay files shared by others

## v2 Requirements

Deferred until product-market fit is established.

- [ ] **GAP-01**: System can reconstruct game state from scrollable MTGO logs when window was minimized
- [ ] **ANAL-01**: System provides turn-by-turn analysis with probability calculations
- [ ] **WEB-01**: System includes web-based viewer for replay sharing platform
- [ ] **BACK-01**: System can capture MTGO content when window is minimized (background capture)

## Out of Scope

Explicitly excluded. Documented to prevent scope creep.

| Feature | Reason |
|---------|--------|
| Real-time opponent hand detection | Violates competitive integrity, potentially illegal |
| Live overlay during gameplay | May violate TOS, distracting, conflicts with MTGO UI |
| Perfect 100% accuracy | Computer vision cannot guarantee perfect recognition |
| All card sets supported (20+ years) | MTGO has 20+ years of sets, maintenance burden |
| Automatic deck building suggestions | Complex, requires ML, prone to errors |
| Video-only replays | Huge file sizes, cannot extract data, no interactivity |
| Auto-upload to cloud | Privacy concerns, costs, depends on third-party |
| Background capture in v1 | MTGO doesn't render when minimized, requires API/mod |

## Traceability

Which phases cover which requirements. Updated during roadmap creation.

| Requirement | Phase | Status |
|-------------|-------|--------|
| CAPT-01 | Phase 1 | Pending |
| CAPT-02 | Phase 1 | Pending |
| CAPT-03 | Phase 1 | Pending |
| STOR-01 | Phase 2 | Pending |
| STOR-02 | Phase 2 | Pending |
| STOR-03 | Phase 2 | Pending |
| CARD-02 | Phase 2 | Pending |
| CARD-01 | Phase 3 | Pending |
| CARD-03 | Phase 3 | Pending |
| VIS-01 | Phase 4 | Pending |
| VIS-02 | Phase 4 | Pending |
| VIS-03 | Phase 4 | Pending |

**Coverage:**
- v1 requirements: 12 total
- Mapped to phases: 12
- Unmapped: 0 ✓

---
*Requirements defined: 2025-01-30*
*Last updated: 2025-01-30 after roadmap creation*
