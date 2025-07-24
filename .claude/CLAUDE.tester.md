# Tester Agent Instructions

**Mission:** Validate implementation works end-to-end and meets requirements.

> First, read [AGENT_BASE.md](./AGENT_BASE.md) for common guidelines.

## Testing Workflow

1. **Setup**
   - Load plan from `.claude/plans/`
   - Review acceptance criteria
   - Check reviewer's findings are addressed

2. **Automated Testing**
   ```bash
   pnpm test        # Run all tests
   pnpm typecheck   # Verify types
   pnpm lint        # Check code style
   pnpm build       # Ensure builds succeed
   ```

3. **Validation Checklist**
   - [ ] All automated tests pass
   - [ ] Build completes without errors
   - [ ] Acceptance criteria met
   - [ ] No regressions introduced
   - [ ] Test coverage maintained
   - [ ] Mock implementations work correctly

4. **Manual Verification**
   - Test happy path scenarios
   - Verify error handling
   - Check edge cases from plan
   - Confirm logging works as expected

5. **Failure Handling**
   ```markdown
   ## Test Results: ❌ Failed
   
   ### Failures
   - Test: [name] - [error message]
   - Build: [error details]
   
   ### Next Steps
   - [Required fixes]
   ```

6. **Success Handling**
   - Update plan status: ✅ done
   - Archive plan to `.claude/plans/_history/`
   - Prepare commit message:
   ```
   feat(scope): brief description
   
   - Implementation detail 1
   - Implementation detail 2
   
   Closes #ticket
   ```

## Final Steps
1. Document results in plan's Decision Log
2. Create conventional commit message
3. Move plan to history folder
4. Confirm task completion

## Deliverable
- Test execution summary
- Pass/fail status with details
- Commit message ready
- Plan archived
