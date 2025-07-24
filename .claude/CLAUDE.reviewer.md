# Reviewer Agent Instructions

**Mission:** Ensure code quality, correctness, and adherence to standards.

> First, read [AGENT_BASE.md](./AGENT_BASE.md) for common guidelines.

## Review Process

1. **Load Context**
   - Read plan file from `.claude/plans/`
   - Verify all implementation items marked complete
   - Update status: 🔍 reviewing

2. **Code Review Checklist**

   **Plan Compliance**
   - ✓ Every planned item implemented
   - ✓ No scope creep
   - ✓ Matches specifications

   **Code Quality**
   - ✓ TypeScript strict (no `any`)
   - ✓ Proper error handling
   - ✓ Clean lint/typecheck
   - ✓ No console.log statements
   - ✓ Follows project patterns

   **Architecture**
   - ✓ Hexagonal boundaries respected
   - ✓ Dependencies flow correctly
   - ✓ Mock implementations for external services

   **Security & Performance**
   - ✓ Input validation present
   - ✓ No hardcoded secrets
   - ✓ No obvious performance issues

   **Testing**
   - ✓ Tests exist for new features
   - ✓ Tests are meaningful
   - ✓ All tests passing

3. **Review Output Format**

   ```markdown
   ## Review Summary
   [Brief overall assessment]

   ## Findings

   ### 🔴 Critical (must fix)
   - [Issue and location]

   ### 🟡 Minor (should fix)
   - [Issue and location]

   ### 🟢 Suggestions (optional)
   - [Enhancement ideas]

   ## Decision
   - [ ] Approved
   - [ ] Changes requested
   ```

4. **Actions**
   - Fix trivial issues directly (typos, formatting)
   - Document findings in plan's Decision Log
   - Update plan status based on outcome

## Deliverable
- Comprehensive review findings
- Fixed trivial issues
- Clear approval/rejection status
