# Think Inside the Box

> **Ship the docs in the box:** A proposal for co-packaging code and AI-optimized documentation as an industry standard.

## Overview

This repository contains a white paper proposing **co-packaged documentation** — the practice of shipping structured, AI-optimized documentation alongside library code within the same distributable package artifact.

## The Problem

AI coding assistants (GitHub Copilot, Claude, etc.) are transforming software development, but they operate without the contextual knowledge they need when consuming libraries. Today's packages contain code but not documentation — the docs live on websites that AI agents struggle to access and consume. This gap forces agents to rely on stale training data, producing hallucinated APIs, deprecated patterns, and incorrect usage.

## The Solution

The **Think Inside the Box Principle**: Everything a consumer needs should arrive in one box — the code *and* the documentation, version-matched and ready to use.

We propose a new industry standard:
- **A `docs/` directory** in every package containing structured markdown documentation
- **An entry point** (`README.md`) describing the 5 Ws of the library
- **Generated, not duplicated** — documentation sourced from existing docs, not maintained separately
- **Auto-discoverable** — AI tools can detect and use it without configuration

## Why This Matters

- **Correct code immediately** — AI agents use the right APIs for the installed version
- **Offline capability** — Works in air-gapped environments, CI pipelines, restricted networks
- **Zero configuration** — No MCP servers, no API keys, no external tools
- **Version-matched** — Documentation always describes the exact installed version

## Reference Implementation

Design Factory design system ships a complete AI documentation system in its npm package under `.ai/` directory, covering 56 components with APIs, examples, guidelines, and demos. Read the full white paper for architectural details and lessons learned.

## What's in the White Paper

1. **Problem analysis** — Why the current code/documentation separation fails AI agents
2. **Reference implementation** — Design Factory's working approach
3. **Proposed standard** — The CoDoc (Co-packaged Documentation) specification
4. **Security considerations** — Addressing prompt injection risks
5. **Path forward** — How library authors, registries, and tool vendors can adopt this

## Reading the Paper

The complete white paper is available at [white-paper.md](white-paper.md).

Key sections:
- [Section 3](white-paper.md#3-the-think-inside-the-box-principle) — The core principle
- [Section 4](white-paper.md#4-design-factory-a-reference-implementation) — Design Factory implementation
- [Section 7](white-paper.md#7-proposed-standard-co-packaged-documentation-codoc) — The CoDoc specification
- [Section 8](white-paper.md#8-security-considerations) — Security and prompt injection risks

## Historical Precedent

This follows the TypeScript trajectory:
- Early days: Community-maintained types in DefinitelyTyped (separate from code)
- Today: Types ship directly in packages
- Future: AI-optimized documentation ships with code, not maintained separately

## Call to Action

**For library authors:** Start shipping a `docs/` directory with your next release

**For AI tool vendors:** Auto-discover `docs/` directories in installed dependencies

**For package registries:** Recognize and promote packages with AI-optimized documentation

**For developers:** Request co-packaged documentation from the libraries you use

## Contributing

We welcome feedback, critiques, and adoption reports. This is a community RFC — help us refine the specification and build its future before formal standardization.

---

**Authors:** AI Documentation Working Group (Andres, Andrea)  
**Version:** 1.0  
**Date:** July 7, 2026
