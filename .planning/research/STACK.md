# Technology Stack

**Domain:** Windows desktop application with screen capture, OCR, and image recognition
**Researched:** 2025-01-30
**Confidence:** HIGH

## Recommended Stack

### Core Framework

| Technology | Version | Purpose | Why Recommended |
|------------|---------|---------|-----------------|
| Python | 3.12+ | Core runtime | Mature ecosystem for computer vision, excellent libraries for screen capture, OCR, and image processing. Simpler than Electron/Tauri for this use case. |
| PyQt6 | 6.10.1 | Windows desktop GUI | Native-feeling Windows desktop application with proper window management, system tray integration, and lower resource overhead than Electron. |
| MSS | 10.1.0 | Screen capture | Ultra-fast cross-platform screen capture in pure Python using ctypes. Actively maintained (latest release Aug 2025), thread-safe, no dependencies. |
| Tesseract OCR | Latest (via pytesseract) | OCR engine | Open-source OCR engine with excellent text recognition. Python wrapper (pytesseract 0.3.13) provides clean API. Offline capable. |
| OpenCV | 4.13.0.90 | Computer vision / image recognition | Industry-standard library for image processing and computer vision. Includes template matching, feature detection, and deep learning modules. |
| Pillow | 12.1.0 | Image processing | Python Imaging Library fork for efficient image I/O and manipulation. Standard library for image handling in Python. |
| NumPy | 2.4.1 | Array computing | Fundamental package for numerical operations. Required by OpenCV for efficient array operations. |
| pywin32 | 311 | Windows API access | Direct access to Windows APIs including COM, window management, and Desktop Duplication API if needed. |
| Python multiprocessing | Built-in (3.14.2) | Parallel processing | Bypasses GIL for true parallelism. Essential for real-time processing pipeline without blocking UI. |

### Supporting Libraries

| Library | Version | Purpose | When to Use |
|---------|---------|---------|-------------|
| pytesseract | 0.3.13 | Tesseract Python wrapper | Always needed for OCR functionality |
| opencv-python | 4.13.0.90 | OpenCV Python bindings | Always needed for computer vision |
| watchdog | Latest | File system monitoring | When monitoring MTGO log files for game events |

### Development Tools

| Tool | Purpose | Notes |
|------|---------|-------|
| PyInstaller | Packaging Python app | Creates standalone Windows executables |
| pytest | Testing | Unit and integration testing |
| ruff | Linting/formatting | Fast Python linter and formatter |
| mypy | Type checking | Static type checking for better code quality |

## Installation

```bash
# Core desktop framework and image processing
pip install PyQt6 Pillow numpy

# Screen capture
pip install mss==10.1.0

# OCR and computer vision
pip install opencv-python==4.13.0.90 pytesseract==0.3.13

# Windows API access
pip install pywin32==311

# Dev dependencies
pip install pytest ruff mypy pyinstaller

# Optional: File monitoring for log files
pip install watchdog
```

Note: Tesseract OCR binary must be installed separately on Windows. Download from https://github.com/UB-Mannheim/tesseract/wiki

## Alternatives Considered

| Recommended | Alternative | When to Use Alternative |
|-------------|-------------|-------------------------|
| Python + PyQt6 | Electron (JavaScript) | Choose Electron if team is more comfortable with web stack and can accept ~150MB binary size. Electron offers cross-platform at cost of size. |
| Python + PyQt6 | Tauri (Rust) | Choose Tauri if binary size is critical and team knows Rust. Tauri produces smaller binaries (~10MB) but steeper learning curve. |
| MSS | Windows Desktop Duplication API (Direct) | Choose native Desktop Duplication API if you need sub-millisecond capture performance and are comfortable with C++/DirectX integration. |
| Tesseract | Commercial OCR (Azure Vision, Google Vision) | Choose commercial if you need superior accuracy on MTGO-specific text and can pay for API calls. Tesseract is free and offline. |
| OpenCV template matching | Deep learning (YOLO, Faster R-CNN) | Choose deep learning for robust card recognition with variations in art/orientation. Template matching is faster but brittle. |

## What NOT to Use

| Avoid | Why | Use Instead |
|-------|-----|-------------|
| pyscreenshot | Marked as obsolete in documentation. Original issue (PIL Windows-only) resolved in modern Pillow. | MSS or Pillow.ImageGrab |
| Electron + puppeteer | Overkill for Windows-only app. Larger binary size, higher memory usage, unnecessary Node.js dependency chain. | Python + PyQt6 |
| Simple PIL ImageGrab | Less performant than MSS. No optimized multi-monitor support. | MSS |
| threading module for heavy CV work | Python GIL limits true parallelism for CPU-bound image processing. | multiprocessing module |
| Legacy PyQt5 | PyQt6 is current. PyQt5 will receive fewer updates going forward. | PyQt6 |

## Stack Patterns by Variant

**If prioritizing maximum capture performance:**
- Use Windows Desktop Duplication API via pywin32
- Implement capture in C++ with DirectX integration
- Because MSS (while fast) has Python overhead. Native API provides sub-millisecond capture.

**If prioritizing rapid development:**
- Use Python + PyQt6 + MSS + OpenCV
- Because mature ecosystem, one language, all libraries have Python bindings
- Tradeoff: Slightly slower capture than native solution

**If needing cross-platform (future-proofing):**
- Keep Python core, use PyQt6
- Because PyQt6 runs on Windows, macOS, Linux
- Only MSS needs adjustment (already cross-platform)

**If binary size is critical (<20MB):**
- Consider Tauri (Rust) instead of Python
- Because Python runtime + PyInstaller typically produces 50-100MB+ executables
- Tradeoff: Much steeper learning curve, smaller ecosystem for CV libraries

## Version Compatibility

| Package A | Compatible With | Notes |
|-----------|-----------------|-------|
| Python 3.12+ | PyQt6 6.10.1 | PyQt6 supports Python 3.10+ |
| Python 3.12+ | opencv-python 4.13.0.90 | Supports Python 3.6+ |
| Python 3.12+ | Pillow 12.1.0 | Requires Python 3.10+ |
| Python 3.12+ | pytesseract 0.3.13 | Requires Python 3.8+ |
| opencv-python | numpy 2.4.1 | NumPy dependency satisfied |
| MSS | Pillow | MSS integrates with Pillow for image formats |
| pywin32 | Windows API | pywin32 version 311 supports Python 3.8-3.14 |

## Architecture Notes

The recommended stack supports a multi-process architecture:

1. **Main Process (UI):** PyQt6 application handling user interface and configuration
2. **Capture Process:** Dedicated process using MSS for continuous screen capture at target frame rate
3. **Processing Process Pool:** Worker processes (multiprocessing.Pool) for OCR and image recognition
4. **Storage Process:** Background process for writing compact replay files

This architecture ensures UI remains responsive while heavy CV work runs in parallel processes without GIL blocking.

## Sources

- MSS documentation — PyPI package page verified (version 10.1.0, released Aug 16, 2025) — HIGH confidence
- pytesseract documentation — PyPI package page verified (version 0.3.13, released Aug 16, 2024) — HIGH confidence
- opencv-python documentation — PyPI package page verified (version 4.13.0.90, released Jan 18, 2026) — HIGH confidence
- Pillow documentation — PyPI package page verified (version 12.1.0, released Jan 2, 2026) — HIGH confidence
- NumPy documentation — PyPI package page verified (version 2.4.1, released Jan 10, 2026) — HIGH confidence
- PyQt6 documentation — Official docs verified (version 6.10.1) — HIGH confidence
- pywin32 documentation — PyPI package page verified (version 311, released Jul 14, 2025) — HIGH confidence
- Python multiprocessing — Official Python 3.14.2 documentation — HIGH confidence
- Windows Desktop Duplication API — Microsoft Learn documentation (last updated Jan 2021) — MEDIUM confidence (older docs but stable API)
- Electron documentation — Official docs — HIGH confidence
- Tauri v1 documentation — Official docs (v1, not v2) — MEDIUM confidence

---
*Stack research for: Windows desktop application with screen capture, OCR, and image recognition*
*Researched: 2025-01-30*
