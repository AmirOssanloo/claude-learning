# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**ArcadeForge** - A scalable drag-and-drop game creation SaaS platform built with TypeScript. Empowers users to create 2D games (platformers, puzzle games, arcade-style) without programming knowledge.

- **Game Editor**: React-based visual editor with drag-and-drop interface using Pixi.js
- **API Server**: Fastify backend managing users, games, assets, and real-time collaboration
- **Game Runtime**: Optimized player for published games with multi-platform support
- **Purpose**: Production SaaS platform for accessible game development

## Key Commands

| Command              | Purpose                                      |
| -------------------- | -------------------------------------------- |
| `pnpm install`       | Install dependencies                         |
| `pnpm dev`           | Start all development servers                |
| `pnpm dev:editor`    | Game editor only (port 3000)                |
| `pnpm dev:api`       | API server only (port 8000)                 |
| `pnpm dev:runtime`   | Game runtime only (port 3001)               |
| `pnpm test`          | Run all tests                                |
| `pnpm test:e2e`      | End-to-end game creation tests               |
| `pnpm test:engine`   | Game engine performance tests                |
| `pnpm lint`          | Run ESLint                                   |
| `pnpm typecheck`     | Run TypeScript type checking                 |
| `pnpm build`         | Build for production                         |
| `pnpm build:editor`  | Build game editor only                       |
| `pnpm build:runtime` | Build game runtime only                      |
| `pnpm db:setup`      | Initialize local database                    |
| `pnpm db:migrate`    | Run database migrations                      |
| `pnpm db:seed`       | Seed database with sample games             |

**Package Manager**: Always use `pnpm` (never npm or yarn)  
**Performance**: Game engine tests must maintain 60fps for approval

## Tech Stack

### Game Editor (Frontend)
- **React 19**: UI framework with concurrent features for smooth editor performance
- **Vite 6**: Fast build tool optimized for game asset bundling
- **Redux Toolkit**: State management for game projects, assets, and editor state
- **Pixi.js**: High-performance 2D rendering engine for game preview and editing
- **@dnd-kit**: Accessible drag-and-drop for game object placement
- **Radix UI + StyleX**: Accessible components with optimized styling

### API Server (Backend)
- **Fastify 5**: High-performance web framework for game data APIs
- **Prisma 5**: Type-safe ORM for game projects, users, and asset metadata
- **Zod**: Runtime validation for game configurations and user input
- **Socket.io**: Real-time collaboration for multi-user game editing
- **Bull**: Background job processing for asset optimization and game builds
- **Sharp**: Image processing for sprite optimization and thumbnail generation

### Game Runtime Engine
- **Pixi.js**: Optimized 2D rendering with WebGL acceleration
- **Howler.js**: Cross-platform audio management for game sounds
- **Custom Physics**: Lightweight 2D physics engine for platformer mechanics

### Infrastructure
- **AWS EC2**: Auto-scaling compute for API servers
- **AWS RDS**: PostgreSQL with read replicas for game data
- **AWS S3**: Asset storage with CloudFront CDN for fast game loading
- **Redis**: Caching and real-time collaboration state management

### Development & Testing
- **TypeScript**: Strict typing across entire game development stack
- **Vitest**: Fast unit testing with game engine performance benchmarks
- **Playwright**: End-to-end testing for complete game creation workflows
- **Docker**: Containerized development environment

**Architecture**: Hexagonal (Ports & Adapters) with game-specific domain separation

## Development Workflow

1. **Plan** → Create plan in `.claude/plans/`
2. **Approve** → Get user approval
3. **Implement** → Follow architecture
4. **Review** → Validate code quality
5. **Test** → Ensure all tests pass
6. **Commit** → Use conventional commits

## Project Structure

```
arcadeforge/
├── apps/
│   ├── game-editor/        # Main drag-and-drop game editor (React)
│   │   ├── src/
│   │   │   ├── editor/     # Canvas-based game editor components
│   │   │   ├── assets/     # Asset management & upload UI
│   │   │   ├── preview/    # Game preview & testing interface
│   │   │   ├── dashboard/  # User projects & collaboration
│   │   │   └── auth/       # Authentication flows
│   ├── api-server/         # Backend API & game data management
│   │   ├── src/
│   │   │   ├── modules/
│   │   │   │   ├── games/    # Game CRUD & project management
│   │   │   │   ├── assets/   # Asset storage & processing
│   │   │   │   ├── users/    # User management & billing
│   │   │   │   └── collab/   # Real-time collaboration
│   │   │   └── infrastructure/
│   │   │       ├── s3/       # AWS S3 asset integration
│   │   │       ├── db/       # Database connections
│   │   │       └── redis/    # Caching & sessions
│   └── game-runtime/       # Standalone game player
├── packages/
│   ├── game-engine/        # Core 2D rendering & physics (Pixi.js)
│   ├── editor-ui/          # Reusable drag-and-drop components
│   ├── asset-processor/    # Image/audio optimization pipeline
│   ├── collaboration/      # Real-time editing (Socket.io)
│   └── shared-types/       # TypeScript definitions
├── infrastructure/         # AWS deployment & Infrastructure as Code
│   ├── terraform/          # AWS resource definitions
│   ├── docker/             # Container configurations
│   └── scripts/            # Deployment automation
├── .claude/                # AI assistant configuration
│   ├── plans/              # Task planning for game development
│   ├── docs/               # Domain-specific documentation
│   └── *.md                # Agent configurations
└── docs/                   # API documentation & architecture guides
```

## Game Development Considerations

### Performance Requirements
- **60fps Target**: All game engine changes must maintain 60fps performance
- **Asset Optimization**: Images optimized for web delivery, sprites atlased efficiently
- **Memory Management**: Proper cleanup of game objects and textures
- **Loading Performance**: Progressive loading for large games and assets

### User Experience Priorities
- **No-Code Approach**: All game creation through visual drag-and-drop interface
- **Real-time Collaboration**: Multiple users editing games simultaneously
- **Instant Preview**: Changes reflected immediately in game preview
- **Asset Pipeline**: Seamless upload, processing, and integration of user assets

### Development Domains
- **Game Engine**: Core rendering, physics, input handling
- **Editor Interface**: Drag-and-drop tools, property panels, timeline
- **Asset Management**: Upload, storage, optimization, organization
- **Collaboration**: Real-time synchronization, conflict resolution
- **Publishing**: Game export, hosting, sharing mechanisms

## Claude Configuration

For detailed guidelines and agent behavior, see:
- `.claude/CLAUDE.md` - Architecture & detailed rules for game development
- `.claude/plans/TEMPLATE.md` - Plan template for game features
- `.claude/docs/game-editor.md` - Editor architecture patterns
- `.claude/docs/game-engine.md` - Engine development guidelines
- `.claude/docs/asset-pipeline.md` - Asset processing workflows
- Agent-specific configs in `.claude/` for game development tasks
