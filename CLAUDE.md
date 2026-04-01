# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

XTS_Base is a TwinCAT 3 PLC project for controlling Beckhoff XTS (eXtended Transport System) linear movers. It is an open-source community framework (Zero-Clause BSD) intended as a reusable starting point. The primary language is **IEC 61131-3 Structured Text (ST)** with function blocks (FB) and interfaces.

Key library versions: TF5850 XTS v4.4.5, TF5410 Advanced Motion v3.4.19, TwinCAT XAE v4026.20.1.

## Build & Development

This project is developed in **TwinCAT 3 XAE** (integrated into Visual Studio 2022 via TcXaeShell). There are no command-line build tools — all compilation and deployment is done through the TwinCAT IDE.

- Open `Base Project.sln` in Visual Studio with TcXaeShell installed
- Build via TwinCAT menu → Build Solution or Activate Configuration
- Deploy to runtime via TwinCAT → Activate Configuration → Restart TwinCAT

There are no automated tests — testing is done via runtime debugging in the TwinCAT environment.

## Documentation

Documentation is MkDocs-based in `Documentation/docs/`. To build and serve locally:

```bash
pip install mkdocs-material
cd Documentation
mkdocs serve        # local preview at http://127.0.0.1:8000
mkdocs build        # static site output
```

GitHub Actions (`.github/workflows/main.yml`) auto-deploys docs to GitHub Pages on push to `master`.

## Project Structure

```
Base Project/PLC1/
├── POUs/                   # User application code (MAIN, ApplicationMover)
├── GVLs/                   # Global variable lists (GVL, MainCommands, Param)
├── DUTs/                   # User-defined data types (MachineState, MoverPayload)
├── VISUs/                  # HMI visualization panels
└── XTS (Do Not Edit)/      # Core system library — do not modify
    ├── Objects/            # Core FBs: Mover, MoverList, Station, PositionTrigger, Track, Zone
    ├── Administrative/     # Mediator, ErrorMover, Objective
    ├── Trace/              # Tracing/logging system (iTrace interface)
    ├── DUTs/               # System data types
    └── ITFs/               # System interfaces
```

**The `XTS (Do Not Edit)/` folder is the framework core.** User code lives outside it. This separation enables clean merges when the framework is updated.

## Architecture

### Core Design Principles

1. **Contextual Mover Selection** — Application logic never addresses movers by index. Instead, movers are selected through context objects: `Station`, `PositionTrigger`, `MoverList`, or `Zone`. This makes application code scale-independent.

2. **Single FB_XTS Entry Point** — The entire XTS system is instantiated as one function block (`FB_XTS`) called each PLC cycle. All system objects are accessed through it.

3. **Method Chaining (Fluent Interface)** — Most mover/station methods return `THIS^`, enabling command chains like `Station[1].Mover.MoveToStation(Station[2])`. Order of operations matters.

4. **Automatic Registration** — Movers auto-register with selector objects (Stations, PositionTriggers) when commands are issued. Manual registration is available for advanced scenarios.

5. **Collision Avoidance** — Handled automatically by TF5410. Movers follow behind each other with configurable gaps; no explicit collision logic needed in application code.

### Key Objects

| Object | Role |
|---|---|
| `Mover` | Represents a single physical mover on the track. Exposes motion commands. |
| `MoverList` | Dynamic collection of movers; used for group operations and context selection. |
| `Station` | Fixed position on the track. Movers target stations; provides `.Mover` reference to arrived mover. |
| `PositionTrigger` | Fires at a specific track position, captures the mover passing through. |
| `Zone` | Track segment; provides mover membership queries. |
| `Track` | Represents the physical XTS track hardware. |
| `Mediator` | Central manager handling initialization, recovery, error movers, and system state. |
| `Objective` | Tracks a mover's in-progress motion goal. |
| `ErrorMover` | Represents a faulted mover for targeted recovery. |

### State Machine Pattern

`MAIN.TcPOU` implements an application state machine with actions:
- `Initializing` — Wait for `Mediator.SystemReady`
- `Recovering` / `RecoverOneshot` — Move error movers to reset positions
- `StationLogic` — Main cyclic application logic

### Tracing System

The `Trace/` subsystem replaced the older Event Logger. Use the `iTrace` interface to emit messages at severity levels. `GVL_Trace.Trace` is the global instance. `AdsLogger` and `TcEventLogger` are the backend implementations.

## Contribution Notes

- All user application code belongs **outside** `XTS (Do Not Edit)/`
- `ApplicationMover.TcPOU` is the pattern for extending mover behavior with application-specific payload and logic
- `MoverPayload_typ.TcDUT` is where custom per-mover data fields are added
- `Param.TcGVL` holds all application-tunable constants
