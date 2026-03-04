# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# First-time setup (install deps, generate Prisma client, run migrations)
npm run setup

# Development server (Turbopack)
npm run dev

# Production build
npm run build

# Lint
npm run lint

# Run tests
npm test

# Run a single test file
npx vitest run src/path/to/file.test.ts

# Reset database
npm run db:reset
```

Add `ANTHROPIC_API_KEY=your-key` to `.env` to use Claude for generation. Without it, a mock provider returns static code.

## Architecture

UIGen is a Next.js 15 App Router application where users describe React components in a chat interface and receive AI-generated code with live preview.

### Data flow

1. User sends a chat message → `ChatContext` forwards to `/api/chat`
2. `/api/chat` streams a response from Claude (Vercel AI SDK) with tool calls
3. Tool calls invoke `str_replace_editor` (create/edit files) or `file_manager` (rename/delete)
4. `FileSystemContext` updates an in-memory virtual file system — nothing is written to disk
5. The preview iframe re-renders: `jsx-transformer.ts` runs Babel standalone to transpile JSX/TS and resolves imports via `esm.sh` CDN into blob URLs
6. For authenticated users, file system state is serialized and persisted to SQLite via Prisma

### Key directories

- `src/app/api/chat/` — streaming AI endpoint; tool definitions and execution live here
- `src/lib/contexts/` — `ChatContext` (AI state, streaming) and `FileSystemContext` (virtual FS)
- `src/lib/tools/` — AI tool schemas (`str_replace_editor`, `file_manager`)
- `src/lib/transform/` — Babel-based JSX→ESM transformer for in-browser execution
- `src/lib/prompts/` — system prompts sent to Claude
- `src/lib/provider.ts` — returns either the Anthropic Claude provider or a `MockLanguageModel`
- `src/actions/` — server actions for auth (JWT + bcrypt) and project CRUD
- `src/components/` — React components: `chat/`, `editor/` (Monaco), `preview/`, `auth/`, `ui/` (shadcn)
- `prisma/schema.prisma` — `User` and `Project` models; SQLite database at `prisma/dev.db`

### Path alias

`@/` maps to `src/` (configured in `tsconfig.json`).

### UI components

Uses shadcn/ui (New York style) with Tailwind CSS v4 and Radix UI primitives. Component source is in `src/components/ui/`. Add new shadcn components via `npx shadcn@latest add <component>`.

### Testing

Vitest with jsdom and React Testing Library. Test files sit next to source files (e.g., `component.test.tsx` beside `component.tsx`).

### Authentication

JWT sessions stored in HTTP-only cookies. Anonymous users get a local-only session; authenticated users have their projects stored in the database.
