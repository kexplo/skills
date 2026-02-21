---
name: research
description: "Research a folder or repository in depth, identify architecture, behavior, and domain-specific details, and produce a comprehensive `research.md` report."
---

# Folder Research

## Goal

Help deeply analyze a target folder or repository by reading structure, source code, configs, and docs to produce a complete findings report.

## Read and Analyze a Folder

When invoked, perform a structured review in this order:

1. Confirm the target folder (default to the current workspace if not provided).
2. Map the top-level layout (apps, packages, scripts, docs, configs, tests, runtime files).
3. Read representative files from each major area (entry points, core modules, shared utilities, config, and docs).
4. Extract:
   - overall purpose and user-facing behavior
   - architecture and component boundaries
   - key data flow and dependencies
   - integration points and external interfaces
   - implementation patterns, constraints, and notable design choices
   - risks, gaps, and follow-up questions
5. Continue scanning deeper in directories that appear critical even when names are ambiguous.
6. Write the findings to `research.md` in the target folder.

## report Requirements (`research.md`)

- Use this fixed structure:
  - `Overview`
  - `What this folder does`
  - `Architecture and structure`
  - `Important files and behavior`
  - `Specificities and edge details`
  - `Risks and assumptions`
  - `Recommended next steps`
- Include file path references for all major findings.
- Include unresolved questions where evidence is missing.
- Keep findings evidence-based and avoid assumptions without note.

## Scope and Conduct

- Prefer reading core implementation files before peripheral files.
- Read enough to gain deep understanding, not a shallow file list.
- Do not use web sources unless explicitly requested.
