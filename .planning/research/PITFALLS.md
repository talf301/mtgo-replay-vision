# Pitfalls Research

**Domain:** Computer vision and replay recording systems
**Researched:** 2026-01-30
**Confidence:** HIGH

## Critical Pitfalls

### Pitfall 1: Screenshot Bottleneck (Performance Death Spiral)

**What goes wrong:**
Real-time capture fails because screenshot operations are too slow. At 1920x1080 resolution, basic screenshot functions can take 100-200ms per capture. At 30 FPS requirement, this allows only 33ms per frame. The capture pipeline accumulates latency, leading to frame drops, missed events, and corrupted replay data.

**Why it happens:**
Developers start with simple libraries like PyScreeze or PIL.ImageGrab, which trade performance for simplicity. PyScreeze explicitly notes: "On a 1920 x 1080 screen, the screenshot() function takes roughly 100 milliseconds." Developers add OCR processing, image matching, and state serialization on top, creating a cascade where each operation takes 50-200ms. By the time the full pipeline runs, 3-5 seconds have passed per frame.

**How to avoid:**
- Use ultra-fast capture methods: MSS (Python) or direct Windows API (BitBlt/PrintWindow) from day one
- Implement region-based capture (capture only MTGO game window, not full screen)
- Process in parallel: capture thread, OCR thread, state thread running simultaneously
- Use frame skipping: capture at 15-20 FPS instead of 30 if state changes infrequently
- Benchmark early: write a simple "capture → save → repeat" loop before any other code

**Warning signs:**
- Screenshot takes >50ms (measured with `time.perf_counter()`)
- Frame rate drops below 10 FPS during gameplay
- Replay files show gaps in event timestamps >500ms apart
- CPU usage at 80-100% during capture

**Phase to address:**
Phase 1 - Foundation setup. Choose and benchmark capture method before any other development. This is the foundation everything else depends on.

---

### Pitfall 2: OCR Hallucinations (Silent Data Corruption)

**What goes wrong:**
OCR confidently returns incorrect text without any confidence score or error indication. Game log shows "Lightning Bolt deals 3 damage" but OCR reads "Lightning Bolt deals 8 damage" or "Lightning Bolt deals ? damage". The replay system records these wrong values faithfully, creating an irrecoverably corrupted replay.

**Why it happens:**
Tesseract and other OCR engines don't fail when they can't read text—they return their best guess. Without confidence thresholds, the system treats "95% sure" and "5% sure" identically. MTGO game logs use specific fonts, sizes, and colors that may not match Tesseract's training data. Character overlap, semi-transparent backgrounds, and anti-aliasing patterns in game UI create noise that OCR interprets as valid text.

**How to avoid:**
- Always request confidence scores from OCR (Tesseract: `GetIterator().GetConfidence()`)
- Set minimum confidence threshold (typically 70-80% for game logs)
- Reject low-confidence reads and log them as "[OCR FAILED]" with retry count
- Implement cross-validation: compare OCR with previous frame's log state
- Train Tesseract on MTGO-specific fonts using custom training data
- Preprocess images: grayscale → threshold → denoise → binarize before OCR

**Warning signs:**
- Replay playback shows impossible game states (e.g., negative life, card count >7)
- OCR produces "?" or unusual characters in middle of otherwise clean text
- Logs show inconsistent card names (e.g., "Lightnin Bolt", "Lightnng Bolt")
- No confidence checking in OCR code

**Phase to address:**
Phase 2 - Log capture implementation. Build confidence thresholding and validation before integrating OCR into the main pipeline.

---

### Pitfall 3: Card Recognition False Positives (Misidentified Cards)

**What goes wrong:**
Image matching identifies the wrong card, or finds a card where none exists. A player plays "Counterspell" but the system records it as "Cancel" because both cards share similar art layout and color scheme. Or the system detects a "creature" in an empty space because texture noise resembles card borders.

**Why it happens:**
Card art variations across sets, printings, and foil effects create a massive search space. Even with a hybrid approach (image matching + OCR), developers underestimate the number of variations. Image matching with OpenCV's `matchTemplate()` is sensitive to lighting, rotation, and scale. Using `grayscale=True` for a 30% speedup (as PyScreeze recommends) increases false positives because color is a key differentiator for Magic cards (blue vs white vs black borders).

**How to avoid:**
- Use hybrid recognition: primary method (image matching) + secondary method (OCR text) with agreement required
- Implement confidence thresholds for both methods (e.g., require both >80% confidence)
- Build card database with multiple art variants per card (different printings, foil/non-foil)
- Add context validation: check that recognized card matches game state (e.g., can't have 5 copies of a legendary creature)
- Cache successful recognitions and use them as priors for future matches in same game
- Never rely solely on grayscale matching for card recognition

**Warning signs:**
- Same card recognized differently in successive frames
- Cards appear/disappear without player action
- Recognized card doesn't match zone (e.g., creature in library, land in graveyard)
- No OCR cross-verification for image-based card detection

**Phase to address:**
Phase 3 - Card recognition implementation. Hybrid approach with cross-validation must be built before trusting any card identification.

---

### Pitfall 4: Event Timeline Fragmentation (Unreplayable State)

**What goes wrong:**
Replay file contains events but lacks enough context to reconstruct valid game state. At playback, the viewer can't place cards, resolve conflicts, or show intermediate states. For example, replay shows "Player plays creature" but doesn't show which creature, where it was played from, or what the board looked like before/after.

**Why it happens:**
Developers focus on capturing "events" (card played, ability activated) but neglect "state" (life totals, mana pool, phase, turn number). MTGO's log file is event-heavy but state-light. The computer vision system sees surface changes but doesn't capture the underlying rules engine state. Without explicit state snapshots, replay reconstruction requires inference that can fail in edge cases.

**How to avoid:**
- Capture both event stream AND state snapshots
- Include full board state at game start (all zones, all visible information)
- Include state after each major phase change (main phase → combat, etc.)
- Store references to card definitions (ID, name, type, abilities) not just card names
- Include checksums/hashes to detect corruption
- Design replay format to handle partial information (e.g., "[unknown card in hand]")

**Warning signs:**
- Replay format only stores event list without state
- No way to validate replay integrity during playback
- Replay breaks on first error (no error recovery)
- Can't pause and inspect game state at any point in replay

**Phase to address:**
Phase 2 - Replay format design. State snapshots and validation must be designed into the format before event capture implementation.

---

### Pitfall 5: Window/Frame Desynchronization (Missed Critical Events)

**What goes wrong:**
The capture system reads frames from MTGO window at different times than MTGO renders them, or captures during screen refresh cycles. This leads to missing events (card played between frames) or partial renders (card halfway through draw animation). Replay shows incomplete sequences that are impossible to reconstruct.

**Why it happens:**
MTGO renders at its own framerate while the capture system runs at its own. Without synchronization, captures sample at arbitrary points in MTGO's render cycle. Windows display refresh (60Hz) and MTGO's internal animation timing create multiple timing domains. Frame buffering and vsync add latency between when MTGO "thinks" an event happens and when it appears on screen.

**How to avoid:**
- Use vsync-aware capture timing (capture immediately after display refresh)
- Detect frame changes using checksums rather than fixed-interval captures
- Implement frame buffering: store 2-3 recent frames for when events span multiple frames
- Use MTGO's log file as ground truth for event detection, with CV as supplement
- Add "frame continuity" checks: detect sudden large changes that may indicate missed frames
- Consider Windows-specific synchronization (DirectX/OpenGL capture APIs)

**Warning signs:**
- Cards appear/disappear without animation (indicates missed intermediate frames)
- Card names in log don't match what's shown on screen
- Timestamps in replay file don't align with expected game progression
- No checksum-based change detection in capture pipeline

**Phase to address:**
Phase 1 - Capture synchronization. Frame change detection and vsync handling must be implemented before adding any state recognition logic.

---

## Technical Debt Patterns

| Shortcut | Immediate Benefit | Long-term Cost | When Acceptable |
|----------|-------------------|----------------|-----------------|
| Use full-screen capture instead of window-specific | Simpler code, no window detection needed | 5-10x slower, more data to process, can't multi-task while capturing | Never - performance is critical |
| Skip OCR confidence checking | Faster processing, no threshold tuning | Silent data corruption, unrecoverable replay errors | Never - data integrity is core value |
| Use grayscale-only image matching | 30% faster matching | Increased false positives, can't distinguish by color | Never - card color is essential |
| Store card names as strings only | Simpler format, human-readable | Can't verify card identity, typo issues, localization problems | Only for prototyping, never for production |
| Frame-skipping for performance | Reduces CPU load, smooths framerate | May miss critical events, creates replay gaps | If explicitly designed for (e.g., 15 FPS target with known slow animations) |
| Hardcode MTGO window coordinates | No window detection logic | Breaks if user resizes/moves window, different monitor setups | Only for single-user development environment |
| Skip error recovery in replay | Simpler playback logic | Replay breaks completely on any error | Never - robustness is essential for user trust |

## Integration Gotchas

| Integration | Common Mistake | Correct Approach |
|-------------|----------------|------------------|
| Windows screen capture | Using BitBlt without CAPTUREBLT flag - layered windows/overlays not captured | Use CAPTUREBLT flag or PrintWindow API for complete window capture |
| Tesseract OCR | Loading default English language model for all text | Load and train models specifically for MTGO fonts and character sets |
| OpenCV template matching | Using raw pixel comparison without normalization | Normalize images before matching, use appropriate matching method (TM_CCOEFF_NORMED) |
| MSS screenshot | Capturing all monitors when only MTGO window needed | Use MSS's monitor-specific capture or region parameter for single window |
| File I/O for replay | Writing events as plain text logs | Use binary/structured format with checksums for reliability |
| Thread synchronization | No locks between capture and processing threads | Use thread-safe queues, explicit locks, or actor model for pipeline stages |
| Windows DPI awareness | Ignoring DPI scaling causes coordinate mismatches on high-DPI displays | Set DPI awareness mode before any window/coordinate operations |

## Performance Traps

| Trap | Symptoms | Prevention | When It Breaks |
|------|----------|------------|----------------|
| Python GIL bottleneck | CPU-bound processing scales poorly across threads | Use multiprocessing, or move critical code to compiled extensions (Cython, C++, Rust) | At >2 concurrent threads on CPU-bound operations |
| Memory leak in image buffers | Memory grows steadily during long capture sessions | Explicitly release image objects, use context managers, limit buffer size | After 30+ minutes of continuous capture |
| Excessive file I/O | Replay writing slows down capture thread | Write in batches, use async I/O, buffer events in memory | With >100 events/second or large replay files |
| No frame rate throttling | Capture loop maxes CPU even when nothing changes | Add idle waiting between frames if no change detected | When CPU >50% during idle game states |
| Unnecessary image conversions | Converting color spaces multiple times per frame | Design pipeline with minimal conversions, use native formats where possible | With any image-based recognition (card/OCR) |
| Regex-heavy parsing | Log text parsing becomes bottleneck as game progresses | Pre-compile regex, use string methods for simple patterns, cache common matches | After 10+ turns of game with 50+ cards in play |

## Security Mistakes

| Mistake | Risk | Prevention |
|---------|------|------------|
| No replay file validation | Malicious replay files could crash viewer or execute code | Implement strict schema validation, sandboxed playback environment |
| Storing user credentials in replay | Replay files could leak personal info | Never include usernames, session tokens, or identifiable data in replays |
| No input sanitization in replay viewer | Replay parsing could be exploited for code injection | Treat all replay data as untrusted, use whitelist-based validation |
| Unencrypted network replay sharing | Replays intercepted and tampered with | Sign replays with HMACs, verify signatures on playback |
| Logging sensitive game state | Logs could reveal private information | Log only necessary state, implement log rotation and secure storage |

## UX Pitfalls

| Pitfall | User Impact | Better Approach |
|---------|-------------|-----------------|
| No progress indicator during replay loading | User doesn't know if app is frozen | Show loading bar with estimated time, allow cancel |
| Replay can't be paused/resumed | Can't analyze specific moments | Add playback controls: play, pause, step, speed control |
| No way to skip to key events | Users must watch entire 20-minute game | Add chapter markers, event timeline, search for specific cards/actions |
| Silent failures during capture | User discovers corrupted replay hours later | Real-time error indicators, save partial replays on failure, automatic retries |
| Replay requires exact MTGO version to play | Obsolescence when MTGO updates | Store card data separately from replay, support version-independent playback |

## "Looks Done But Isn't" Checklist

- [ ] **Screenshot speed:** Verify capture <30ms at 1920x1080 using MSS or BitBlt — benchmark before any CV code
- [ ] **OCR confidence:** Verify Tesseract returns confidence scores and threshold is implemented — test with known-failure images
- [ ] **Card cross-validation:** Verify image matching AND OCR both required for card ID — test with similar-looking cards
- [ ] **State snapshots:** Verify replay format includes full game state at intervals — test by reconstructing state from replay
- [ ] **Frame change detection:** Verify capture only processes changed frames — test by checksumming frames
- [ ] **Error recovery:** Verify replay playback doesn't crash on corrupt data — test with malformed replay files
- [ ] **Window positioning:** Verify capture works with MTGO at different positions/sizes — test multi-monitor setups
- [ ] **Long-duration capture:** Verify system runs 1+ hours without memory leaks/crashes — automated stress test
- [ ] **Background operations:** Verify MTGO remains responsive while recording — test during intensive gameplay
- [ ] **Replay shareability:** Verify replay file is self-contained (no external dependencies) — test on different machines

## Recovery Strategies

| Pitfall | Recovery Cost | Recovery Steps |
|---------|---------------|----------------|
| Screenshot bottleneck | HIGH | Switch capture library, implement multi-threading, reduce resolution, redesign capture pipeline |
| OCR hallucinations | MEDIUM | Add confidence thresholds, implement retry logic with different preprocessing, retrain model on MTGO-specific data |
| Card recognition false positives | MEDIUM | Add OCR cross-validation, improve card database with more variants, adjust confidence thresholds |
| Event timeline fragmentation | HIGH | Redesign replay format to include state snapshots, post-process replays to infer missing state |
| Window desynchronization | HIGH | Implement checksum-based change detection, add frame buffering, integrate with MTGO log file |
| Performance degradation over time | MEDIUM | Implement memory profiling, add buffer limits, add periodic restart of capture thread |
| Replay file corruption | HIGH | Implement checksum validation, add automatic backup copies, provide replay repair tools |

## Pitfall-to-Phase Mapping

| Pitfall | Prevention Phase | Verification |
|---------|------------------|--------------|
| Screenshot bottleneck | Phase 1 (Foundation) | Benchmark capture speed before any other code; target <30ms per frame |
| OCR hallucinations | Phase 2 (Log capture) | Unit tests with known-failure OCR images; confidence threshold logging |
| Card recognition false positives | Phase 3 (Card recognition) | Test suite with ambiguous card pairs (e.g., similar art, same color) |
| Event timeline fragmentation | Phase 2 (Replay format design) | Mock replay files with missing state; verify reconstruction succeeds |
| Window desynchronization | Phase 1 (Capture synchronization) | Frame continuity tests; checksum validation across capture runs |
| Performance traps | Phase 4 (Performance optimization) | Profiling during 1-hour capture; memory leak detection tests |
| UX pitfalls | Phase 5 (Viewer polish) | User testing with corrupted replays, long games, edge cases |

## Sources

- PyScreeze documentation - "On a 1920 x 1080 screen, the screenshot() function takes roughly 100 milliseconds"
- PyScreeze documentation - "On a 1920 x 1080 screen, the locate function calls take about 1 or 2 seconds...install opencv...take less than 1 millisecond"
- PyAutoGUI documentation - PAUSE setting adds 0.1s delay by default; failsafe corner triggers
- Python-MSS documentation - Ultra-fast cross-platform screenshots using ctypes; optimized for CV/game capture
- OpenCV documentation - Camera calibration, lens distortion correction, template matching methods
- Windows BitBlt API - CAPTUREBLT flag needed for layered windows; coordinate system transformations
- MTGO community knowledge - Card art variations across sets, foil/non-foil differences, printings
- CNN captcha recognition (nickliqian/cnn_captcha) - OCR preprocessing steps, confidence issues, training requirements
- MTGO gameplay analysis - Event vs state distinction, replay reconstruction challenges

---
*Pitfalls research for: MTGO Replay Recorder*
*Researched: 2026-01-30*
