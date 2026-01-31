# Architecture Research

**Domain:** Game Replay Capture System
**Researched:** 2026-01-30
**Confidence:** MEDIUM

## Standard Architecture

### System Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                        Capture Layer                            │
├─────────────────────────────────────────────────────────────────────┤
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐    │
│  │Window Monitor │  │Screen Capture │  │Frame Buffer │    │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘    │
│         │                  │                  │                │
├─────────┼──────────────────┼──────────────────┼────────────────┤
│         ↓                  ↓                  ↓                │
│  ┌──────────────────────────────────────────────────────┐        │
│  │           Processing Pipeline                   │        │
│  ├──────────────────────────────────────────────────────┤        │
│  │  ┌─────────┐  ┌──────────┐  ┌─────────────┐  │        │
│  │  │Log OCR  │  │Card Match │  │State Update │  │        │
│  │  └────┬────┘  └────┬─────┘  └──────┬──────┘  │        │
│  │       │             │                  │          │        │
│  └───────┼─────────────┼──────────────────┼──────────┘        │
│          ↓             ↓                  ↓                   │
├───────────┼─────────────┼──────────────────┼───────────────────┤
│          ↓             ↓                  ↓                   │
│  ┌──────────────────────────────────────────────────────┐        │
│  │            Event Stream                     │        │
│  └──────────────────────────────────────────────────────┘        │
│                          ↓                                 │
├──────────────────────────────────────────────────────────────────┤
│                          ↓                                 │
│  ┌──────────────────────────────────────────────────────┐        │
│  │           Storage Layer                     │        │
│  │  ┌─────────┐  ┌──────────┐  ┌─────────────┐  │        │
│  │  │Replay   │  │Card DB   │  │Game State   │  │        │
│  │  │File     │  │(Scryfall)│  │Cache        │  │        │
│  │  └─────────┘  └──────────┘  └─────────────┘  │        │
│  └──────────────────────────────────────────────────────┘        │
└─────────────────────────────────────────────────────────────────────┘
```

### Component Responsibilities

| Component | Responsibility | Typical Implementation |
|-----------|---------------|------------------------|
| Window Monitor | Detect MTGO window, track position/size | Windows API (ctypes/winfrey) |
| Screen Capture | Grab frames from MTGO window | Python MSS, PyAutoGUI |
| Frame Buffer | Queue frames for processing | Thread-safe queue (queue.Queue) |
| Log OCR | Extract text from game log panel | Tesseract OCR via pytesseract |
| Card Match | Identify cards on board via image recognition | OpenCV feature matching (ORB/FLANN) |
| State Update | Merge OCR and card data into game state | Event-driven state machine |
| Event Stream | Record game events in compact format | Timestamped event list |
| Replay File | Persistent storage of event stream | JSON/MessagePack/Custom binary |
| Card DB | Reference card images and data | Scryfall API bulk data |
| Replay Viewer | Reconstruct and display games | Event playback engine |

## Recommended Project Structure

```
src/
├── capture/              # Screen capture pipeline
│   ├── window.py         # MTGO window detection/tracking
│   ├── screen.py         # Frame capture (MSS wrapper)
│   └── buffer.py         # Thread-safe frame buffer
├── processing/           # CV/OCR processing
│   ├── ocr/            # Text recognition
│   │   ├── log_reader.py   # Game log OCR
│   │   └── preprocessor.py # Image preprocessing
│   ├── vision/          # Computer vision
│   │   ├── card_matcher.py # Card image matching
│   │   └── board_detector.py # Board state detection
│   └── pipeline.py      # Coordinate processing loop
├── state/               # Game state management
│   ├── events.py        # Event definitions
│   ├── state_machine.py # Track game state changes
│   └── merge.py         # Combine OCR + vision data
├── storage/             # Data persistence
│   ├── replay.py        # Replay file format
│   ├── card_db.py       # Scryfall card database
│   └── cache.py         # In-memory game state cache
├── replay/              # Replay viewer
│   ├── player.py        # Event playback engine
│   └── renderer.py     # Visual reconstruction
├── config/              # Configuration
│   ├── settings.py      # App settings
│   └── card_index.py   # Card reference index
└── main.py              # Application entry point
```

### Structure Rationale

- **capture/**: Isolates platform-specific screen capture logic (Windows API, MSS). Makes testing and porting easier.
- **processing/**: Separates OCR (text) from vision (images) - different concerns, different optimization strategies.
- **state/**: Central to the application - event-based state machine needs clear separation from capture noise.
- **storage/**: Replay format and card database are independent concerns - can swap storage backends without touching processing.
- **replay/**: Viewer is separate from capture - can be different process, different UI framework.

## Architectural Patterns

### Pattern 1: Producer-Consumer Pipeline

**What:** Decouple capture from processing using thread-safe queues. Capture runs as fast producer, processing consumes at own pace.

**When to use:** High-frequency capture with slower processing (OCR is CPU-bound, needs time).

**Trade-offs:**
- Pros: Backpressure handling, parallel execution, graceful slowdown
- Cons: Queue management complexity, memory overhead

**Example:**
```python
# capture/screen.py
import threading
import queue

class CapturePipeline:
    def __init__(self, max_frames=10):
        self.frame_queue = queue.Queue(maxsize=max_frames)
        self.running = False

    def producer_loop(self):
        with mss() as sct:
            while self.running:
                frame = sct.grab(self.mtgo_window)
                self.frame_queue.put(frame, block=False)

    def consumer_loop(self):
        while self.running:
            frame = self.frame_queue.get()
            process_frame(frame)
```

### Pattern 2: Event Sourcing

**What:** Store game state as sequence of events, reconstruct state on replay. Instead of saving snapshots, save "what happened."

**When to use:** Compact replay files, game state reconstruction, timeline navigation.

**Trade-offs:**
- Pros: Dramatic file size reduction, easy scrubbing, diffable
- Cons: Replay must replay all events to get current state, event design is critical

**Example:**
```python
# state/events.py
from dataclasses import dataclass
from datetime import datetime

@dataclass
class GameEvent:
    timestamp: datetime
    event_type: str  # 'card_played', 'life_change', 'turn_begin'
    player: int
    data: dict

# storage/replay.py
class ReplayFile:
    def __init__(self):
        self.events: List[GameEvent] = []

    def add_event(self, event: GameEvent):
        self.events.append(event)

    def save(self, filepath: str):
        # Compact JSON/MessagePack format
        with open(filepath, 'wb') as f:
            msgpack.pack([asdict(e) for e in self.events], f)
```

### Pattern 3: Hybrid Card Recognition

**What:** Combine image matching (for card art) with OCR (for card names) - fallback chain.

**When to use:** Card art varies, need reliable identification, multiple recognition methods available.

**Trade-offs:**
- Pros: Higher recognition rate, handles edge cases, confidence scoring
- Cons: Slower (multiple passes), more complex

**Example:**
```python
# processing/vision/card_matcher.py
import cv2
import pytesseract

class CardMatcher:
    def __init__(self, card_db):
        self.card_db = card_db
        self.orb = cv2.ORB_create()
        self.matcher = cv2.FlannBasedMatcher()

    def identify_card(self, card_image):
        # Try 1: Image feature matching
        matches = self.match_features(card_image)
        if matches and matches[0].distance < threshold:
            return matches[0].card_id

        # Try 2: OCR card name
        text = pytesseract.image_to_string(
            self.extract_name_box(card_image),
            config='--psm 7'  # Single line text
        )
        return self.card_db.lookup_by_name(text.strip())

    def match_features(self, image):
        kp1, des1 = self.orb.detectAndCompute(image, None)
        # FLANN matching with reference card database
        pass
```

## Data Flow

### Capture Flow

```
[MTGO Window]
     ↓
[Window Monitor] → Detect position/size changes
     ↓
[Screen Capture] → Grab frame (MSS)
     ↓
[Frame Buffer] → Queue (bounded, blocks when full)
     ↓
[Processing Pipeline] → OCR + CV (threaded)
```

### State Update Flow

```
[Log OCR] → Parsed log entries
     ↓
[Card Match] → Identified cards on board
     ↓
[State Merge] → Combine into state snapshot
     ↓
[Diff Engine] → Compare with previous state
     ↓
[Event Stream] → Emit delta events only
```

### Replay Flow

```
[Replay File] → Load events
     ↓
[Event Stream] → Iterate in timestamp order
     ↓
[State Machine] → Apply events, build current state
     ↓
[Renderer] → Visualize reconstructed game
```

### Key Data Flows

1. **Capture to OCR:** Raw frames → Preprocessed (threshold/resize) → Tesseract → Text
2. **Capture to Card Match:** Raw frames → Extract regions → ORB features → FLANN match → Card ID
3. **State to Events:** Current state ↔ Previous state → Delta calculation → Event emission

## Scaling Considerations

| Scale | Architecture Adjustments |
|-------|--------------------------|
| 1 user (single game) | Single process, thread pipeline, local card DB |
| 10-100 users (shared storage) | Separate capture/storage, shared card DB (cache), compressed replay files |
| 1000+ users (cloud) | Distributed capture, cloud storage for replays, card DB as service |

### Scaling Priorities

1. **First bottleneck:** Card matching (FLANN on large card database). Fix: Index by set/region, ORB feature caching.
2. **Second bottleneck:** OCR on every frame. Fix: Throttle OCR (only when log panel changes), skip duplicate frames.

## Anti-Patterns

### Anti-Pattern 1: Tight Capture-Processing Coupling

**What people do:** Capture directly calls OCR/CV in same thread, blocking new frames.

**Why it's wrong:** OCR/CV are slow (10-100ms), frame capture is fast (<5ms). Missed frames = dropped game events.

**Do this instead:** Producer-consumer pipeline with bounded queue. Capture fills queue, processing drains independently.

### Anti-Pattern 2: State Snapshots Instead of Events

**What people do:** Save full game state every second. 10-minute game = 600 snapshots.

**Why it's wrong:** Massive file sizes, slow replay scrubbing, storage bloat.

**Do this instead:** Event sourcing - save only what changed. State is reconstructed on replay.

### Anti-Pattern 3: No Frame Duplicates

**What people do:** Process every captured frame, even if nothing changed.

**Why it's wrong:** Wastes CPU, OCR/CV on identical frames, heat generation.

**Do this instead:** Frame differencing - skip frames with <1% pixel change from previous frame.

## Integration Points

### External Services

| Service | Integration Pattern | Notes |
|---------|---------------------|-------|
| Scryfall API | Bulk data sync (daily), reference lookups | Rate limit: 10 req/sec, cache locally |
| Tesseract OCR | Process spawn per OCR call, or persistent engine | --psm mode critical for log vs card names |

### Internal Boundaries

| Boundary | Communication | Notes |
|----------|---------------|-------|
| Capture ↔ Processing | Thread-safe queue (bounded) | Backpressure: drop oldest if queue full |
| Processing ↔ State | Direct function calls (same thread) | Event emission async via callback |
| State ↔ Storage | Batch writes (every N events) | Flush on game end for final save |

## Sources

- Python MSS documentation (official) - Screen capture API: https://python-mss.readthedocs.io/
- PyAutoGUI documentation (official) - Window manipulation: https://pyautogui.readthedocs.io/
- OpenCV documentation (official) - ORB features, FLANN matching: https://docs.opencv.org/
- Tesseract OCR documentation (official) - OCR engine: https://github.com/tesseract-ocr/tesseract
- Pytesseract documentation (GitHub) - Python wrapper: https://github.com/madmaze/pytesseract
- Scryfall API documentation (official) - Card database: https://scryfall.com/docs/api
- OBS Studio documentation (official) - Video capture pipeline patterns: https://obsproject.com/docs

---
*Architecture research for: MTGO Replay Recorder*
*Researched: 2026-01-30*
