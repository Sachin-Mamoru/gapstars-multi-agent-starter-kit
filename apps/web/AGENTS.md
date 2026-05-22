# Agent Instructions — Web

> Part of a monorepo — read [../../README.md](../../README.md) and [../../AGENTS.md](../../AGENTS.md) first.

Next.js 15 + shadcn/ui frontend. TypeScript, `bun` package manager.

## Dev Commands

```bash
bun install      # install / update dependencies
bun dev          # Turbopack dev server → :3000
bun build        # production build (standalone output for Docker)
bun lint         # ESLint
bun typecheck    # tsc --noEmit
bun format       # Prettier
```

## Source Layout

```
app/
├── globals.css        Tailwind v4 imports + OKLCH color tokens + dark mode vars
├── layout.tsx         Root layout — fonts, metadata, ThemeProvider
└── page.tsx           Home page — renders <ChatInterface />

components/
├── theme-provider.tsx next-themes wrapper; 'd' key toggles dark/light
├── chat/
│   ├── chat-interface.tsx   Top-level layout — header, MessageList, MessageInput
│   ├── message-list.tsx     Scrollable feed — markdown rendering, streaming indicator
│   ├── message-input.tsx    Auto-resize textarea, Enter to send
│   └── provider-selector.tsx  Dropdown to switch openai / mistral
└── ui/                shadcn/ui components — DO NOT edit manually

hooks/
└── use-chat.ts        All chat state — messages, isStreaming, error, threadId

lib/
├── api.ts             streamChat() async generator — SSE consumption
├── types.ts           ChatMessage, StreamChunk, LLMProvider types + API_URL const
└── utils.ts           cn() — clsx + tailwind-merge
```

## Key Data Flow

```
ChatInterface
  └── useChat(provider)       ← owns all state
        └── streamChat()      ← async generator → POST /api/chat/stream
              └── eventsource-parser
```

## useChat Hook (`hooks/use-chat.ts`)

- `sendMessage(content)` — optimistically appends user + assistant placeholder, accumulates tokens character by character
- `resetThread()` — new UUID thread, clears messages; called automatically on provider change
- Thread ID lives in a `useRef` — changing provider resets it to prevent cross-provider state bleed

## streamChat (`lib/api.ts`)

```ts
streamChat(message: string, threadId: string, provider?: LLMProvider): AsyncGenerator<StreamChunk>
```
Fetches `POST ${API_URL}/api/chat/stream`, parses SSE via `eventsource-parser`, yields each `StreamChunk` immediately without buffering.

## How to Add a Page

1. Create `app/<route>/page.tsx` (Next.js App Router)
2. Add `"use client"` if the page uses hooks or browser APIs

## How to Add a UI Component

- **Custom component:** `components/chat/<name>.tsx`, use `cn()` from `lib/utils.ts` for class composition
- **shadcn/ui component:** `bunx shadcn add <name>` — writes to `components/ui/`, never edit those files manually

## Coding Conventions

- `"use client"` at the top of any component using hooks or browser APIs
- `cn()` for all className composition (`clsx` + `tailwind-merge`)
- No semicolons, 80-char line width — Prettier enforced
- Path alias `@/*` maps to project root (`tsconfig.json`)
- Markdown in assistant messages uses `ReactMarkdown` + `remark-gfm`; all element overrides live in `message-list.tsx`
- Icons: `lucide-react` only
- Tailwind CSS v4 — CSS-first config (no `tailwind.config.js`); theme tokens in `app/globals.css`
- Dark mode: class strategy via `next-themes` (`.dark` on `<html>`)
