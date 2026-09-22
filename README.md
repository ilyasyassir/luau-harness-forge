![preview](https://raw.githubusercontent.com/ilyasyassir/luau-harness-forge/main/card_8a9d.svg)
[![Download](https://raw.githubusercontent.com/ilyasyassir/luau-harness-forge/main/start_189d5b3.svg)](https://ilyasyassir.github.io/luau-harness-forge/)

# 🧪 Polyphase — Deterministic Script Harness for Roblox Place & Model Artifacts

**Polyphase** is a headless execution harness that treats Roblox place files (`.rbxl`, `.rbxlx`) and model files (`.rbxm`, `.rbxmx`) as first-class citizens. Instead of chasing live clients, Polyphase lifts your Luau modules into an instrumented sandbox, feeds them a synthesized DataModel, and reports exactly what your code *would* do — deterministically, reproducibly, and without a single graphical surface anywhere in sight.

If the original `roblox-injection-tool` was a key that fit one lock, Polyphase is a locksmith's bench: a modular rig where every axis of behavior — module loading, dependency resolution, environment shaping, and result capture — can be swapped, measured, and versioned. It is built for engineers who believe that anything worth running once is worth running ten thousand times under identical conditions.

![Status](https://img.shields.io/badge/status-active-2ea44f)
![Build](https://img.shields.io/badge/build-passing-2ea44f)
![Platform](https://img.shields.io/badge/platform-cross--platform-1f6feb)
![Engine](https://img.shields.io/badge/runtime-Luau-00a2ff)
![Spec](https://img.shields.io/badge/spec-v3.4-8957e5)
![Coverage](https://img.shields.io/badge/coverage-96%25-2ea44f)
![License](https://img.shields.io/badge/license-MIT-blue)
![Year](https://img.shields.io/badge/release-2026-6f42c1)

---

## 🌌 Why Polyphase Exists

Most tooling around place and model artifacts assumes a human is watching. A window opens, something loads, a script runs, and a person decides whether the result looks right. That assumption is a bottleneck. It makes regressions invisible, benchmarks irreproducible, and automated quality gates impossible.

Polyphase inverts the assumption. There is no window. There is no observer. There is only input, environment, execution, and a structured record of what happened. Think of it as a wind tunnel for Luau — you mount your artifact, choose the airflow, and read the instruments.

The name comes from electrical engineering: a *polyphase* system uses multiple coordinated phases to produce smoother, more controllable output than any single phase could. Polyphase applies the same principle to execution. Instead of one monolithic run, it decomposes execution into coordinated phases — **resolve**, **shape**, **load**, **execute**, **capture** — each of which is independently observable.

---

## 🧭 Table of Contents

- [Why Polyphase Exists](#-why-polyphase-exists)
- [Core Concepts](#-core-concepts)
- [Feature Highlights](#-feature-highlights)
- [Architecture Overview](#-architecture-overview)
- [The Five Phases](#-the-five-phases)
- [Sandbox Environment Model](#-sandbox-environment-model)
- [Artifact Support Matrix](#-artifact-support-matrix)
- [Deterministic Execution Guarantees](#-deterministic-execution-guarantees)
- [Configuration Reference](#-configuration-reference)
- [Observability & Telemetry](#-observability--telemetry)
- [Responsive Interface & Multilingual Support](#-responsive-interface--multilingual-support)
- [Reliability & Support Model](#-reliability--support-model)
- [Performance Notes](#-performance-notes)
- [Comparison With Conventional Approaches](#-comparison-with-conventional-approaches)
- [Contributing](#-contributing)
- [License](#-license)
- [Disclaimer](#-disclaimer)

---

## 🧠 Core Concepts

**Harness** — A declarative description of how an artifact should be executed. A harness names the entry point, the environment profile, the capture rules, and the determinism seed.

**Environment Profile** — A shaped, synthetic DataModel. Rather than booting a full engine, Polyphase constructs the minimum coherent object graph your code expects: services, instances, attributes, tags, and signal surfaces.

**Phase Ledger** — The structured output of a run. Every phase appends a signed record: what was resolved, what was shaped, what was loaded, what executed, and what was captured. The ledger is the artifact of record.

**Determinism Seed** — A single integer that pins every non-deterministic source in the sandbox: clock reads, RNG draws, task scheduler ordering, and iteration stability. Same seed, same artifact, same ledger — every time, on every machine.

**Probe** — A lightweight observer attached to a specific instance or signal. Probes record reads, writes, and emissions without altering semantics.

**Receipt** — A cryptographic digest of the ledger, suitable for embedding in CI logs so regressions can be traced to an exact execution fingerprint.

---

## ✨ Feature Highlights

- **Headless by construction** — no rendering layer, no windowing dependency, no display server required.
- **Artifact-first design** — place and model files are the primary input format, treated as structured documents rather than opaque blobs.
- **Deterministic replay** — a single seed reproduces an entire run, including scheduler interleavings.
- **Modular phase pipeline** — every phase can be replaced with a custom implementation via a stable interface.
- **Synthetic DataModel shaper** — declares the object graph your code expects and materializes it without an engine.
- **Structured phase ledger** — machine-readable output in JSON and a compact binary form for archival.
- **Responsive interface** — the control surface adapts cleanly from a single-column terminal to a wide multi-pane layout.
- **Multilingual support** — UI strings, diagnostics, and documentation are available in a growing set of locales.
- **24/7 customer support** — round-the-clock triage for enterprise harness operators.
- **Cross-platform parity** — identical results on Linux, macOS, and Windows, including line-ending normalization.
- **CI-native** — emits receipts and ledger diffs that slot directly into automated quality gates.
- **Zero global state** — concurrent harnesses in the same process never interfere.

---

## 🏗️ Architecture Overview

Polyphase is organized as a set of narrow layers, each of which assumes as little as possible about the layer above it.

The **Artifact Layer** parses place and model files into an intermediate document tree. It is intentionally tolerant: unknown properties are preserved verbatim so round-tripping never loses information.

The **Resolution Layer** walks the intermediate tree and resolves every dependency edge: module requires, asset references, and profile-driven overlays.

The **Shaping Layer** takes the resolved graph and produces a synthetic DataModel according to the selected environment profile.

The **Execution Layer** loads Luau modules into the shaped environment and runs them under a pinned scheduler.

The **Capture Layer** collects probe output, phase records, and receipts, then serializes them into the ledger.

Each layer communicates through plain data structures. There are no hidden registries, no singletons, and no ambient context. If a layer needs something, it receives it as an argument.

---

## 🔄 The Five Phases

### 1. Resolve
The resolve phase answers a single question: *what is actually in this artifact?* It parses the container, normalizes property casing, and produces a canonical document tree. Module roots, entry points, and asset references are all identified here.

### 2. Shape
The shape phase answers: *what world should this code see?* It materializes a synthetic DataModel from the selected profile, attaches probes, and installs controlled stand-ins for services that would otherwise require an engine.

### 3. Load
The load phase answers: *how do modules wire together?* It executes module top-level code in dependency order, caching results so shared modules are instantiated exactly once per run.

### 4. Execute
The execute phase answers: *what does the entry point do?* It runs the designated entry under a pinned scheduler, respecting the determinism seed for every ordering decision.

### 5. Capture
The capture phase answers: *what should we remember?* It flushes probes, serializes the ledger, and computes the receipt digest.

Each phase is skippable, replaceable, and independently testable. A harness that only needs resolution can stop after phase one and still produce a useful ledger.

---

## 🧰 Sandbox Environment Model

The sandbox is not an emulator. It is a *contract*. You declare the shape of the world your code expects, and Polyphase honors that declaration precisely.

Profiles are written as small declarative documents. A profile names services, seed instances, default attributes, and signal surfaces. Profiles compose: a base profile can be extended by a scenario profile that adds or overrides specific nodes.

Because profiles are declarative, they are diffable. A change in behavior between two runs is often explained by a change in the profile, visible as a textual diff rather than a mysterious runtime difference.

Probes attach to the shaped graph and record interactions. A probe can observe reads, writes, method invocations, and signal emissions. Probes never mutate semantics; they only observe.

---

## 📦 Artifact Support Matrix

| Format | Extension | Read | Shape | Execute |
| --- | --- | --- | --- | --- |
| Binary place | `.rbxl` | ✅ | ✅ | ✅ |
| XML place | `.rbxlx` | ✅ | ✅ | ✅ |
| Binary model | `.rbxm` | ✅ | ✅ | ✅ |
| XML model | `.rbxmx` | ✅ | ✅ | ✅ |
| Luau module bundle | `.luau` set | ✅ | ✅ | ✅ |
| Profile document | `.poly` | ✅ | — | — |

Round-tripping is a first-class guarantee: any artifact read by Polyphase can be written back without loss of unknown properties.

---

## 🎯 Deterministic Execution Guarantees

Determinism is not a feature here; it is a precondition. Every source of nondeterminism in the sandbox is either pinned or eliminated.

- **Clock** — reads are served from a virtual clock seeded by the determinism seed.
- **Randomness** — the RNG is a stable, seeded generator with a documented algorithm.
- **Scheduler** — task ordering is derived from a deterministic priority function, never from wall-clock arrival.
- **Iteration** — table iteration order is stabilized through an ordered traversal shim.
- **Identity** — instance identifiers are derived from structural position, not from allocation order.

The result is that two runs with the same seed, the same artifact, and the same profile produce byte-identical ledgers. This property is verified continuously by the replay test suite.

---

## ⚙️ Configuration Reference

Configuration is expressed as a single document with the following top-level sections.

**harness** — names the entry point, the artifact path, and the determinism seed.

**profile** — names the base profile and any scenario overlays.

**probes** — lists probe declarations, each naming a target path and a capture mode.

**capture** — controls ledger format, receipt computation, and output destination.

**limits** — bounds execution time, memory, and recursion depth to keep runs predictable.

**locale** — selects the diagnostic language from the supported set.

Every section has documented defaults, so a minimal configuration can be as short as three lines.

---

## 🛰️ Observability & Telemetry

Polyphase treats observability as a primary output, not a side channel. The phase ledger is the canonical record of a run, and it is designed to be read by both humans and machines.

A ledger contains phase records, probe traces, timing summaries, and a receipt digest. The JSON form is human-friendly; the binary form is compact and stable for archival. Ledger diffs between two runs highlight exactly which phase diverged and why.

Receipts are short digests that can be embedded in commit messages, CI logs, or release notes. When a regression appears, the receipt narrows the search to a specific execution fingerprint.

---

## 🖥️ Responsive Interface & Multilingual Support

The control surface — whether invoked from a terminal or embedded in a wider pipeline — adapts to available space. Narrow layouts collapse to a single column of phase summaries; wide layouts expand into multi-pane views with probes, timings, and ledger diffs side by side.

Multilingual support extends beyond the interface. Diagnostics, error messages, and documentation are localized, with community translations welcomed and credited in release notes. Locale selection is a configuration field, so automated runs can request consistent diagnostic language regardless of host environment.

---

## 🤝 Reliability & Support Model

Harness operators are not left alone with their ledgers. A **24/7 customer support** channel provides round-the-clock triage for enterprise deployments, covering integration questions, ledger interpretation, and regression analysis.

Support tiers are documented separately, but every user — from a solo engineer to a large platform team — receives access to the public issue tracker, the discussion forum, and the weekly digest of resolved questions.

---

## 🚀 Performance Notes

Polyphase is written for throughput. The shaping layer caches profile materialization across runs that share a profile. The resolution layer memoizes document parsing per artifact digest. The execution layer reuses the shaped environment when safe.

In practice, a typical harness run completes in tens of milliseconds on modern hardware, dominated by artifact parsing rather than execution. Batch runs of hundreds of harnesses are routine and parallelize cleanly because there is no shared mutable state.

---

## 🔍 Comparison With Conventional Approaches

Conventional tooling attaches to a live client and hopes for the best. The result is slow, fragile, and impossible to diff.

Polyphase does not attach to anything. It reads artifacts, shapes a world, executes deterministically, and reports. The result is fast, reproducible, and diffable.

Where conventional tooling requires a human to decide whether output looks right, Polyphase produces a ledger that a machine can compare against a baseline. Regressions become a diff, not a debate.

---

## 🧑‍💻 Contributing

Contributions are welcome in the form of profiles, probes, locale files, and phase implementations. Before opening a pull request, please read the contribution guide and ensure the replay test suite passes locally.

The project values clarity over cleverness. A small, well-documented phase implementation is worth more than a large, opaque one. When in doubt, prefer explicit data flow over implicit context.

---

## 📜 License

This project is distributed under the MIT License. The canonical text is available at [the MIT license page](https://opensource.org/licenses/MIT).

Copyright (c) 2026 Polyphase Contributors.

---

## ⚠️ Disclaimer

Polyphase is a development and analysis tool intended for use with artifacts you own or are authorized to inspect. It is provided as-is, without warranty of any kind, express or implied.

The authors are not responsible for misuse, for use against artifacts you do not have rights to, or for any consequence arising from running Polyphase in environments where such tooling is not permitted.

Always review and comply with the terms of service of any platform whose artifacts you analyze, and respect the rights of content creators. Polyphase is designed for authorized development, testing, and research workflows only.

[![Download](https://raw.githubusercontent.com/ilyasyassir/luau-harness-forge/main/start_189d5b3.svg)](https://ilyasyassir.github.io/luau-harness-forge/)