# Game Development Workflow Validation Guide

Quick checks for agents to ensure proper game development workflow compliance.

## Planner Validation (Game Features)

Before requesting approval:
```bash
# Check plan file exists
ls .claude/plans/PLAN_*.md

# Verify plan has proper status
grep "Status: ⏸️ awaiting approval" .claude/plans/PLAN_*.md

# Validate game-specific planning elements
grep -E "(60fps|performance|asset|collaboration)" .claude/plans/PLAN_*.md
```

**Game Development Checklist:**
- [ ] Performance impact assessed (60fps target)
- [ ] Asset requirements identified
- [ ] Real-time collaboration considerations
- [ ] Cross-platform compatibility planned
- [ ] Memory management strategy defined

## Programmer Validation (Game Implementation)

Before starting implementation:
```bash
# Check for approved status
grep "Status: 🛠 implementing" .claude/plans/PLAN_*.md

# Verify approval is documented
grep "Approved By:" .claude/plans/PLAN_*.md

# Game engine performance baseline
pnpm test:engine
```

**Game Development Checks:**
- [ ] Game engine performance tests passing
- [ ] Asset loading strategy implemented
- [ ] Real-time features tested with multiple users
- [ ] Memory cleanup implemented for game objects
- [ ] Cross-platform compatibility verified

## Reviewer Validation (Game Code Review)

During review:
```bash
# Run quality checks
pnpm lint
pnpm typecheck
pnpm test
pnpm test:e2e

# Game-specific validations
# Check for console.logs (not allowed in production)
grep -r "console.log" apps/ packages/ --include="*.ts" --include="*.tsx"

# Verify performance optimizations
grep -r "performance.now\|requestAnimationFrame" apps/game-editor/ apps/game-runtime/

# Check for proper memory cleanup
grep -r "destroy\|cleanup\|removeEventListener" packages/game-engine/

# Validate asset optimization
find assets/ -type f -size +1M -exec echo "Large asset: {}" \;
```

**Game Review Checklist:**
- [ ] 60fps performance maintained
- [ ] No memory leaks in game objects
- [ ] Assets properly optimized
- [ ] Real-time collaboration working
- [ ] Cross-platform tested
- [ ] Security validated for user content
- [ ] Accessibility features implemented

## Tester Validation (Game Testing)

Final checks:
```bash
# Full validation suite including game tests
pnpm test && pnpm test:engine && pnpm test:e2e && pnpm lint && pnpm typecheck && pnpm build

# Performance validation
pnpm test:performance

# Game-specific tests
pnpm test:collaboration  # Multi-user testing
pnpm test:assets        # Asset pipeline testing
pnpm test:mobile        # Mobile device testing

# Verify all checklist items completed
grep -c "\[x\]" .claude/plans/PLAN_*.md
```

**Game Testing Requirements:**
- [ ] 60fps maintained during gameplay
- [ ] Asset loading under 2 seconds
- [ ] Real-time collaboration latency <100ms
- [ ] Cross-platform compatibility verified
- [ ] Memory usage within acceptable limits
- [ ] Game creation workflow end-to-end tested
- [ ] Publishing pipeline validated

## Game Development Status Transitions

Valid status flow with game-specific gates:
1. 🟡 **planning** (Planner creates with performance considerations)
2. ⏸️ **awaiting approval** (Planner requests with game impact assessment)
3. 🛠 **implementing** (User approves after reviewing game requirements)
4. 🔍 **reviewing** (Programmer completes with performance validation)
5. ✅ **done** (Tester validates including game-specific tests)

## Game Development Common Issues

### Performance Issues
- **Frame rate drops**: Check for expensive operations in game loop
- **Memory leaks**: Verify proper cleanup of Pixi.js objects
- **Asset loading delays**: Implement progressive loading strategies
- **Real-time lag**: Optimize WebSocket message frequency

### Collaboration Issues
- **Conflict resolution**: Test operational transforms with simultaneous edits
- **State synchronization**: Verify game state consistency across clients
- **User presence**: Ensure cursor and selection updates work properly

### Asset Pipeline Issues
- **Large file sizes**: Validate optimization pipeline is working
- **Format compatibility**: Ensure WebP/AVIF fallbacks exist
- **CDN caching**: Verify cache invalidation after asset updates

### Code Quality Issues
- **Console.logs**: Remove all debug logging before production
- **Type errors**: No `any` types in game data structures
- **Missing tests**: All game engine functions need unit tests
- **Security gaps**: Validate all user-generated content

## Performance Benchmarks

Maintain these targets throughout development:
- **Editor Frame Rate**: 60fps consistent
- **Game Runtime**: 60fps with 100+ objects
- **Asset Loading**: <2s for typical game scene
- **Collaboration Latency**: <100ms for real-time updates
- **Memory Usage**: <500MB for complex games
- **Build Time**: <30s for incremental builds