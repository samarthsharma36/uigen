# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev          # Start dev server with Turbopack on localhost:3000
npm run build        # Build production bundle
npm run lint         # Run ESLint
npm run test         # Run Vitest unit tests
npm run setup        # Install deps + generate Prisma client + run migrations
npm run db:reset     # Reset SQLite database (destructive)
```

Run a single test file:
```bash
npx vitest run src/components/chat/__tests__/ChatInterface.test.tsx
```

## Architecture

UIGen is a 3-panel AI-powered React component generator:
- **Left panel**: Chat interface for natural language UI generation
- **Right panel**: Toggle between live component preview (iframe) and Monaco code editor with file tree

### AI Integration

The core loop lives in [src/app/api/chat/route.ts](src/app/api/chat/route.ts): user messages → Claude (via Vercel AI SDK + `@ai-sdk/anthropic`) → streaming response with tool calls → file system updates.

Claude uses two tools to modify files:
- `str_replace_editor` ([src/lib/tools/str-replace.ts](src/lib/tools/str-replace.ts)): patch specific lines in files
- `file_manager` ([src/lib/tools/file-manager.ts](src/lib/tools/file-manager.ts)): rename/delete files

The system prompt controlling Claude's behavior is in [src/lib/prompts/generation.tsx](src/lib/prompts/generation.tsx).

### Virtual File System

All component files live in-memory — no disk I/O. The virtual FS ([src/lib/file-system.ts](src/lib/file-system.ts)) serializes to JSON for database persistence. React state is managed via `FileSystemProvider` context ([src/lib/contexts/file-system-context.tsx](src/lib/contexts/file-system-context.tsx)).

### State Management

Two React contexts carry all app state:
- `ChatProvider` ([src/lib/contexts/chat-context.tsx](src/lib/contexts/chat-context.tsx)): messages, streaming, AI SDK hooks
- `FileSystemProvider`: virtual file system + selected file

### Authentication

JWT-based with HTTP-only cookies (7-day expiry). Anonymous users can use the app without signing in. Middleware at [src/middleware.ts](src/middleware.ts) handles route protection. Server-side logic in [src/lib/auth.ts](src/lib/auth.ts).

### Database

Prisma + SQLite. Reference [prisma/schema.prisma](prisma/schema.prisma) whenever you need to understand the data structure — two models: `User` and `Project` (stores messages + virtual FS as JSON strings). Server actions in [src/actions/](src/actions/) handle all DB operations from the frontend.

### Code Style

Use comments sparingly — only for complex logic that isn't self-evident.

### Path Aliases

`@/*` maps to `./src/*` (configured in `tsconfig.json`).

### Environment

Requires `OPENAI_API_KEY` in `.env`. Falls back to a mock provider if missing. Model is `gpt-4o` (set in `src/lib/provider.ts`).
