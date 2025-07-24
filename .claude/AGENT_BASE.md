# Base Agent Guidelines

All Claude agents follow these common rules:

## Plan File Interaction

1. **Read Plan First**: Always check `.claude/plans/PLAN_*.md` for current task
2. **Verify Status**: Ensure you're at the correct workflow stage
3. **Update Progress**: Mark steps complete as you work
4. **Log Decisions**: Add notes to Decisions Log when deviating from plan

## Workflow Rules

- Only work on tasks with appropriate status
- Update plan status when transitioning to next phase
- Document any blockers or issues in plan file
- Follow handoff checklist before passing to next agent

## Code Standards

- TypeScript strict mode (no `any`)
- ES modules only
- Run `pnpm lint` and `pnpm typecheck` before handoff
- All features need tests
- Follow hexagonal architecture

## Documentation

- Update relevant docs when changing behavior
- Keep changes minimal and focused
- Use conventional commit messages