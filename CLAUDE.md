# CLAUDE.md - AI Assistant Guide for Zola

> **Last Updated**: 2025-11-25
> **Codebase Version**: 0.1.0 (Beta)
> **Purpose**: Comprehensive guide for AI assistants working on the Zola codebase

## Table of Contents

1. [Project Overview](#project-overview)
2. [Tech Stack & Architecture](#tech-stack--architecture)
3. [Directory Structure](#directory-structure)
4. [Key Conventions & Patterns](#key-conventions--patterns)
5. [State Management](#state-management)
6. [API Development](#api-development)
7. [Component Development](#component-development)
8. [Database & Supabase](#database--supabase)
9. [Model Integration](#model-integration)
10. [Testing & Quality](#testing--quality)
11. [Common Tasks](#common-tasks)
12. [Important Files](#important-files)
13. [Don'ts & Pitfalls](#donts--pitfalls)

---

## Project Overview

**Zola** is an open-source, multi-model AI chat interface that supports 100+ AI models across 13 providers. It's built with Next.js 15, React 19, and TypeScript, featuring local-first architecture with Supabase sync.

### Core Features
- Multi-model chat interface with streaming responses
- BYOK (Bring Your Own Key) with encrypted storage
- Ollama integration for local AI models
- File uploads (images, PDFs, documents)
- Project-based chat organization
- Public chat sharing
- Message editing with rollback
- Real-time collaboration

### Key Design Principles
1. **Local-first**: IndexedDB for offline support, Supabase for sync
2. **Security-first**: Encrypted API keys, CSRF protection, CSP headers
3. **Performance**: Server Components by default, optimistic updates
4. **Flexibility**: Works with or without Supabase, optional Ollama
5. **Simplicity**: Avoid over-engineering, keep solutions focused

---

## Tech Stack & Architecture

### Core Framework
```
Next.js 15 (App Router + Turbopack)
├── React 19 (Server + Client Components)
├── TypeScript 5 (Strict mode)
└── Node.js 18+ runtime
```

### Styling
```
Tailwind CSS 4.1 (@import syntax)
├── shadcn/ui (New York style)
├── Radix UI primitives
├── Motion (Framer Motion 12)
└── OKLCH color space
```

### AI Integration
```
Vercel AI SDK v4
├── @ai-sdk/openai (GPT models)
├── @ai-sdk/anthropic (Claude models)
├── @ai-sdk/google (Gemini models)
├── @ai-sdk/mistral (Mistral models)
├── @ai-sdk/xai (Grok models)
├── @openrouter/ai-sdk-provider (OpenRouter)
└── Ollama (local models)
```

### State & Data
```
State Management: Zustand 5 (Provider pattern)
Server State: TanStack Query (React Query) 5
Local Storage: IndexedDB (idb-keyval)
Backend: Supabase (Auth, DB, Storage, Realtime)
```

### Development Tools
```
Linting: ESLint 9 (Next.js config)
Formatting: Prettier 3 (import sorting, Tailwind sorting)
Type Checking: TypeScript strict mode
Deployment: Docker + Vercel
```

---

## Directory Structure

```
zola/
├── app/                          # Next.js App Router
│   ├── api/                      # API route handlers
│   │   ├── chat/                 # Chat streaming endpoint
│   │   ├── create-chat/          # Chat CRUD
│   │   ├── models/               # Model list
│   │   ├── providers/            # Provider list
│   │   ├── user-preferences/     # User settings
│   │   ├── user-keys/            # BYOK key management
│   │   ├── projects/             # Project management
│   │   └── rate-limits/          # Usage tracking
│   ├── components/               # App-specific components
│   │   ├── chat/                 # Chat interface components
│   │   ├── history/              # Chat history sidebar
│   │   ├── layout/               # Layout components
│   │   ├── multi-chat/           # Multi-model comparison
│   │   └── settings/             # Settings UI
│   ├── types/                    # TypeScript definitions
│   │   └── database.types.ts     # Supabase generated types
│   ├── hooks/                    # App-specific hooks
│   ├── c/[chatId]/               # Dynamic chat routes
│   ├── share/[chatId]/           # Public share routes
│   ├── auth/                     # Auth pages
│   ├── layout.tsx                # Root layout with providers
│   └── page.tsx                  # Home page
│
├── components/                   # Shared UI components
│   ├── ui/                       # shadcn/ui base components (27 components)
│   ├── common/                   # Reusable business components
│   │   ├── model-selector/       # Model selection UI
│   │   ├── copy-button/          # Copy to clipboard
│   │   └── feedback-form/        # User feedback
│   ├── prompt-kit/               # AI-focused components
│   │   ├── prompt-input.tsx      # Multi-line input with attachments
│   │   ├── code-block.tsx        # Syntax highlighted code
│   │   ├── markdown.tsx          # Markdown rendering
│   │   └── file-upload.tsx       # File attachment handling
│   └── motion-primitives/        # Animated components
│
├── lib/                          # Core library code
│   ├── chat-store/               # Chat state management
│   │   ├── chats/                # Chat list (provider.tsx, api.ts)
│   │   ├── messages/             # Message history (provider.tsx, api.ts)
│   │   └── session/              # Active session (provider.tsx)
│   ├── model-store/              # Model state (provider.tsx, api.ts)
│   ├── user-store/               # User state (provider.tsx, api.ts)
│   ├── user-preference-store/    # User preferences (provider.tsx, api.ts)
│   ├── models/                   # Model configurations
│   │   ├── index.ts              # Model registry
│   │   ├── openai.ts             # OpenAI models
│   │   ├── anthropic.ts          # Claude models
│   │   └── [provider].ts         # Other providers
│   ├── providers/                # Provider definitions
│   ├── openproviders/            # Provider abstraction layer
│   │   ├── provider-map.ts       # Provider routing
│   │   └── get-api-key.ts        # Key resolution
│   ├── supabase/                 # Supabase clients
│   │   ├── client.ts             # Browser client
│   │   ├── server.ts             # Server client
│   │   └── server-guest.ts       # Guest client
│   ├── hooks/                    # Shared React hooks
│   ├── server/                   # Server-side utilities
│   ├── user-keys/                # BYOK encryption/decryption
│   └── config.ts                 # App configuration
│
├── utils/                        # Utility functions
│   └── supabase/
│       └── middleware.ts         # Session refresh middleware
│
├── public/                       # Static assets
├── middleware.ts                 # Next.js middleware (CSRF, CSP)
├── next.config.ts                # Next.js configuration
├── tailwind.config.ts            # Tailwind configuration
├── tsconfig.json                 # TypeScript configuration
├── .env.example                  # Environment variables template
└── docker-compose.yml            # Docker deployment
```

### Key Directory Purposes

- **`app/api/`**: All API routes return JSON or SSE streams. Use `createErrorResponse` for errors.
- **`lib/*/provider.tsx`**: Zustand stores using Context + Provider pattern
- **`lib/*/api.ts`**: Server-side API functions (Supabase queries, mutations)
- **`components/ui/`**: Pure presentational components from shadcn/ui (no business logic)
- **`components/prompt-kit/`**: AI-specific components with business logic
- **`lib/models/`**: Model configurations (id, name, provider, context window, pricing)

---

## Key Conventions & Patterns

### File Naming
- **Components**: PascalCase files, e.g., `ModelSelector.tsx`
- **Utilities**: kebab-case files, e.g., `create-error-response.ts`
- **Routes**: kebab-case folders, e.g., `user-preferences/`
- **Types**: kebab-case with `.types.ts`, e.g., `database.types.ts`

### Component Patterns

#### Server vs Client Components
```typescript
// Default: Server Component (no directive)
export default function Page() {
  // Can use async/await, direct DB access
  const user = await fetchUser()
  return <div>{user.name}</div>
}

// Client Component (requires "use client")
"use client"
export function InteractiveButton() {
  const [count, setCount] = useState(0)
  return <button onClick={() => setCount(c => c + 1)}>{count}</button>
}
```

**Rule**: Keep Server Components by default. Only use `"use client"` when you need:
- React hooks (useState, useEffect, useContext)
- Event handlers (onClick, onChange)
- Browser APIs (window, localStorage)
- Third-party libraries that require browser APIs

#### Component Structure
```typescript
// Preferred structure for complex components
"use client"

import { useState } from "react"
import { Button } from "@/components/ui/button"

type MyComponentProps = {
  title: string
  onSave: (data: string) => void
  className?: string
}

export function MyComponent({ title, onSave, className }: MyComponentProps) {
  const [data, setData] = useState("")

  return (
    <div className={className}>
      <h2>{title}</h2>
      <input value={data} onChange={(e) => setData(e.target.value)} />
      <Button onClick={() => onSave(data)}>Save</Button>
    </div>
  )
}
```

### Import Aliases
```typescript
// Always use @ alias for imports
import { Button } from "@/components/ui/button"
import { cn } from "@/lib/utils"
import { UserProfile } from "@/lib/user/types"

// NOT: ../../../components/ui/button
```

### Styling Conventions

#### Tailwind Classes
```typescript
// Use cn() for conditional classes
import { cn } from "@/lib/utils"

<div className={cn(
  "flex items-center gap-2",
  isActive && "bg-primary text-primary-foreground",
  className
)} />

// Prettier automatically sorts Tailwind classes
```

#### Color Variables
```typescript
// Use CSS variables (defined in app/globals.css)
<div className="bg-background text-foreground">
  <div className="bg-muted text-muted-foreground" />
</div>

// Dark mode automatically works via next-themes
// Variables defined in :root and .dark
```

### TypeScript Patterns

#### Type Imports
```typescript
// Use type imports for type-only imports
import type { UserProfile } from "@/lib/user/types"
import type { Database } from "@/app/types/database.types"

// Regular import for runtime code
import { createClient } from "@/lib/supabase/server"
```

#### API Response Types
```typescript
// Define request/response types for API routes
type CreateChatRequest = {
  title: string
  model: string
  userId: string
}

type CreateChatResponse = {
  chatId: string
  createdAt: string
}

// In route handler
export async function POST(req: Request) {
  const body = await req.json() as CreateChatRequest
  // ... logic
  return Response.json({ chatId, createdAt } satisfies CreateChatResponse)
}
```

### Error Handling

#### API Routes
```typescript
import { createErrorResponse } from "./utils"

export async function POST(req: Request) {
  try {
    // ... logic
    if (!userId) {
      return createErrorResponse({
        code: "MISSING_USER_ID",
        message: "User ID is required",
        statusCode: 400
      })
    }

    return Response.json({ success: true })
  } catch (error) {
    console.error("Error in API route:", error)
    return createErrorResponse({
      code: "INTERNAL_ERROR",
      message: "An unexpected error occurred",
      statusCode: 500
    })
  }
}
```

#### Client Components
```typescript
"use client"

import { toast } from "sonner"

async function handleSave() {
  try {
    const response = await fetch("/api/save", { method: "POST", body: JSON.stringify(data) })
    if (!response.ok) {
      const error = await response.json()
      throw new Error(error.message)
    }
    toast.success("Saved successfully")
  } catch (error) {
    toast.error(error instanceof Error ? error.message : "Failed to save")
  }
}
```

---

## State Management

### Zustand Store Pattern

All stores follow the **Provider Pattern** with Context API:

```typescript
// lib/[store-name]/provider.tsx
"use client"

import { createContext, useContext, useState } from "react"
import { fetchData, updateData } from "./api"

type StoreContextType = {
  data: DataType | null
  isLoading: boolean
  refresh: () => Promise<void>
  update: (data: Partial<DataType>) => Promise<void>
}

const StoreContext = createContext<StoreContextType | undefined>(undefined)

export function StoreProvider({
  children,
  initialData
}: {
  children: React.ReactNode
  initialData: DataType | null
}) {
  const [data, setData] = useState<DataType | null>(initialData)
  const [isLoading, setIsLoading] = useState(false)

  const refresh = async () => {
    setIsLoading(true)
    try {
      const updated = await fetchData()
      setData(updated)
    } finally {
      setIsLoading(false)
    }
  }

  const update = async (updates: Partial<DataType>) => {
    setIsLoading(true)
    try {
      await updateData(updates)
      setData(prev => prev ? { ...prev, ...updates } : null)
    } finally {
      setIsLoading(false)
    }
  }

  return (
    <StoreContext.Provider value={{ data, isLoading, refresh, update }}>
      {children}
    </StoreContext.Provider>
  )
}

export function useStore() {
  const context = useContext(StoreContext)
  if (!context) throw new Error("useStore must be used within StoreProvider")
  return context
}
```

### Store Organization

**Six main stores** (wrap in `app/layout.tsx`):

1. **UserStore** (`lib/user-store/`)
   - User profile, authentication state
   - Real-time Supabase subscription
   ```typescript
   const { user, isLoading, updateUser, refreshUser, signOut } = useUser()
   ```

2. **ModelStore** (`lib/model-store/`)
   - Available models, user key status, favorites
   ```typescript
   const { models, userKeyStatus, toggleFavorite } = useModelStore()
   ```

3. **ChatsStore** (`lib/chat-store/chats/`)
   - Chat list, CRUD operations, IndexedDB persistence
   ```typescript
   const { chats, createChat, updateChat, deleteChat, pinChat } = useChats()
   ```

4. **ChatSessionStore** (`lib/chat-store/session/`)
   - Active chat ID, current model, streaming state
   ```typescript
   const { activeChatId, currentModel, isStreaming, setActiveChat } = useChatSession()
   ```

5. **MessagesStore** (`lib/chat-store/messages/`)
   - Message history per chat, IndexedDB persistence
   ```typescript
   const { messages, addMessage, updateMessage, deleteMessage } = useMessages()
   ```

6. **UserPreferencesStore** (`lib/user-preference-store/`)
   - System prompt, theme, layout, interaction preferences
   ```typescript
   const { preferences, updatePreferences } = useUserPreferences()
   ```

### Provider Nesting (in `app/layout.tsx`)

```typescript
<UserProvider initialUser={user}>
  <ModelProvider initialModels={models}>
    <ChatsProvider initialChats={chats}>
      <ChatSessionProvider>
        <MessagesProvider>
          <UserPreferencesProvider initialPreferences={preferences}>
            {children}
          </UserPreferencesProvider>
        </MessagesProvider>
      </ChatSessionProvider>
    </ChatsProvider>
  </ModelProvider>
</UserProvider>
```

### IndexedDB Persistence

**Database**: `zola-db` (version 2)
**Object Stores**: `chats`, `messages`, `sync`

```typescript
// Used in ChatsStore and MessagesStore
import { get, set, del } from "idb-keyval"

// Save to IndexedDB
await set(`chat-${chatId}`, chatData)

// Load from IndexedDB
const chatData = await get(`chat-${chatId}`)

// Delete from IndexedDB
await del(`chat-${chatId}`)
```

**Pattern**: Write to IndexedDB first (optimistic), then sync to Supabase in background.

---

## API Development

### API Route Structure

```typescript
// app/api/[route-name]/route.ts
import { createClient } from "@/lib/supabase/server"
import { createErrorResponse } from "./utils"

export async function GET(req: Request) {
  const supabase = await createClient()

  // Get authenticated user
  const { data: { user } } = await supabase.auth.getUser()
  if (!user) {
    return createErrorResponse({
      code: "UNAUTHORIZED",
      message: "Authentication required",
      statusCode: 401
    })
  }

  // Your logic here
  const data = await fetchData(user.id)

  return Response.json({ data })
}

export async function POST(req: Request) {
  const body = await req.json()
  // ... handle POST request
}
```

### Authentication Patterns

#### Standard Routes (requires auth)
```typescript
const supabase = await createClient()
const { data: { user } } = await supabase.auth.getUser()

if (!user) {
  return createErrorResponse({
    code: "UNAUTHORIZED",
    message: "Authentication required",
    statusCode: 401
  })
}
```

#### Guest-Friendly Routes
```typescript
import { createClient } from "@/lib/supabase/server"
import { createGuestClient } from "@/lib/supabase/server-guest"

const supabase = await createClient()
const { data: { user } } = await supabase.auth.getUser()

// If no user, create guest client
const effectiveSupabase = user ? supabase : await createGuestClient()
const userId = user?.id || "guest"
```

### Rate Limiting

```typescript
import { validateAndTrackUsage } from "./api"

// In API route
const supabase = await validateAndTrackUsage({
  userId,
  model,
  isAuthenticated: !!user
})

if (!supabase) {
  return createErrorResponse({
    code: "RATE_LIMIT_EXCEEDED",
    message: "Daily message limit reached",
    statusCode: 429
  })
}
```

**Limits**:
- Guest users: 5 messages/day
- Authenticated users: 1000 messages/day
- Pro models: 500 messages/day

### Streaming Responses (Chat API)

```typescript
import { streamText } from "ai"

export async function POST(req: Request) {
  const { messages, model } = await req.json()

  const provider = getProviderForModel(model)
  const apiKey = await getEffectiveApiKey(userId, provider)

  const result = streamText({
    model: provider(model, { apiKey }),
    messages,
    system: systemPrompt,
    maxTokens: 4096,
    temperature: 0.7,
  })

  return result.toDataStreamResponse()
}
```

### BYOK (Bring Your Own Key)

```typescript
import { getEffectiveApiKey } from "@/lib/user-keys"

// Get user key if available, fallback to env
const apiKey = await getEffectiveApiKey(
  userId,
  "openai" as ProviderWithoutOllama
)

// Use with provider
const provider = openai({ apiKey })
```

**Encryption**: User keys encrypted with AES-256-CBC, IV stored separately.

---

## Component Development

### shadcn/ui Components

**27 base components** in `components/ui/`:
- Form elements: `button`, `input`, `textarea`, `select`, `checkbox`, `switch`
- Overlays: `dialog`, `drawer`, `popover`, `sheet`, `alert-dialog`
- Navigation: `tabs`, `dropdown-menu`, `command`
- Feedback: `toast`, `sonner`, `progress`, `skeleton`
- Layout: `card`, `separator`, `scroll-area`, `sidebar`
- Data: `avatar`, `badge`, `hover-card`, `label`, `tooltip`

**Usage**:
```typescript
import { Button } from "@/components/ui/button"
import { Dialog, DialogContent, DialogHeader } from "@/components/ui/dialog"

<Button variant="default" size="lg">Click me</Button>
<Dialog>
  <DialogContent>
    <DialogHeader>Title</DialogHeader>
  </DialogContent>
</Dialog>
```

### Custom Components (prompt-kit)

#### PromptInput (`components/prompt-kit/prompt-input.tsx`)
Multi-line textarea with:
- File attachment support
- Auto-resize
- Submit on Enter (Shift+Enter for new line)
- Character/token counter

```typescript
<PromptInput
  value={input}
  onChange={setInput}
  onSubmit={handleSubmit}
  onFileAttach={handleFileAttach}
  disabled={isLoading}
/>
```

#### CodeBlock (`components/prompt-kit/code-block.tsx`)
Syntax-highlighted code with:
- Shiki syntax highlighting
- Copy button
- Language badge
- Line numbers

```typescript
<CodeBlock
  code="const x = 42"
  language="typescript"
  showLineNumbers
/>
```

#### Markdown (`components/prompt-kit/markdown.tsx`)
Markdown rendering with:
- react-markdown + remark-gfm
- DOMPurify sanitization
- Custom code blocks (CodeBlock component)
- Link handling

```typescript
<Markdown content={messageContent} />
```

### Animation Components

**Motion Primitives** (`components/motion-primitives/`):
- `morphing-dialog`: Smooth dialog transitions
- `morphing-popover`: Animated popover
- `text-morph`: Text transition effects
- `progressive-blur`: Blur animations

```typescript
import { MorphingDialog } from "@/components/motion-primitives/morphing-dialog"

<MorphingDialog open={isOpen} onOpenChange={setIsOpen}>
  <MorphingDialogContent>
    Content here
  </MorphingDialogContent>
</MorphingDialog>
```

### Component Guidelines

1. **Composition over Props**: Break into smaller components rather than adding many props
2. **Accessibility**: Use Radix UI primitives for built-in a11y
3. **TypeScript**: Always define prop types
4. **Styling**: Use Tailwind + cn() for conditional classes
5. **Server First**: Start with Server Components, add "use client" only when needed

---

## Database & Supabase

### Database Schema

**Tables**:
1. **users** - User profiles, usage tracking, preferences
2. **chats** - Chat metadata (title, model, created_at, pinned, public)
3. **messages** - Chat messages (content, role, attachments, parts)
4. **projects** - Chat organization/folders
5. **chat_attachments** - File upload metadata
6. **user_keys** - Encrypted API keys (BYOK)
7. **user_preferences** - User settings
8. **feedback** - User feedback

**Key Relationships**:
```
users (1) -> (N) chats
chats (1) -> (N) messages
users (1) -> (N) projects
projects (1) -> (N) chats
chats (1) -> (N) chat_attachments
users (1) -> (N) user_keys
users (1) -> (1) user_preferences
```

### Supabase Clients

#### Server-side (API routes)
```typescript
import { createClient } from "@/lib/supabase/server"

const supabase = await createClient()
const { data, error } = await supabase
  .from("chats")
  .select("*")
  .eq("user_id", userId)
```

#### Client-side (components)
```typescript
import { createBrowserClient } from "@/lib/supabase/client"

const supabase = createBrowserClient()
const { data, error } = await supabase
  .from("chats")
  .select("*")
  .eq("user_id", userId)
```

#### Guest users
```typescript
import { createGuestClient } from "@/lib/supabase/server-guest"

const supabase = await createGuestClient()
// Limited access, read-only for public content
```

### Row Level Security (RLS)

All tables have RLS enabled. Users can only access their own data:

```sql
-- Example policy
CREATE POLICY "Users can view own chats"
  ON chats FOR SELECT
  USING (auth.uid() = user_id);

CREATE POLICY "Users can update own chats"
  ON chats FOR UPDATE
  USING (auth.uid() = user_id);
```

### Realtime Subscriptions

```typescript
// In store provider
useEffect(() => {
  if (!userId) return

  const subscription = supabase
    .channel(`user-${userId}`)
    .on(
      "postgres_changes",
      { event: "UPDATE", schema: "public", table: "users", filter: `id=eq.${userId}` },
      (payload) => {
        setUser(prev => ({ ...prev, ...payload.new }))
      }
    )
    .subscribe()

  return () => {
    subscription.unsubscribe()
  }
}, [userId])
```

### Storage (File Uploads)

**Buckets**: `chat-attachments`, `avatars`

```typescript
// Upload file
const { data, error } = await supabase.storage
  .from("chat-attachments")
  .upload(`${userId}/${fileName}`, file)

// Get public URL
const { data: { publicUrl } } = supabase.storage
  .from("chat-attachments")
  .getPublicUrl(filePath)

// Delete file
await supabase.storage
  .from("chat-attachments")
  .remove([filePath])
```

**File Limits**:
- Max size: 10MB
- Daily uploads: 5 for free users
- Supported: images (PNG, JPG, WebP, GIF), PDFs, text files, spreadsheets

---

## Model Integration

### Model Registry (`lib/models/`)

Models organized by provider in separate files:

```
lib/models/
├── index.ts              # Main model registry
├── openai.ts             # OpenAI models (GPT-4, GPT-3.5, etc.)
├── anthropic.ts          # Claude models
├── google.ts             # Gemini models
├── mistral.ts            # Mistral models
├── xai.ts                # Grok models
├── perplexity.ts         # Perplexity models
└── openrouter.ts         # OpenRouter models
```

### Model Configuration

```typescript
// lib/models/[provider].ts
import type { ModelConfig } from "./types"

export const openaiModels: ModelConfig[] = [
  {
    id: "gpt-4o",
    name: "GPT-4o",
    provider: "openai",
    providerId: "openai",
    apiSdk: "openai",
    contextWindow: 128000,
    maxOutputTokens: 16384,
    pricing: {
      inputTokens: 2.5,
      outputTokens: 10.0
    },
    tags: ["pro", "coding", "multimodal"],
    featured: true,
    capabilities: {
      vision: true,
      functionCalling: true,
      streaming: true
    }
  }
]
```

### Adding a New Model

1. **Add to provider file** (`lib/models/[provider].ts`):
```typescript
export const myProviderModels: ModelConfig[] = [
  {
    id: "model-id",
    name: "Model Name",
    provider: "provider-name",
    providerId: "provider-name",
    apiSdk: "provider-name",
    contextWindow: 8192,
    maxOutputTokens: 4096,
    tags: ["open-source"],
    capabilities: { streaming: true }
  }
]
```

2. **Add SDK provider** (`lib/openproviders/provider-map.ts`):
```typescript
import { createMyProvider } from "@ai-sdk/my-provider"

export function getProviderForModel(modelId: string) {
  if (modelId.startsWith("model-id")) {
    return (model: string, config: { apiKey?: string }) =>
      createMyProvider({ apiKey: config.apiKey })(model)
  }
  // ... other providers
}
```

3. **Add to model list** (`lib/models/index.ts`):
```typescript
import { myProviderModels } from "./my-provider"

export async function getAllModels(): Promise<ModelConfig[]> {
  return [
    ...openaiModels,
    ...anthropicModels,
    ...myProviderModels, // Add here
    // ... other models
  ]
}
```

4. **Add environment variable** (`.env.example`):
```bash
MY_PROVIDER_API_KEY=your_api_key_here
```

### Ollama Integration

Ollama models are **automatically detected** at runtime:

```typescript
// lib/models/ollama.ts
export async function getOllamaModels(): Promise<ModelConfig[]> {
  const baseUrl = process.env.OLLAMA_BASE_URL || "http://localhost:11434"

  const response = await fetch(`${baseUrl}/api/tags`)
  const data = await response.json()

  return data.models.map(model => ({
    id: `ollama:${model.name}`,
    name: model.name,
    provider: "ollama",
    providerId: "ollama",
    apiSdk: "ollama",
    contextWindow: 8192,
    tags: ["local", "open-source"],
    capabilities: { streaming: true }
  }))
}
```

**No configuration needed** - Zola automatically detects all Ollama models.

---

## Testing & Quality

### Current State

**Testing**: No test files currently exist. Consider adding:
- Unit tests (Vitest recommended)
- Integration tests (API routes)
- E2E tests (Playwright)

### Type Checking

```bash
# Run type check
npm run type-check

# Check before commit
tsc --noEmit
```

### Linting

```bash
# Run ESLint
npm run lint

# Fix auto-fixable issues
npm run lint -- --fix
```

**Note**: `ignoreDuringBuilds: true` in `next.config.ts` - should be fixed.

### Code Formatting

```bash
# Format all files
npx prettier --write .

# Check formatting
npx prettier --check .
```

**Prettier plugins**:
- Import sorting (`@ianvs/prettier-plugin-sort-imports`)
- Tailwind class sorting (`prettier-plugin-tailwindcss`)

### Security Checklist

When adding new features, verify:

- [ ] Input validation (Zod recommended)
- [ ] CSRF token validation (POST/PUT/DELETE)
- [ ] XSS prevention (use DOMPurify for user content)
- [ ] SQL injection prevention (use Supabase parameterized queries)
- [ ] File upload validation (magic bytes, size limits)
- [ ] API key encryption (use `lib/user-keys/encrypt.ts`)
- [ ] Rate limiting (use `validateAndTrackUsage`)
- [ ] Authentication check (use Supabase auth)

---

## Common Tasks

### Adding a New API Route

1. **Create route file**:
```typescript
// app/api/my-route/route.ts
import { createClient } from "@/lib/supabase/server"
import { createErrorResponse } from "../utils"

export async function POST(req: Request) {
  const supabase = await createClient()
  const { data: { user } } = await supabase.auth.getUser()

  if (!user) {
    return createErrorResponse({
      code: "UNAUTHORIZED",
      message: "Authentication required",
      statusCode: 401
    })
  }

  const body = await req.json()

  // Your logic here

  return Response.json({ success: true })
}
```

2. **Add to API folder**: `app/api/my-route/route.ts`

3. **Test with curl**:
```bash
curl -X POST http://localhost:3000/api/my-route \
  -H "Content-Type: application/json" \
  -d '{"data": "value"}'
```

### Adding a New Store

1. **Create provider** (`lib/my-store/provider.tsx`):
```typescript
"use client"

import { createContext, useContext, useState } from "react"
import { fetchData, updateData } from "./api"
import type { MyDataType } from "./types"

type MyStoreContextType = {
  data: MyDataType | null
  isLoading: boolean
  refresh: () => Promise<void>
}

const MyStoreContext = createContext<MyStoreContextType | undefined>(undefined)

export function MyStoreProvider({
  children,
  initialData
}: {
  children: React.ReactNode
  initialData: MyDataType | null
}) {
  const [data, setData] = useState<MyDataType | null>(initialData)
  const [isLoading, setIsLoading] = useState(false)

  const refresh = async () => {
    setIsLoading(true)
    try {
      const updated = await fetchData()
      setData(updated)
    } finally {
      setIsLoading(false)
    }
  }

  return (
    <MyStoreContext.Provider value={{ data, isLoading, refresh }}>
      {children}
    </MyStoreContext.Provider>
  )
}

export function useMyStore() {
  const context = useContext(MyStoreContext)
  if (!context) throw new Error("useMyStore must be used within MyStoreProvider")
  return context
}
```

2. **Create API functions** (`lib/my-store/api.ts`):
```typescript
import { createBrowserClient } from "@/lib/supabase/client"

export async function fetchData() {
  const supabase = createBrowserClient()
  const { data, error } = await supabase.from("my_table").select("*")
  if (error) throw error
  return data
}
```

3. **Add to root layout** (`app/layout.tsx`):
```typescript
<MyStoreProvider initialData={initialData}>
  {children}
</MyStoreProvider>
```

### Adding a New UI Component

1. **Use shadcn/ui if available**:
```bash
# Check if component exists
npx shadcn@latest add [component-name]
```

2. **Create custom component**:
```typescript
// components/common/my-component.tsx
"use client"

import { Button } from "@/components/ui/button"
import { cn } from "@/lib/utils"

type MyComponentProps = {
  title: string
  className?: string
}

export function MyComponent({ title, className }: MyComponentProps) {
  return (
    <div className={cn("flex items-center gap-2", className)}>
      <h3>{title}</h3>
      <Button>Action</Button>
    </div>
  )
}
```

3. **Export from index** (if in common/):
```typescript
// components/common/index.ts
export { MyComponent } from "./my-component"
```

### Adding Database Table

1. **Create migration SQL**:
```sql
-- Add to database via Supabase SQL editor
CREATE TABLE my_table (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  name TEXT NOT NULL,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Enable RLS
ALTER TABLE my_table ENABLE ROW LEVEL SECURITY;

-- Add policies
CREATE POLICY "Users can view own data"
  ON my_table FOR SELECT
  USING (auth.uid() = user_id);

CREATE POLICY "Users can insert own data"
  ON my_table FOR INSERT
  WITH CHECK (auth.uid() = user_id);
```

2. **Generate types**:
```bash
# In Supabase dashboard: Settings > API > Generate TypeScript Types
# Copy to app/types/database.types.ts
```

3. **Use in code**:
```typescript
import type { Database } from "@/app/types/database.types"

type MyTable = Database["public"]["Tables"]["my_table"]["Row"]
```

---

## Important Files

### Configuration Files

- **`next.config.ts`**: Next.js configuration (standalone output, image domains)
- **`middleware.ts`**: Supabase session refresh, CSRF protection, CSP headers
- **`tailwind.config.ts`**: Tailwind configuration (though Tailwind v4 uses CSS)
- **`tsconfig.json`**: TypeScript configuration (strict mode, path aliases)
- **`.env.example`**: Environment variables template (29 variables)
- **`lib/config.ts`**: App configuration (limits, defaults, system prompt)

### Key Source Files

- **`app/layout.tsx`**: Root layout with all providers
- **`app/api/chat/route.ts`**: Main chat streaming endpoint (most complex API)
- **`lib/models/index.ts`**: Model registry (100+ models)
- **`lib/openproviders/provider-map.ts`**: Provider routing logic
- **`lib/user-keys/index.ts`**: BYOK encryption/decryption
- **`components/prompt-kit/prompt-input.tsx`**: Main chat input
- **`components/prompt-kit/markdown.tsx`**: Message rendering
- **`app/types/database.types.ts`**: Supabase type definitions

### Database Schema

- **`INSTALL.md`**: Complete database schema SQL (lines 158-290)

---

## Don'ts & Pitfalls

### ❌ Common Mistakes

1. **Don't import server code in client components**
   ```typescript
   // ❌ BAD
   "use client"
   import { createClient } from "@/lib/supabase/server" // Server-only!

   // ✅ GOOD
   "use client"
   import { createBrowserClient } from "@/lib/supabase/client"
   ```

2. **Don't add "use client" unless necessary**
   ```typescript
   // ❌ BAD - Unnecessarily client component
   "use client"
   export default function StaticPage({ data }) {
     return <div>{data}</div>
   }

   // ✅ GOOD - Server component
   export default function StaticPage({ data }) {
     return <div>{data}</div>
   }
   ```

3. **Don't ignore CSRF protection**
   ```typescript
   // ❌ BAD - No CSRF token
   fetch("/api/update", { method: "POST", body: JSON.stringify(data) })

   // ✅ GOOD - Include CSRF token
   const csrfToken = await fetch("/api/csrf").then(r => r.json())
   fetch("/api/update", {
     method: "POST",
     headers: { "x-csrf-token": csrfToken },
     body: JSON.stringify(data)
   })
   ```

4. **Don't use relative imports for components**
   ```typescript
   // ❌ BAD
   import { Button } from "../../../components/ui/button"

   // ✅ GOOD
   import { Button } from "@/components/ui/button"
   ```

5. **Don't forget to validate user input**
   ```typescript
   // ❌ BAD
   const { data } = await req.json()
   await supabase.from("table").insert(data)

   // ✅ GOOD
   const body = await req.json()
   if (!body.name || typeof body.name !== "string") {
     return createErrorResponse({ message: "Invalid name" })
   }
   ```

6. **Don't expose API keys in client code**
   ```typescript
   // ❌ BAD
   const apiKey = process.env.OPENAI_API_KEY // Exposed to client!

   // ✅ GOOD - Keep in API routes
   export async function POST(req: Request) {
     const apiKey = process.env.OPENAI_API_KEY // Server-side only
   }
   ```

7. **Don't create components without TypeScript types**
   ```typescript
   // ❌ BAD
   export function MyComponent({ title, onClick }) {
     return <button onClick={onClick}>{title}</button>
   }

   // ✅ GOOD
   type MyComponentProps = {
     title: string
     onClick: () => void
   }
   export function MyComponent({ title, onClick }: MyComponentProps) {
     return <button onClick={onClick}>{title}</button>
   }
   ```

8. **Don't ignore ESLint/TypeScript errors**
   - Never commit with `@ts-ignore` or `eslint-disable`
   - Fix the root cause instead

9. **Don't add unnecessary dependencies**
   - Check if shadcn/ui or existing utils can solve the problem
   - Prefer vanilla solutions over libraries for simple tasks

10. **Don't forget to add environment variables to `.env.example`**
    - Document all required and optional env vars
    - Provide example values

### ⚠️ Security Pitfalls

1. **SQL Injection**: Always use Supabase's parameterized queries
   ```typescript
   // ❌ BAD
   .eq("id", userId) // Don't trust user input

   // ✅ GOOD - Validate first
   if (!isValidUuid(userId)) throw new Error("Invalid ID")
   .eq("id", userId)
   ```

2. **XSS**: Sanitize user content before rendering
   ```typescript
   // ✅ Markdown component uses DOMPurify
   <Markdown content={userContent} />
   ```

3. **CSRF**: Validate token on state-changing operations
   ```typescript
   // middleware.ts handles this automatically for POST/PUT/DELETE
   ```

4. **File Uploads**: Validate file type with magic bytes, not just extension
   ```typescript
   import { fileTypeFromBuffer } from "file-type"
   const type = await fileTypeFromBuffer(buffer)
   ```

### 🚀 Performance Tips

1. **Use Server Components by default** - Reduce JS bundle size
2. **Optimize imports** - Import only what you need
   ```typescript
   // ✅ GOOD
   import { Button } from "@/components/ui/button"

   // ❌ BAD
   import * as Components from "@/components/ui"
   ```
3. **Use IndexedDB for large data** - Chat/message history
4. **Implement optimistic updates** - Update UI before API call completes
5. **Lazy load heavy components** - Use `next/dynamic`
   ```typescript
   import dynamic from "next/dynamic"
   const HeavyComponent = dynamic(() => import("./heavy-component"))
   ```

---

## Quick Reference

### Environment Variables (Required)

```bash
# Supabase (Required)
NEXT_PUBLIC_SUPABASE_URL=https://xxx.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=xxx
SUPABASE_SERVICE_ROLE=xxx

# Security (Required)
CSRF_SECRET=xxx # 32 character random string

# BYOK (Required for user API keys)
ENCRYPTION_KEY=xxx # 32-byte base64 key

# AI Providers (At least one required)
OPENAI_API_KEY=xxx
ANTHROPIC_API_KEY=xxx
GOOGLE_GENERATIVE_AI_API_KEY=xxx
MISTRAL_API_KEY=xxx
XAI_API_KEY=xxx
PERPLEXITY_API_KEY=xxx
OPENROUTER_API_KEY=xxx

# Ollama (Optional, default: http://localhost:11434)
OLLAMA_BASE_URL=http://localhost:11434
DISABLE_OLLAMA=false # Set to true to disable

# Tools (Optional)
EXA_API_KEY=xxx # Search integration
```

### NPM Scripts

```bash
npm run dev          # Start dev server (with turbopack)
npm run build        # Build for production
npm start            # Start production server
npm run lint         # Run ESLint
npm run type-check   # Run TypeScript type checking
```

### Common Imports

```typescript
// UI Components
import { Button } from "@/components/ui/button"
import { Dialog } from "@/components/ui/dialog"

// AI Components
import { PromptInput } from "@/components/prompt-kit/prompt-input"
import { Markdown } from "@/components/prompt-kit/markdown"

// Utils
import { cn } from "@/lib/utils"

// Stores
import { useUser } from "@/lib/user-store/provider"
import { useChats } from "@/lib/chat-store/chats/provider"
import { useMessages } from "@/lib/chat-store/messages/provider"

// Supabase
import { createClient } from "@/lib/supabase/server" // API routes
import { createBrowserClient } from "@/lib/supabase/client" // Client components

// Types
import type { Database } from "@/app/types/database.types"
import type { UserProfile } from "@/lib/user/types"
```

### Helpful Commands

```bash
# Generate CSRF secret
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"

# Generate encryption key
node -e "console.log(require('crypto').randomBytes(32).toString('base64'))"

# Check Ollama models
curl http://localhost:11434/api/tags

# Pull Ollama model
ollama pull llama3.2

# Check TypeScript errors
npx tsc --noEmit

# Format code
npx prettier --write .

# Analyze bundle size
ANALYZE=true npm run build
```

---

## Getting Help

### Documentation
- **Next.js**: https://nextjs.org/docs
- **Supabase**: https://supabase.com/docs
- **Vercel AI SDK**: https://sdk.vercel.ai/docs
- **shadcn/ui**: https://ui.shadcn.com
- **Tailwind CSS**: https://tailwindcss.com/docs

### Codebase
- **README.md**: Quick start guide
- **INSTALL.md**: Detailed installation and setup
- **CODE_OF_CONDUCT.md**: Community guidelines

### Community
- **GitHub Issues**: https://github.com/ibelick/zola/issues
- **GitHub Discussions**: https://github.com/ibelick/zola/discussions

---

## Changelog

- **2025-11-25**: Initial CLAUDE.md created based on codebase analysis

---

**Note**: This is a living document. Update it when major architectural changes are made to the codebase.
