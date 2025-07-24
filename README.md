# Claude Learning Sample

A TypeScript monorepo designed to explore and optimize Claude AI agent workflows with minimal implementation complexity.

## Overview

This project serves as a testing ground for Claude AI development patterns. It uses mock implementations to maintain clean architecture while keeping business logic minimal.

## Project Structure

```
claude-learning/
├── apps/                    # Applications
│   ├── frontend/           # React application
│   └── backend/            # Node.js service
├── packages/               # Shared packages
│   ├── types/             # TypeScript definitions
│   └── ui/                # UI component library
├── .claude/                # Claude AI configuration
│   ├── plans/             # Task planning documents
│   ├── docs/              # Architecture documentation
│   └── *.md               # Agent configurations
└── docs/                   # Project documentation
```

## Getting Started

### Prerequisites

- Node.js 18+
- pnpm 8+ (required - do not use npm or yarn)

### Installation

```bash
pnpm install
```

### Development

```bash
pnpm dev        # Start development servers
pnpm test       # Run tests
pnpm lint       # Run linter
pnpm typecheck  # Check TypeScript types
pnpm build      # Build for production
```

## Claude AI Integration

This repository includes comprehensive Claude AI assistant configuration:

- **Agent System**: Specialized agents for planning, programming, reviewing, and testing
- **Plan Management**: Structured task planning with approval workflows
- **Architecture Guidelines**: Hexagonal architecture with clear separation of concerns
- **Documentation**: Self-updating documentation system

See [CLAUDE.md](./CLAUDE.md) for Claude-specific guidelines.

## Tech Stack

- **Frontend**: React 19, Vite 6, Redux Toolkit, Radix UI, StyleX
- **Backend**: Fastify 5, Prisma 5, Zod, PostgreSQL
- **Testing**: Vitest, Testing Library
- **Architecture**: Hexagonal (Ports & Adapters)

## Contributing

1. Create a plan in `.claude/plans/` before implementing features
2. Get plan approval before starting implementation
3. Follow the established architecture patterns
4. Ensure all tests pass before committing
5. Use conventional commits

## License

This is a sample project for testing Claude AI workflows.
