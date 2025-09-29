# Documentation Index

Use this guide to file new documents consistently. All paths are relative to the repository root.

## Core Folders
- `docs/guides/` - How-to material and integration walkthroughs (e.g., Identity Base, FormFeeder, Tailwind setup).
- `docs/specs/` - Architectural specs, feature specifications, ADRs, and RFCs.
- `docs/planning/` - Roadmaps, milestone plans, OKRs, and backlog shaping notes.
- `docs/sprints/` - Iteration summaries, sprint goals, and retrospectives.
- `docs/runbooks/` - Operational checklists, incident playbooks, and on-call procedures.
- `docs/audits/` - Security reviews, compliance evidence, post-audit action logs.
- `docs/reference/` - Durable reference material (engineering principles, coding standards, database design).
- `docs/assets/` - Static assets used by other docs (diagrams, images, logo files).

## Naming & Format
- Use PascalCase for directory names and Title Case for Markdown file names (e.g., `docs/specs/IdentityProvisioning.md`).
- Keep filenames ASCII-only; avoid spaces in folder names, use hyphenated or PascalCase file names.
- Time-bound documents (sprints, plans) should use `YYYY-MM` prefixes for easy sorting, e.g., `docs/sprints/2025-01-Sprint-Review.md`.

## Filing Checklist
- Confirm a document does not already exist before creating another version.
- Link related material at the top of the file (e.g., prior specs, relevant guides).
- Update indices or navigation links when you add a new doc (e.g., amend this README or team wiki as needed).

Following this structure keeps documentation discoverable and aligned with the repo's backend (`backend/`) and frontend (`apps/`) conventions.
