# Reference Implementation Guide

Before substantial implementation in an unfamiliar domain, study serious
production codebases or official reference implementations that match the
problem.

Use this to learn ownership, naming, control flow, testing strategy, and
boundary shape. Do not cargo-cult code, dependency choices, or compatibility
layers that do not fit the current project.

## Suggested Anchors

- Editor, IDE, diagnostics, extension, terminal, and language tooling: study
  VS Code, TypeScript, tree-sitter, LSP implementations, or mature editor
  plugins.
- Media playback, codecs, timing, and streaming: study VLC, FFmpeg, GStreamer,
  mpv, or platform media frameworks.
- Rendering, shaders, GPU resources, and frame scheduling: study Chromium,
  WebKit, Skia, bgfx, wgpu, or mature game engines.
- Audio engines, mixing, scheduling, and DSP: study Web Audio implementations,
  SuperCollider, JUCE, miniaudio, or mature engine audio modules.
- Compilers, interpreters, bytecode, parsers, and language runtimes: study LLVM,
  Lua, V8, CPython, QuickJS, Roslyn, or TypeScript.
- Persistence, storage, transaction, and query behavior: study SQLite, RocksDB,
  PostgreSQL, or mature migration tooling.
- UI layout, input, accessibility, and interaction models: study browser engine
  code, React, Flutter, Qt, or native platform UI frameworks.

## What To Extract

- Where the real owner lives.
- Which boundary is public and which details stay private.
- How state is represented, updated, invalidated, and serialized.
- Which work is precomputed and which work happens in the hot path.
- How fallibility is modeled at true external boundaries.
- Which tests prove the important behavior.

## What To Reject

- Compatibility branches for runtimes the project does not support.
- Generic abstraction layers that exist for ecosystem reasons, not this codebase.
- Defensive checks around impossible internal states.
- Framework-specific ceremony that hides the domain owner.
- Formatting or naming style that conflicts with the current project.
