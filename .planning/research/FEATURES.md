# Feature Research

**Domain:** Game replay systems (MTGO-focused)
**Researched:** 2025-01-30
**Confidence:** MEDIUM

## Feature Landscape

### Table Stakes (Users Expect These)

Features users assume exist. Missing these = product feels incomplete.

| Feature | Why Expected | Complexity | Notes |
|---------|--------------|------------|-------|
| Play/Pause/Rewind controls | Standard in all media playback tools | LOW | Users expect timeline scrubbing, not just start-to-end playback |
| Replay file export/import | Without sharing, replays have limited utility | MEDIUM | Standard file format needed for community adoption |
| Basic board state visualization | Cannot replay what you cannot see | HIGH | Must show cards, hands, battlefield at minimum |
| Game timeline display | Users need context of what happened when | LOW | Turn markers, phases, key events on timeline |
| Card tooltips/hover info | MTGO has thousands of cards, users need reference | MEDIUM | Links to card database or cached card data |
| Search/filter within replay | Long games require finding specific moments | LOW | Search by card name, turn number, player action |
| Game metadata display | Match context is critical for study | LOW | Deck lists, players, format, date/time |
| Basic statistics | Users expect insights from replay data | MEDIUM | Turn count, cards drawn, mana spent |

### Differentiators (Competitive Advantage)

Features that set the product apart. Not required, but valuable.

| Feature | Value Proposition | Complexity | Notes |
|---------|-------------------|------------|-------|
| Computer vision card recognition | MTGO doesn't expose board state via API | HIGH | Hybrid approach: image matching + OCR for reliability |
| Compact event-list format | Video files are too large for sharing | MEDIUM | 10-100x smaller than video, shareable via Discord/Reddit |
| Sideboarding detection | No MTGO tool currently detects board changes | HIGH | Analyzes deck list changes between games in match |
| Multi-game match support | MTGO matches are best-of-3 typically | MEDIUM | Links games together, shows sideboarding flow |
| Real-time capture while visible | No need for API access or mods | MEDIUM | Works with vanilla MTGO, stays on-screen requirement |
| Turn-by-turn analysis | Deep study requires granular data | HIGH | Probability calculations, optimal play suggestions |
| Deck list extraction | Users want to know what opponents played | MEDIUM | Reconstruct deck lists from played cards |
| Replay sharing platform | Community growth requires ecosystem | HIGH | Web viewer, embeddable replays, social features |
| Interactive replayer | Not just playback, but exploration | MEDIUM | Click cards to see history, alternate timeline views |
| Gap handling for minimized windows | MTGO must stay visible in v1, but gaps occur | HIGH | Reconstruct from scrollable logs when window restored |
| Export to other formats | Users want flexibility in data usage | LOW | JSON, CSV, video export, deck list export |

### Anti-Features (Commonly Requested, Often Problematic)

Features that seem good but create problems.

| Feature | Why Requested | Why Problematic | Alternative |
|---------|---------------|-----------------|-------------|
| Background/minimized capture | Users want to use PC while recording | MTGO doesn't render when minimized, requires API/mod | Require window visible, add notification warnings |
| Real-time streaming | Twitch/YouTube integration | Adds complexity, conflicts with OBS, video recording already solves this | Export video for streaming, use dedicated streaming tools |
| Auto-upload to cloud | Easy backup and sharing | Privacy concerns, costs, depends on third-party | Local file export, optional cloud sync via user's choice |
| Live overlay during gameplay | HUD-style information while playing | May violate TOS, distracting, conflicts with MTGO UI | Post-game analysis only, separate viewer application |
| Perfect 100% accuracy | Users want reliable replays | Computer vision cannot guarantee perfect recognition | Acknowledge accuracy limits, provide confidence indicators |
| All card sets supported | Comprehensive card database | MTGO has 20+ years of sets, maintenance burden | Start with recent/modern formats, expand based on demand |
| Real-time opponent hand detection | Strategic advantage | Violates competitive integrity, potentially illegal | Never reveal hidden information (opponent's hand, library) |
| Automatic deck building suggestions | Help users improve | Complex, requires ML, prone to errors | Provide deck list extraction only, let users analyze |
| Video-only replays | Simple solution to capture | Huge file sizes, cannot extract data, no interactivity | Event-list format with optional video export |

## Feature Dependencies

```
[Computer Vision Card Recognition]
    └──requires──> [Board State Visualization]
                   └──requires──> [Event-list Storage Format]

[Sideboarding Detection]
    └──requires──> [Deck List Extraction]
                   └──requires──> [Multi-game Match Support]

[Interactive Replayer]
    └──requires──> [Event-list Storage Format]
                   └──enhances──> [Turn-by-turn Analysis]
    └──requires──> [Board State Visualization]

[Replay Sharing Platform]
    └──requires──> [Compact Event-list Format]
    └──requires──> [Basic Board State Visualization]

[Gap Handling]
    └──enhances──> [Real-time Capture]
```

### Dependency Notes

- **[Computer Vision Card Recognition] requires [Board State Visualization]:** Cannot capture cards without knowing what to display and where
- **[Sideboarding Detection] requires [Deck List Extraction]:** Must know what cards were in/out to detect changes
- **[Interactive Replayer] enhances [Turn-by-Turn Analysis]:** Ability to scrub and explore enables deeper analysis
- **[Replay Sharing Platform] requires [Compact Event-list Format]:** Cannot efficiently share large video files

## MVP Definition

### Launch With (v1)

Minimum viable product — what's needed to validate the concept.

- [ ] Real-time capture of visible MTGO window — Core value delivery
- [ ] Computer vision card recognition (hybrid approach) — Technical validation
- [ ] Event-list storage format for replays — Enables compact sharing
- [ ] Basic interactive replayer (play/pause, timeline scrub) — Usable playback
- [ ] Single game capture — Validate concept before multi-game complexity

### Add After Validation (v1.x)

Features to add once core is working.

- [ ] Multi-game match support — Most matches are best-of-3
- [ ] Sideboarding detection — Differentiator not yet validated
- [ ] Deck list extraction — Secondary use case
- [ ] Basic statistics — Enhanced utility without major complexity
- [ ] Replay file export/import — Community sharing validation

### Future Consideration (v2+)

Features to defer until product-market fit is established.

- [ ] Gap handling for minimized windows — Engineering complexity, solve later
- [ ] Turn-by-turn analysis — Nice to have, not core value
- [ ] Replay sharing platform — Requires user base first
- [ ] Background capture via API or mods — High complexity, explore after validation
- [ ] Advanced AI analysis — Research phase, requires data from v1

## Feature Prioritization Matrix

| Feature | User Value | Implementation Cost | Priority |
|---------|------------|---------------------|----------|
| Real-time capture | HIGH | HIGH | P1 |
| Card recognition (hybrid) | HIGH | HIGH | P1 |
| Event-list format | HIGH | MEDIUM | P1 |
| Basic replayer controls | HIGH | LOW | P1 |
| Single game capture | HIGH | MEDIUM | P1 |
| Multi-game match support | MEDIUM | MEDIUM | P2 |
| Sideboarding detection | HIGH | HIGH | P2 |
| Deck list extraction | MEDIUM | MEDIUM | P2 |
| Basic statistics | MEDIUM | MEDIUM | P2 |
| Replay file export | HIGH | MEDIUM | P2 |
| Gap handling | MEDIUM | HIGH | P3 |
| Turn-by-turn analysis | MEDIUM | HIGH | P3 |
| Sharing platform | HIGH | HIGH | P3 |
| Video export | LOW | LOW | P3 |
| Background capture | HIGH | HIGH | P3 |

**Priority key:**
- P1: Must have for launch
- P2: Should have, add when possible
- P3: Nice to have, future consideration

## Competitor Feature Analysis

| Feature | MORT (Existing MTGO Tool) | Video Recorders (OBS/RePlays) | Our Approach |
|---------|--------------------------|-------------------------------|--------------|
| Board state capture | Yes (from MTGO logs) | No (video only) | Computer vision on visible window |
| Card recognition | Yes (from logs) | No | Hybrid image matching + OCR |
| Interactivity | Yes (statistical analysis) | Limited (video playback) | Full interactive replayer |
| File size | Compact | Large (GBs) | Compact event-list format |
| Real-time capture | No (post-processing) | Yes | Yes (requires visible window) |
| Sideboarding detection | Partial (manual) | No | Automatic detection |
| Multi-game support | Yes | No | Yes (planned) |
| Sharing | No (local analysis) | Via video platforms | Shareable replay files |
| Deck list extraction | Yes | No | Yes (planned) |
| Gap handling | N/A (no real-time) | No | Partial (from scrollable logs) |

**Note:** MORT extracts data from MTGO log files after games, not real-time capture. Our approach is real-time computer vision capture with interactive replay.

## Sources

- MORT (Magic Online Replay Tool): https://github.com/PennyDreadfulMTG/MagicOnlineReplayTool — LOW confidence, only README available
- RePlays (game recording software): https://github.com/lulzsun/RePlays — LOW confidence, only README available
- General game replay patterns (training data): Unreal/Unity replay systems, esports replay features — LOW confidence, requires verification
- MTGO constraints (PROJECT.md): Logs don't capture full board state, computer vision required — HIGH confidence, stated in project documentation

---
*Feature research for: Game replay systems (MTGO)*
*Researched: 2025-01-30*
