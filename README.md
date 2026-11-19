<<<<<<< HEAD
#    LUNANAV

> **The AI proposes. The math proves.**

An offline, self-correcting code analysis system that studies obfuscated code, proposes a readable reconstruction, and provides machine-checkable evidence for precisely stated properties.

PALIMPSEST does **not** claim to prove arbitrary programs correct, to prove arbitrary program equivalence, or to reverse every possible obfuscation.

> **Status:** Research prototype / concept stage. Claims below describe design goals and the intended contract. Items marked *(needs evidence)* have not yet been validated by measurement.

---

## Table of Contents

1. [Overview](#overview)
2. [Why PALIMPSEST](#why-palimpsest)
3. [How It Works](#how-it-works)
4. [Trust Model](#trust-model)
5. [What "Proof" Means Here](#what-proof-means-here)
6. [Recommended MVP Scope](#recommended-mvp-scope)
7. [Architecture](#architecture)
8. [Evaluation Plan](#evaluation-plan)
9. [Threat Model and Offline Operation](#threat-model-and-offline-operation)
10. [Risks and Mitigations](#risks-and-mitigations)
11. [Novelty and Differentiators](#novelty-and-differentiators)
12. [NASA-Inspired Engineering Principles](#nasa-inspired-engineering-principles)
13. [Roadmap](#roadmap)
14. [Demo Script](#demo-script)
15. [Non-Negotiable Accuracy Rules](#non-negotiable-accuracy-rules)
16. [References](#references)
17. [Contributing](#contributing)

---

## Overview

PALIMPSEST is a local-first code analysis and reconstruction tool. It combines AI-assisted hypothesis generation with deterministic program analysis and formal verification.

- The **AI** may suggest candidate names, explanations, and transformations.
- The **verification layer** independently checks defined properties and reports the exact limits of each result.
- The AI can never label a reconstruction "correct" merely because it looks plausible.

**Core promise:** For supported languages and transformations, PALIMPSEST proposes candidate reconstructions and proves specific equivalence properties under explicit assumptions, or clearly reports when proof is unavailable.

The system is designed to work offline after installation, keeping source code on the user's machine.

## Why PALIMPSEST

Obfuscated code is hard to audit, maintain, debug, or assess. Common transformations include identifier renaming, control-flow flattening, dead-code insertion, constant encoding, and expression restructuring. Some preserve behavior; others may change behavior or conceal malicious intent.

Key challenges:

1. **Recover structure:** identify functions, control-flow regions, data dependencies, and likely transformations.
2. **Preserve semantics:** avoid "cleaning up" code in ways that change observable behavior.
3. **Separate evidence from guesses:** distinguish syntax recovery, static-analysis findings, AI hypotheses, tests, and formally verified claims.
4. **Handle uncertainty:** report unsupported features, dynamic behavior, missing dependencies, and unresolved paths.
5. **Stay local:** protect source code and operate without a network connection.
6. **Resist adversarial input:** treat analyzed code as untrusted data, never as instructions to the AI or the analysis environment.

## How It Works

1. **Input and isolation**
   - Accept a source file or project snapshot.
   - Record hashes, language/version, dependencies, and analysis configuration.
   - Parse in a restricted, non-executing environment by default.
2. **Structural analysis**
   - Build an AST, control-flow graph (CFG), and data-flow representation where supported.
   - Detect known obfuscation patterns.
   - Mark dynamic or unsupported constructs explicitly.
3. **Candidate generation**
   - A local language model proposes names, explanations, and candidate transformations.
   - The original source stays immutable.
   - Each proposal carries a rationale; confidence is not proof.
4. **Deterministic checks**
   - Parser/type checks where available.
   - Control-flow and data-flow comparison.
   - Symbolic execution, SMT solving, or translation validation for supported subsets.
   - Differential tests as supporting evidence only, never as universal proof.
5. **Proof and evidence report**
   - States the exact property checked, assumptions, tool/version, scope, and result.
   - Labels each result `PROVEN`, `TESTED`, `HEURISTIC`, or `UNKNOWN`.
   - Reports timeouts and unsupported behavior as such.
6. **Human review and export**
   - Side-by-side original and proposed code with changed regions and proof status highlighted.
   - Export of reconstructed code plus a reproducible evidence bundle.

## Trust Model

| Output class | Meaning | Can it be called a proof? |
|---|---|---|
| **PROVEN** | A sound verifier established a stated property within declared assumptions and scope | Yes, only for that property and scope |
| **TESTED** | Selected tests passed | No |
| **HEURISTIC** | AI or pattern matching suggests a likely interpretation | No |
| **UNKNOWN** | Analysis could not establish the property | No |

A timeout, unsupported feature, or missing dependency is **never** converted into a success.

## What "Proof" Means Here

Program equivalence is hard and, for unrestricted languages, undecidable in general. PALIMPSEST therefore proves **narrow, precisely stated contracts**, for example:

- **Expression equivalence:** two expressions produce the same result for all inputs in a declared domain.
- **Local transformation validation:** a proposed rewrite preserves the semantics of a specific code region.
- **Bounded equivalence:** two programs behave identically for all inputs within stated bounds.
- **Invariant proof:** a stated invariant holds under specified preconditions.
- **Translation validation:** independently checking that a particular transformed output preserves the relevant semantics.

Every proof report must include:

- the exact claim and preconditions,
- assumptions,
- language subset and runtime model,
- unsupported features encountered,
- verifier name and version,
- result, timeout, or failure reason.

A proof of one property must never be presented as proof of all program behavior.

## Recommended MVP Scope

Prefer a small, working, measurable prototype over an ambitious but unverifiable architecture.

- **One language** with a clearly defined supported subset *(unresolved design decision: language choice)*.
- **2–4 semantics-preserving obfuscation patterns**, e.g. identifier renaming, dead-code insertion, constant encoding, expression restructuring.
- **One rigorous verification path**, e.g. SMT-based equivalence checking of pure/loop-free regions *(unresolved design decision: solver and encoding)*.
- Everything outside the subset returns `UNKNOWN`.

## Architecture

Modular components, each independently testable and replaceable:

```
 Source ──► Parser ──► AST / CFG / Data-flow ──► Pattern Detector
                                                       │
                          Local Model ◄────────────────┤
                              │                        │
                              ▼                        │
                     Candidate Transformer             │
                              │                        │
                              ▼                        ▼
                     Verifier (SMT / symbolic / translation validation)
                              │
                              ▼
                 Evidence Report  ──►  UI / Export (code + evidence bundle)
```

| Component | Responsibility |
|---|---|
| Parser | Non-executing parse; record hashes and configuration |
| AST/CFG builder | Structural and data-flow representation; flag dynamic constructs |
| Pattern detector | Recognize known obfuscation patterns |
| Local model | Propose names, explanations, candidate rewrites (proposal only) |
| Candidate transformer | Apply proposals to a copy; original remains immutable |
| Verifier | Independent, deterministic checking; the only source of `PROVEN` |
| Evidence report | Claim, assumptions, versions, results, unresolved issues |
| UI | Side-by-side diff with per-region proof status |

## Evaluation Plan

Measure on a curated benchmark with baselines:

- Reconstruction accuracy
- Percentage of transformations that can be verified
- **False-proof rate (target: zero)**
- Unsupported / `UNKNOWN` rate
- Runtime and peak memory
- Offline behavior and observed network activity
- Reviewer time to understand a result

Include deliberately incorrect candidates (which must be rejected), solver timeouts, and unsupported-construct cases as explicit failure tests. Benchmark design and baselines are *(needs evidence / to be defined)*.

## Threat Model and Offline Operation

- Analyzed code is **untrusted data**, not instructions to the model or tooling.
- Parsing is non-executing by default; any optional execution happens in a sandbox.
- Pin dependencies and audit for telemetry.
- **Offline privacy is claimed only if network behavior is tested.** The demo includes repeating analysis with the network disconnected, and tests should confirm no outbound connections.

## Risks and Mitigations

| Risk | Mitigation |
|---|---|
| AI invents a plausible but incorrect reconstruction | AI output is proposal only; independent validation gates proof labels |
| Dynamic language behavior defeats static analysis | Mark dynamic features; return `UNKNOWN` |
| Solver timeout or state explosion | Bound analysis, expose timeout, never convert it to success |
| Tests miss edge cases | Label tests as tests, not proofs; combine with formal checks |
| Malicious input attacks tooling | Parse without execution; sandbox optional execution |
| "Offline" claim undermined by telemetry/dependencies | Audit network access, pin dependencies, provide an offline test |
| NASA connection is overstated | Cite primary NASA sources; state clearly the project is independent |

## Novelty and Differentiators

The contribution is the **separation of proposal from proof**:

- AI generates hypotheses; deterministic tools decide what can be verified.
- Proof status is attached to each transformation, not just the final file.
- The original program remains immutable and every change is traceable.
- Uncertainty is a first-class output, not hidden behind a confidence score.
- Offline operation supports sensitive code and disconnected environments.
- A reproducible evidence bundle lets another engineer inspect how a conclusion was reached.

AI deobfuscation, symbolic execution, SMT solving, and offline code models are not individually new. The contribution is the integrated workflow, proof-scoped user experience, and measured evaluation.

## NASA-Inspired Engineering Principles

PALIMPSEST is an **independent concept** that adapts engineering principles seen in NASA contexts: verification planning, objective evidence, traceability, and modular design.

> **Accuracy boundary:** These sources provide engineering context and design inspiration only. They do **not** establish NASA sponsorship, partnership, endorsement, use, or validation of PALIMPSEST. PALIMPSEST is not flight-qualified, and no claim is made that it solves a named NASA mission requirement.

| Source | Connection to PALIMPSEST |
|---|---|
| NASA Software Engineering Handbook, SWE-028 Verification Planning | Define in advance what is verified, by which methods, with what success criteria |
| NASA Software Engineering Handbook, SWE-029 Validation Planning | Separate verification (meets spec) from validation (useful for intended task) |
| NASA Software Engineering Handbook, SWE-030 Verification Results | Export reproducible evidence: claim, assumptions, tool versions, checks, unresolved issues |
| NASA article, "Identifying Objective Evidence Improves Requirement Implementation" (Mar 25, 2026) | Link each transformation to its rationale, verification method, and result |
| NASA Core Flight System (cFS), Goddard ETD | Architectural example of modularity and clear interfaces; not a deobfuscation tool |
| NASA Systems Engineering Handbook, NASA/SP-2016-6105 Rev. 2 | Requirements, risks, verification plans, and measurable acceptance criteria for the prototype |

> *(Needs evidence: each URL below should be re-verified against the exact claim it supports before external use.)*

## Roadmap

1. **More language frontends**, added only when semantics and tooling are supported.
2. **Proof-carrying patches:** each proposed change packaged with a machine-checkable validation artifact and assumptions.
3. **Differential toolchain validation:** compare independent parsers, compilers, or verifiers where practical.
4. **Constrained-compute mode:** benchmark compact local models and analysis on limited CPU, RAM, and power.
5. **Mission-software audit profile:** configurable traceability and evidence templates (an audit aid, not a certification).
6. **Malware-aware analysis sandbox:** inspect suspicious samples without executing them on the host.
7. **Team review mode:** signed reports, reproducible runs, reviewer annotations.
8. **Explainable uncertainty:** identify exactly which constructs block a proof and suggest a narrower checkable property.
9. **Fault-aware analysis:** treat timeouts, parser failures, unsupported syntax, and missing dependencies as explicit states.

## Demo Script

1. Show a small, intentionally obfuscated input.
2. Show the AST/CFG or detected patterns.
3. Let the local model propose a readable candidate.
4. Display the exact transformation diff.
5. Run the verifier on the supported subset.
6. Show a proof report with assumptions and scope.
7. Demonstrate a deliberately incorrect candidate being rejected.
8. Disconnect the network and repeat the analysis to substantiate offline operation.

## Non-Negotiable Accuracy Rules

- Never say the system proves arbitrary program equivalence.
- Never call AI confidence a mathematical proof.
- Never turn a timeout, unsupported feature, or missing dependency into a successful result.
- Use proof claims only when the verifier and assumptions justify them.
- Do not invent NASA programs, datasets, mission requirements, partnerships, or endorsements.
- Cite primary sources with exact titles, dates where available, and verified URLs.
- If a source does not support a claim, remove or qualify it.
- Mark assumptions and unresolved design decisions explicitly.

## References

**NASA sources (starting points; verify before citing):**

1. NASA Software Engineering Handbook, SWE-028 Verification Planning: https://swehb.nasa.gov/spaces/7150/pages/16450576/SWE-028%2B-%2BVerification%2BPlanning
2. NASA Software Engineering Handbook, SWE-029 Validation Planning: https://swehb.nasa.gov/pages/viewpage.action?pageId=23429440
3. NASA Software Engineering Handbook, SWE-030 Verification Results: https://swehb.nasa.gov/pages/viewpage.action?pageId=16457190
4. "Identifying Objective Evidence Improves Requirement Implementation," NASA, March 25, 2026: https://sma.nasa.gov/news/articles/newsitem/2026/03/25/identifying-objective-evidence-improves-requirement-implementation
5. NASA Core Flight System (cFS), Goddard Engineering and Technology Directorate: https://etd.gsfc.nasa.gov/capabilities/core-flight-system/
6. NASA Systems Engineering Handbook, NASA/SP-2016-6105 Rev. 2: https://www.nasa.gov/reference/systems-engineering-handbook/ (NTRS record, Feb 17, 2017: https://ntrs.nasa.gov/archive/nasa/casi.ntrs.nasa.gov/20170001761.pdf)

**Formal-methods references:** *(to be added: SMT solving, symbolic execution, and translation validation primary sources)*

## Contributing

This is an early-stage research prototype. Contributions that improve rigor are especially welcome: benchmark cases, verifier integrations, failure-case tests, and corrections to any overstated claim. When opening a PR, state exactly which property is checked and under what assumptions.

=======
# 🌕 LunaNav – Lunar Landing Site Navigator

### Finding the Right Place. Choosing the Right Time. Enabling the Next Giant Leap.

LunaNav is an interactive lunar mission-planning platform designed to help evaluate and compare potential landing sites and landing dates near the Moon's South Pole.

By combining lunar terrain information, sunlight availability, and Earth communication visibility, LunaNav helps mission planners understand environmental conditions and assess landing feasibility before a mission begins.

---

## 🌌 The Big Idea

A successful lunar landing is not just about finding a suitable location. It is also about choosing the right time.

The Moon's South Pole contains scientifically valuable regions and potential resources. However, its challenging terrain, long periods of darkness, and communication constraints make landing-site selection difficult.

**LunaNav brings location and time together in one interactive decision-support platform.**

It helps answer three important questions:

- Where should a spacecraft land?
- When should the landing take place?
- How suitable and reliable is that landing scenario?

---

## 🚀 The Problem

Selecting a lunar landing site involves several challenges:

| Challenge | Why it matters |
|---|---|
| Uneven terrain | Slopes, craters, and obstacles can affect landing safety |
| Sunlight availability | Solar-powered systems require sufficient sunlight |
| Long periods of darkness | Extended darkness can create power and thermal challenges |
| Earth visibility | Communication with Earth requires a suitable line of sight |
| Landing date selection | Environmental conditions change with time |

Evaluating these factors separately makes mission planning more complicated.

## ✨ Our Solution

LunaNav provides a unified environment to compare multiple lunar landing scenarios.

Users can select different locations and dates, examine environmental conditions, and evaluate their suitability using clear visualizations and requirement-based analysis.

### Core Features

**1. Multi-Scenario Comparison**

- Compare 2–4 landing-site and date combinations.
- View sunlight availability and darkness duration.
- Compare communication visibility and environmental conditions.
- Identify the most suitable scenario based on selected requirements.

**2. Terrain-Aware Analysis**

- Analyze lunar surface elevation and terrain.
- Examine local horizon conditions.
- Identify potential terrain-related landing risks.
- Consider surrounding terrain when evaluating sunlight visibility.

**3. Schedule and Feasibility Analyzer**

- Evaluate whether a selected landing scenario satisfies mission requirements.
- Analyze sunlight and communication constraints.
- Identify possible scheduling limitations.
- Explore how changes in landing dates affect mission feasibility.

### Additional Decision-Support Capabilities

**Requirement-Based Suitability Heatmap**

Visualize landing suitability using an intuitive color system:

- 🟢 Green – Suitable under the selected requirements.
- 🟠 Orange – Potential risk or constraint.
- 🔴 Red – One or more critical requirements are not satisfied.

**Schedule-Shift Tolerance**

Understand how far a landing date can be shifted while still satisfying mission requirements.

**Landing-Ellipse Robustness**

Explore how uncertainty in the spacecraft's final landing position could affect the suitability of the selected region.

**Explainable Results**

Instead of showing only a final score, LunaNav aims to explain why a scenario is considered suitable, risky, or unsuitable.

---

## 🛰️ How LunaNav Works

1. **Select Locations** – Choose potential landing regions near the lunar South Pole.
2. **Choose Dates** – Define the planned landing dates.
3. **Analyze Terrain** – Examine elevation and surrounding horizon conditions.
4. **Evaluate Sunlight** – Estimate sunlight availability and darkness periods.
5. **Check Communication** – Evaluate Earth visibility and communication windows.
6. **Compare Scenarios** – Examine multiple landing options side by side.
7. **Assess Feasibility** – Evaluate requirements and understand potential risks.

---

## 📡 NASA Data Sources

LunaNav is designed to use relevant scientific datasets and planetary-analysis resources.

| Data Source | Purpose |
|---|---|
| NASA Lunar Reconnaissance Orbiter (LRO) | Lunar surface and mission data |
| Lunar Orbiter Laser Altimeter (LOLA) | Elevation and terrain analysis |
| NASA SPICE | Spacecraft geometry and position calculations |
| JPL Horizons | Astronomical position and ephemeris information |
| NASA Moon Trek | Lunar surface exploration and visualization |
| NASA QuickMap | Lunar geographic and scientific data exploration |

*The exact datasets and services used depend on the implemented analysis modules.*

---

## 🎯 Who Can Use LunaNav?

- Lunar mission planners and researchers
- Space science students and educators
- Researchers exploring lunar landing conditions
- Space enthusiasts interested in lunar exploration

---

## 🌍 Expected Impact

LunaNav aims to make complex lunar landing considerations easier to understand through an interactive and explainable analysis platform.

Its goal is to support better-informed decisions by bringing terrain, sunlight, communication, and scheduling considerations together.

---

## 🛠️ Technology Stack

The technology stack will be updated to reflect the final implementation.

- **Frontend:** To be specified
- **Backend:** To be specified
- **Programming Language:** To be specified
- **Data Processing:** To be specified
- **Scientific Data:** NASA and JPL resources
- **Visualization:** To be specified

---

## 🚀 Getting Started

Clone the repository:

```bash
git clone https://github.com/karthi08nkl/LunaNav_Project_NASA.git
cd LunaNav_Project_NASA
```

Install dependencies according to the project's configuration:

```bash
pip install -r requirements.txt
```

Start the application using the appropriate command for the implemented framework.

*Update these instructions once the project structure and execution commands are finalized.*

---

## 👥 Team

**Project:** LunaNav – Lunar Landing Site Navigator

**Challenge:**
>>>>>>> aeefcaee7d0157568f133194c5d2c21373dcde83
