# Repository Guidance for Coding Agents

This is a **public, intentionally short** entry point. It applies to Codex, Claude Code, and other coding agents working in this repository.

## Current phase

- The repository currently contains **documentation-only foundation files**.
- Its primary product is **framework-neutral agent system-design guidance**.
- Do not claim there is an executable runtime, tested notebook, code generator, or deployed product.
- LangChain / LangGraph is the first *planned* example ecosystem; framework-free designs remain valid.

## Before changing anything

1. Read `README.md`, especially scope, project status, output contract, and engineering principles.
2. **If present locally**, read `local note/CODEX_HANDOFF.md` and any specifically linked research notes. `local note/` contains private collaboration history and is Git-ignored; it is intentionally absent from public clones.
3. If the local note is absent, use public files and ask the project owner for missing decisions rather than inventing their contents.
4. Separate agreed project direction from research drafts, unverified assumptions, and planned features.

## Repository boundaries

- Put polished, reviewed, reusable material intended for readers into public documents.
- Put discussion logs, working thoughts, Codex handoffs, experiments and private planning into **`local note/`**, not `docs/`.
- Never remove or override `.gitignore` rules that protect local notes or secrets.
- Never publish `local note/` without explicit owner review; do not force-add ignored paths.
- Do not expand into a custom agent framework, UI platform, model gateway, or full SDK adapter collection without approval.
- Documentation, diagrams, code, and test claims must agree. Mark examples and future capabilities explicitly.

## Expected change process

Requirements -> architecture alternatives -> design decision -> editable Mermaid diagrams -> security/reliability constraints -> proposed tests -> approval -> implementation -> verification.

Prefer small, reviewable changes. Report what was actually modified and tested, and what remains unverified. Do not push to GitHub without explicit authorization.
