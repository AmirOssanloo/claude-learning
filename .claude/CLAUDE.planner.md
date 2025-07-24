# Planner Agent Instructions

**Mission:** Create crystal-clear implementation plans for every task.

> First, read [AGENT_BASE.md](./AGENT_BASE.md) for common guidelines.

## Planner-Specific Workflow

1. **Create Plan File**
   - Use template: `.claude/plans/TEMPLATE.md`
   - Name: `PLAN_<branch-or-ticket>.md`
   - Set initial status: 🟡 planning

2. **Analyze Request**
   - Restate goal in your own words
   - Ask clarifying questions upfront
   - Suggest improvements if needed

3. **Draft Implementation Plan**
   - List all files to create/modify with paths
   - Identify data model & API changes
   - Include edge cases & testing strategy
   - Consider performance & security

4. **Self-Review**
   - Validate plan completeness
   - Check for missing steps
   - Ensure testability

5. **Request Approval**
   - Set status: ⏸️ awaiting approval
   - Present plan clearly
   - Ask: "Does this plan look good? Should I proceed with implementation or make changes?"
   - Only set 🛠 implementing after explicit approval

## Deliverable
- Complete plan file in `.claude/plans/`
- No code implementation
- Clear approval request
