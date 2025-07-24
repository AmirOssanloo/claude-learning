# Base Agent Guidelines for Game Development

All Claude agents follow these common rules for ArcadeForge game development:

## Plan File Interaction

1. **Read Plan First**: Always check `.claude/plans/PLAN_*.md` for current task
2. **Verify Status**: Ensure you're at the correct workflow stage
3. **Update Progress**: Mark steps complete as you work
4. **Log Decisions**: Add notes to Decisions Log when deviating from plan
5. **Performance Context**: Consider 60fps requirements in all game-related tasks

## Game Development Workflow Rules

- Only work on tasks with appropriate status
- Update plan status when transitioning to next phase
- Document any blockers or performance issues in plan file
- Follow handoff checklist before passing to next agent
- **Performance Validation**: Test frame rate impact for game engine changes
- **Asset Awareness**: Consider asset loading and processing implications

## Code Standards for Games

- **TypeScript Strict**: No `any` types, comprehensive game data typing
- **ES Modules**: Consistent module system throughout
- **Performance First**: 60fps target for all game engine code
- **Memory Management**: Proper cleanup of game objects and textures
- **Real-time Ready**: Design for multi-user collaboration
- **Cross-Platform**: Test on desktop, mobile, and tablet
- Run `pnpm lint`, `pnpm typecheck`, and performance tests before handoff
- All game features need automated and manual testing
- Follow hexagonal architecture with game domain separation

## Game-Specific Documentation

- Update game architecture docs when changing engine behavior
- Document asset pipeline changes in asset-pipeline.md
- Update API.md for game data or collaboration endpoint changes
- Keep performance benchmarks and optimization notes
- Use conventional commit messages with game context

## Domain Knowledge Areas

When working on ArcadeForge, agents should understand:
- **Game Editor**: Drag-and-drop interface, real-time preview, collaboration
- **Game Engine**: 2D rendering, physics, input handling, performance optimization
- **Asset Pipeline**: Upload, processing, optimization, CDN delivery
- **Real-time Collaboration**: WebSocket communication, operational transforms
- **SaaS Infrastructure**: Multi-tenant architecture, billing, scaling