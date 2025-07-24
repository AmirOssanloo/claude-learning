# Programmer Agent Instructions

**Mission:** Transform approved plans into working code.

> First, read [AGENT_BASE.md](./AGENT_BASE.md) for common guidelines.

## Pre-Implementation Checklist

1. **Locate Plan**: Find `PLAN_*.md` in `.claude/plans/`
2. **Verify Status**: Must be 🛠 implementing
3. **Check Approval**: Confirm explicit user approval exists
   - If not approved, ask: "I see the plan is ready. Has this been approved for implementation?"

## Implementation Workflow

1. **Setup**
   - Load plan file and track progress
   - Review architecture docs if needed
   - Set up any new directories

2. **Code Step-by-Step**
   - Implement one checklist item at a time
   - Mark items complete as you finish
   - Write tests alongside features
   - Run `pnpm lint` and `pnpm typecheck` after each step

3. **Quality Checks**
   - No `any` types
   - Defensive error handling
   - Follow existing patterns
   - Mock external services (S3, APIs, etc.)

4. **Documentation**
   - Update docs if behavior changes
   - Add inline comments only if complex
   - Log decisions in plan file

## Handoff Preparation
- All plan items checked off
- Tests written and passing
- Lint/typecheck clean
- Ready for code review

## Deliverable
- Working code matching plan specifications
- Tests for all new functionality
- Clean lint and type checks
