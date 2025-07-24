# ArcadeForge

A powerful drag-and-drop game creation platform that empowers anyone to build 2D games without programming. Create platformers, puzzle games, and arcade-style experiences similar to Super Mario Bros and Megaman directly in your browser.

## Overview

ArcadeForge is a comprehensive SaaS platform built for game creators of all skill levels. Our browser-based editor provides intuitive drag-and-drop tools, asset management, and real-time collaboration features. Games can be instantly previewed, tested, and published without any coding knowledge required.

### Key Features

- **🎮 Drag-and-Drop Editor**: Visual game creation with intuitive controls
- **🎨 Asset Management**: Upload, organize, and optimize game assets (sprites, sounds, backgrounds)
- **⚡ Real-time Preview**: Instantly test your games as you build them
- **👥 Collaboration**: Share projects and work together in real-time
- **🚀 One-Click Publishing**: Deploy games with built-in hosting and sharing
- **📱 Multi-Platform**: Games work on desktop, mobile, and tablets
- **☁️ Cloud Storage**: Projects automatically saved and synced across devices

## Architecture

ArcadeForge is built as a scalable TypeScript monorepo designed for high-performance game creation and delivery.

```
arcadeforge/
├── apps/
│   ├── game-editor/        # Main drag-and-drop game editor
│   ├── api-server/         # Backend API & game data management
│   └── game-runtime/       # Standalone game player application
├── packages/
│   ├── game-engine/        # Core 2D rendering engine (Pixi.js based)
│   ├── editor-ui/          # Reusable editor components & tools
│   ├── asset-processor/    # Image/audio optimization pipeline
│   ├── collaboration/      # Real-time editing & multiplayer features
│   └── shared-types/       # TypeScript definitions across all apps
├── infrastructure/         # AWS deployment configurations
│   ├── terraform/          # Infrastructure as Code
│   ├── docker/             # Container configurations
│   └── scripts/            # Deployment automation
└── docs/                   # Documentation & API specifications
```

### Core Applications

- **Game Editor**: React-based visual editor with drag-and-drop interface
- **API Server**: Fastify backend handling user management, game storage, and assets
- **Game Runtime**: Lightweight player for published games with optimized loading

## Getting Started

### Prerequisites

- **Node.js 18+** - JavaScript runtime
- **pnpm 8+** - Package manager (required)
- **Docker** - For local development environment
- **AWS CLI** - For deployment and asset management
- **PostgreSQL** - Database (or use AWS RDS for production)
- **Redis** - Caching and real-time features

### Local Development Setup

```bash
# Install dependencies
pnpm install

# Start all development servers
pnpm dev

# Available development commands
pnpm dev:editor     # Game editor only (http://localhost:3000)
pnpm dev:api        # API server only (http://localhost:8000)
pnpm dev:runtime    # Game runtime only (http://localhost:3001)

# Database setup
pnpm db:setup       # Initialize local database
pnpm db:migrate     # Run database migrations
pnpm db:seed        # Seed with sample data

# Testing
pnpm test           # Run all tests
pnpm test:e2e       # End-to-end game creation tests
pnpm test:engine    # Game engine performance tests

# Build for production
pnpm build          # Build all applications
pnpm build:editor   # Build game editor only
pnpm build:runtime  # Build game runtime only
```

### Environment Configuration

Create a `.env.local` file:

```bash
# Database
DATABASE_URL="postgresql://localhost:5432/arcadeforge"
REDIS_URL="redis://localhost:6379"

# AWS Configuration
AWS_REGION="us-east-1"
AWS_S3_BUCKET="arcadeforge-assets"
AWS_CLOUDFRONT_DOMAIN="cdn.arcadeforge.com"

# Authentication
JWT_SECRET="your-secret-key"
GOOGLE_OAUTH_CLIENT_ID="your-google-oauth-id"

# Development
NODE_ENV="development"
LOG_LEVEL="debug"
```

## Technology Stack

### Frontend (Game Editor)
- **React 19** - UI framework with concurrent features
- **Vite 6** - Fast build tool and development server
- **Redux Toolkit** - State management for editor and game data
- **Pixi.js** - High-performance 2D rendering engine
- **@dnd-kit** - Accessible drag-and-drop interactions
- **Radix UI** - Accessible component primitives
- **StyleX** - Optimized CSS-in-JS styling

### Backend (API Server)
- **Fastify 5** - High-performance web framework
- **Prisma 5** - Type-safe database ORM
- **Zod** - Runtime type validation
- **Socket.io** - Real-time collaboration
- **Bull** - Background job processing
- **Sharp** - Image processing and optimization

### Game Runtime
- **Pixi.js** - Optimized game rendering
- **Howler.js** - Web audio management
- **Custom Physics** - Lightweight 2D physics engine

### Infrastructure & DevOps
- **AWS EC2** - Scalable compute instances
- **AWS RDS** - PostgreSQL database hosting
- **AWS S3** - Asset storage and CDN
- **Redis** - Caching and session management
- **Docker** - Containerization
- **Terraform** - Infrastructure as Code

### Development Tools
- **TypeScript** - Type safety across the entire stack
- **Vitest** - Fast unit testing framework
- **Playwright** - End-to-end testing for game workflows
- **ESLint** - Code quality and consistency
- **Prettier** - Code formatting

## Development Workflow

ArcadeForge uses an AI-assisted development process with Claude Code for enhanced productivity:

### Planning & Implementation
1. **Plan**: Create detailed plans in `.claude/plans/` before implementing features
2. **Approve**: Get stakeholder approval for architectural changes
3. **Implement**: Follow hexagonal architecture patterns
4. **Review**: Automated code review with quality checks
5. **Test**: Comprehensive testing including game engine performance
6. **Deploy**: Automated deployment to AWS infrastructure

### Game Engine Development
- Focus on performance optimization for 60fps gameplay
- Implement comprehensive physics testing for game mechanics
- Maintain backward compatibility for published games
- Use feature flags for experimental game features

See [CLAUDE.md](./CLAUDE.md) for detailed development guidelines and AI assistant configuration.

## Deployment

### Production Infrastructure

ArcadeForge deploys to AWS with the following architecture:

- **Load Balancer**: ALB with SSL termination
- **API Servers**: Auto-scaling EC2 instances
- **Database**: RDS PostgreSQL with read replicas
- **Asset Storage**: S3 with CloudFront CDN
- **Real-time**: ElastiCache Redis cluster
- **Monitoring**: CloudWatch + custom game metrics

### Deployment Commands

```bash
# Deploy to staging
pnpm deploy:staging

# Deploy to production (requires approval)
pnpm deploy:production

# Database migrations
pnpm db:migrate:prod

# Monitor deployment
pnpm monitor:health
```

## API Documentation

- **Game Editor API**: `/docs/api/editor.md`
- **Asset Management**: `/docs/api/assets.md`
- **User Management**: `/docs/api/users.md`
- **Game Runtime**: `/docs/api/runtime.md`

## Contributing

We welcome contributions to ArcadeForge! Please follow these guidelines:

1. **Game Features**: Focus on user experience and performance
2. **Architecture**: Maintain hexagonal architecture principles
3. **Testing**: Include comprehensive tests for game engine changes
4. **Documentation**: Update relevant docs for any API changes
5. **Performance**: Ensure changes don't impact game rendering performance

### Pull Request Process

1. Create feature branch from `main`
2. Implement changes with tests
3. Update documentation
4. Run full test suite including game engine tests
5. Submit PR with clear description and demo video (for game features)

## License

Copyright © 2025 ArcadeForge. All rights reserved.

This software is proprietary and confidential. Unauthorized copying, modification, distribution, or use is strictly prohibited.
