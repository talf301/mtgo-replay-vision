# Phase 1: Capture Infrastructure - Context

**Gathered:** 2025-01-31
**Status:** Ready for planning

<domain>
## Phase Boundary

Reliably capture MTGO window content at minimum 10 FPS while visible, automatically detect match/game state changes, and support monitoring specific window regions (log panel, battlefield, hand zone) independently. This is a foundation phase delivering core capture infrastructure with user control and automatic state detection capabilities.

</domain>

<decisions>
## Implementation Decisions

### Capture control
- Automatic capture when MTGO window is open - no manual start required
- Detailed visual feedback during capture: recording status indicator, current FPS, total frames captured
- Full manual override available: pause/resume buttons, disable auto-detection option, force stop capability
- When capture stops (MTGO closes or user stops): show summary popup with capture duration, total frames, and replay file location
- All capture controls reside in main application window (not tray/minimal UI)
- One replay file per MATCH (all games in match, not per individual game)
- Replay file naming: include player names if detected in log panel, otherwise fall back to date/time format

### Game detection behavior
- Monitor game log panel content to detect match/game state changes
- Detection frequency: check every N frames (not every frame, not time-based)
- State change confirmation: require consecutive frames showing same state before confirming
- Failure handling: pause capture and prompt user when detection is ambiguous or log panel is corrupted/missing

### Region monitoring setup
- Auto-detect MTGO UI elements (log panel, battlefield, hand zone) by image matching
- Handle MTGO UI variations: try auto-detection first, fall back to known preset coordinates if detection fails
- Window resize handling: automatically re-detect UI elements when MTGO window is resized or moved
- Phase 1 scope: monitor log panel, battlefield, and hand zone (other zones deferred to later phases)

### Claude's Discretion
- Specific value of N for detection frequency (balance CPU cost and detection latency - researcher should benchmark)
- Exact number of consecutive frames required for state confirmation (test and tune based on real MTGO footage)
- Preset coordinate sets for different MTGO window sizes/resolutions/themes (research MTGO UI variations)
- Image matching approach and template library design (research OpenCV template matching best practices)
- How to handle edge cases where image matching partially succeeds (confidence thresholds for region detection)

</decisions>

<specifics>
## Specific Ideas

- "One file per match, not per game" - match contains multiple games (typical MTGO: best of 3)
- Auto-detect by image matching makes the system robust to MTGO updates and UI changes
- Consecutive frame confirmation prevents false triggers from transient UI states or screen artifacts
- Fall back to presets ensures system works even if image matching fails (safety net)

</specifics>

<deferred>
## Deferred Ideas

None - discussion stayed within Phase 1 scope (capture infrastructure)

</deferred>

---

*Phase: 01-capture-infrastructure*
*Context gathered: 2025-01-31*
