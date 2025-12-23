# Hyperfactory Agent Rules

## Project Structure
This is a **spec/documentation project** (not code). Key files: `spec.md` (summary), `/spec` folder (detailed spec files for each domain).

## Build, Test, Lint
No code to build/test. Use markdown lint for `.md` files via VS Code.

## Architecture & Domains
Hyperfactory has two main products: (1) hyperfactory core (factory data platform with EdgeX, Odoo integration, OpenZiti security), (2) hyperfactory-cloud (managed IaaS). See `/spec/architecture-infrastructure.md` for details.

## Code Style
* Markdown: kebab-case filenames, enforce markdown lint
* Spec format: `decided` section (finalized decisions), `undecided` section (open questions)
* Documentation: Update existing files instead of creating duplicates; organize into subfolders as needed
* Context folder: reference only—do not edit/add/delete files there

## Key Rules
* **spec.md is source of truth**. Document all spec changes there and in `/spec/` detail files
* Keep docs folder up-to-date with every change
* Use Linux commands (WSL environment)

