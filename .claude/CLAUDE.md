# Claude Learning – Detailed Guidelines

> **Project**: Claude Learning Sample - TypeScript monorepo for testing Claude AI workflows  
> **Package Manager**: pnpm (exclusively)

## Documentation Structure

| Document | Purpose |
| -------- | ------- |
| `/CLAUDE.md` | Quick reference & overview |
| `/.claude/CLAUDE.md` | This file - detailed rules |
| `/.claude/AGENT_BASE.md` | Common agent guidelines |
| `/.claude/plans/TEMPLATE.md` | Plan template |
| `/.claude/WORKFLOW_VALIDATION.md` | Validation checks |
| `/.claude/docs/frontend.md` | Frontend architecture |
| `/.claude/docs/backend.md` | Backend architecture |
| Agent files (`*.planner.md`, etc.) | Role-specific behavior |

## Global Coding Rules

1. **TypeScript only** – strict mode; avoid `any`.
2. **ES modules** everywhere (`import/export`).
3. Follow lint + Prettier. No stray `console.log`.
4. Respect hexagonal layering: `api → services → core → ports ← infrastructure`.
5. **Tests are mandatory** for every feature or bug-fix.
6. **Security:** validate all inputs; never hard-code secrets.
7. **Performance:** avoid N+1 queries, unnecessary re-renders, CPU-heavy loops.
8. **No unauthorized refactors** outside task scope.
9. New deps require justification & must be added via `pnpm`.
10. Commit flow: **Plan → Approve → Implement → Review → Test → Commit**.

See the domain docs (`frontend.md`, `backend.md`, `packages.md`) and agent docs in `.claude/` for details.

---

## Self-Updating Documentation

> **Goal:** Claude must keep project docs accurate by editing the relevant Markdown files whenever a change alters behavior, structure, or workflow.

### When Claude MUST update docs

- **Feature additions / changes** → user-facing docs (README, API spec, UI guides).
- **Architecture / folder layout updates** → corresponding domain file (`docs/frontend.md`, `docs/backend.md`).
- **New scripts or commands** → _Commands_ table in **CLAUDE.md**.
- **Process or rule tweaks** → update the affected agent file in `.claude/`.
- **Deprecations / removals** → delete or mark obsolete sections to prevent drift.

### Update workflow

1. **Identify scope**  
   – Pinpoint the _exact_ file(s) and section(s) affected.  
   – Touch only what is necessary; avoid blanket rewrites.

2. **Edit surgically**  
   – Preserve heading hierarchy and style.  
   – Insert or modify concise bullets or sentences; no verbose prose.  
   – Keep token footprint low—brevity matters for future context windows.

3. **Log the change**  
   – Append a one-line entry to `CHANGELOG.md` if user-visible.  
   – Optionally add a 1-2 sentence note under today’s date in `JOURNAL.md`.

4. **Commit etiquette**  
   – Prefer same-PR commits for tightly-coupled code + doc changes; separate PRs if docs are substantial.  
   – Conventional commit prefix:
   ```
   docs: ✏️ update <file> for <reason>
   ```

### Guardrails

- **Never leak secrets**—no credentials or private URLs in docs.
- **Doc edits follow QA**—Reviewer agent must verify accuracy; Tester ensures build + lint still pass.
- **Delete stale content**—remove outdated instructions rather than leaving contradictory notes.

By following this checklist, Claude guarantees living documentation that remains in lock-step with the evolving codebase.

---

## Shared Plan & Workflow Document

> **Purpose:** Give every agent a single source-of-truth for the current task—plan, progress, and hand-offs.

### File location & naming

.claude/plans/PLAN\_<branch-or-ticket>.md

_Example:_ `.claude/plans/PLAN_feature/123-user-invite.md`  
Each feature/bug-fix gets exactly **one** plan file, created by the Planner.

### Markdown layout (template)

```md
# Plan: <brief title>

## Meta

- Ticket / Branch: <id-or-name>
- Created: YYYY-MM-DD HH:MM (UTC)
- Planner: Claude
- Status: 🟡 planning | ⏸️ awaiting approval | 🛠 implementing | 🔍 reviewing | ✅ done

## Goals

- Bullet list of acceptance criteria.

## Checklist

- [ ] Step 1 – …
- [ ] Step 2 – …
- [ ] …

## Decisions / Notes

- YYYY-MM-DD – Rationale for choice X over Y.
```

### Agent responsibilities

| Role            | Action on this file                                                                                                                                           |
| --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Planner**     | Create file; fill **Goals** & **Checklist**.<br>Set **Status:** 🟡 _planning_ → ⏸️ _awaiting approval_.<br>Get user approval before setting 🛠 _implementing_. |
| **Implementer** | Verify status is 🛠 _implementing_ (not ⏸️ _awaiting approval_).<br>Tick boxes as tasks complete.<br>Append brief notes if deviating.                          |
| **Reviewer**    | Verify all boxes are ticked; add comments under **Decisions / Notes**.<br>Set **Status:** 🔍 _reviewing_.                                                     |
| **Tester**      | After tests pass, change **Status** to ✅ _done_ and add a final confirmation note.                                                                           |

---

### Editing rules

1. **Atomic updates** – edit only your section; avoid rewriting others’ notes.
2. **Timestamp** decisions in `YYYY-MM-DD` format.
3. **No code blobs** – link to files/lines instead.
4. **Keep it concise** – delete obsolete notes rather than piling up noise.
5. When **Status** becomes ✅ _done_, archive or move the file to `.claude/plans/_history/`.

_This shared plan file ensures seamless hand-offs and full transparency between agents._
