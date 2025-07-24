# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Claude Learning Sample** - A full-stack TypeScript monorepo for exploring Claude agent efficiency with minimal implementation. All features use mock implementations with solid architecture.

- **Frontend**: Minimal React components that demonstrate proper structure
- **Backend**: Service architecture with mock implementations that log actions
- **Purpose**: Test and refine Claude AI development workflows

## Key Commands

| Command          | Purpose                                      |
| ---------------- | -------------------------------------------- |
| `pnpm install`   | Install dependencies                         |
| `pnpm test`      | Run all tests                                |
| `pnpm lint`      | Run ESLint                                   |
| `pnpm typecheck` | Run TypeScript type checking                 |
| `pnpm dev`       | Start development servers                    |
| `pnpm build`     | Build for production                         |

**Package Manager**: Always use `pnpm` (never npm or yarn)

## Tech Stack

- **Frontend**: React 19, Vite 6, Redux Toolkit, Radix UI, StyleX
- **Backend**: Fastify 5, Prisma 5, Zod, PostgreSQL
- **Testing**: Vitest, Testing Library
- **Architecture**: Hexagonal (Ports & Adapters)

## Development Workflow

1. **Plan** → Create plan in `.claude/plans/`
2. **Approve** → Get user approval
3. **Implement** → Follow architecture
4. **Review** → Validate code quality
5. **Test** → Ensure all tests pass
6. **Commit** → Use conventional commits

## Project Structure

```
/
├── apps/
│   ├── frontend/     # React application
│   └── backend/      # Node.js service
├── packages/         # Shared packages
│   ├── types/        # TypeScript types
│   └── ui/           # UI components
├── .claude/          # AI assistant config
│   ├── plans/        # Task planning
│   ├── docs/         # Domain docs
│   └── *.md          # Agent configs
├── CLAUDE.md         # This file
├── README.md         # Project docs
└── CHANGELOG.md      # Version history
```

## Claude Configuration

For detailed guidelines and agent behavior, see:
- `.claude/CLAUDE.md` - Architecture & detailed rules
- `.claude/plans/TEMPLATE.md` - Plan template
- Agent-specific configs in `.claude/`
