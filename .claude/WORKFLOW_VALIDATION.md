# Workflow Validation Guide

Quick checks for agents to ensure proper workflow compliance.

## Planner Validation

Before requesting approval:
```bash
# Check plan file exists
ls .claude/plans/PLAN_*.md

# Verify plan has proper status
grep "Status: ⏸️ awaiting approval" .claude/plans/PLAN_*.md
```

## Programmer Validation

Before starting implementation:
```bash
# Check for approved status
grep "Status: 🛠 implementing" .claude/plans/PLAN_*.md

# Verify approval is documented
grep "Approved By:" .claude/plans/PLAN_*.md
```

## Reviewer Validation

During review:
```bash
# Run quality checks
pnpm lint
pnpm typecheck
pnpm test

# Check for console.logs
grep -r "console.log" apps/ packages/ --include="*.ts" --include="*.tsx"
```

## Tester Validation

Final checks:
```bash
# Full validation suite
pnpm test && pnpm lint && pnpm typecheck && pnpm build

# Verify all checklist items completed
grep -c "\[x\]" .claude/plans/PLAN_*.md
```

## Status Transitions

Valid status flow:
1. 🟡 planning (Planner creates)
2. ⏸️ awaiting approval (Planner requests)
3. 🛠 implementing (User approves)
4. 🔍 reviewing (Programmer completes)
5. ✅ done (Tester validates)

## Common Issues

- **Missing approval**: Programmer must wait for user confirmation
- **Incomplete checklist**: All items must be checked before review
- **Failed tests**: Fix before marking complete
- **Console.logs**: Remove before review
- **Type errors**: Resolve all `any` types