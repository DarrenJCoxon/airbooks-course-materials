# Agent Guidelines

## Project: AirBooks

### Tech Stack
- **Framework:** Next.js 16+ with App Router
- **Language:** TypeScript (strict mode)
- **Styling:** Tailwind CSS v4
- **Components:** Shadcn UI
- **Package Manager:** pnpm
- **Testing:** Vitest
- **Compression:** lz-string + custom dictionary
- **AI Provider:** Scaleway (EU-hosted) / OpenRouter (fallback)

---

## Core Build Rules (Non-Negotiables)

### 1. Package Management

**System Standard: pnpm**
- **Strict Rule:** Do not mix npm/yarn/pnpm. Lockfile conflicts break CI/CD.
- **Command:** `pnpm install | add | run`
- **Never** use `npm install` or `yarn add`

### 2. Project Structure & Organization

Maintain separation of concerns. Do not mix business logic inside UI components.

```
airbooks/
├── app/                     # Next.js App Router (routes/pages only)
│   ├── [hash]/              # Student view (dynamic route)
│   ├── teacher/             # Teacher configuration
│   └── api/                 # Stateless API routes
├── components/              # Pure UI components (dumb)
│   ├── student/             # Student-facing components
│   ├── teacher/             # Teacher-facing components
│   ├── ui/                  # Shadcn UI base components
│   └── layout/              # Layout components
├── lib/                     # Singletons & utilities
│   ├── api/                 # API clients, validation
│   ├── compression.ts       # State compression logic
│   ├── content-loader.ts    # Content pack loading
│   └── utils.ts             # Utility functions
├── hooks/                   # React state logic
├── types/                   # TypeScript definitions
├── content/                 # Content packs (JSON assets)
├── scripts/                 # Build/utility scripts
├── docs/                    # Architecture & specs
└── public/                  # Static assets
```

### 3. Component Architecture

**Rules:**
1. **UI Components:** Keep dumb (presentational only). Pass data via props.
2. **Feature Components:** Keep in subfolders (`components/student/`, `components/teacher/`).
3. **Shared Logic:** Extract to hooks (`hooks/`) or lib functions.

```
// ✅ Correct - Dumb component
export function SourcePanel({ content, onSceneChange }: SourcePanelProps) {
  return <div>{/* presentation only */}</div>;
}

// ❌ Incorrect - Logic in component
export function SourcePanel() {
  const [content] = useFetchContent(); // Logic belongs in hook
  return <div>{/* ... */}</div>;
}
```

### 4. Styling & UI Architecture

**Framework:** Tailwind CSS v4 + Shadcn UI

**Rules:**
1. **Variables:** Use CSS variables for all colors/spacing (theme-able). No hardcoded hex values in components.
2. **Merging:** Use `cn()` utility (from `lib/utils.ts`) to merge classes.
   ```typescript
   // ✅ Correct
   className={cn("base-styles", isActive && "active-styles")}
   ```
3. **Colocation:** Keep component styles inline with Tailwind classes.
4. **Dark Mode:** Use `dark:` prefix for dark theme support (mandatory).

### 5. Type Safety & Code Quality

- **Strict Mode:** Enabled in `tsconfig.json`.
- **No `any`:** Use `unknown`, Generics, or specific interfaces.
- **File Limits:**
  - Components: < 250 lines (break into sub-components).
  - Logic/API: < 200 lines (extract services).
- **Generated Types:** All content types defined in `types/` directory.

### 6. API & Security Patterns

**Workflow:**
1. **Validate:** Zod schema validation on all inputs.
2. **Execute:** Perform AI operations.
3. **Error Handle:** Wrap in Try/Catch and return standardized errors.

```typescript
// Pattern for API routes
import { z } from 'zod';

const schema = z.object({
  prompt: z.string().min(1),
  context: z.string(),
});

export async function POST(request: Request) {
  try {
    const body = schema.parse(await request.json());
    // ... logic
  } catch (error) {
    return Response.json({ error: 'Invalid input' }, { status: 400 });
  }
}
```

### 7. Statelessness & Data Privacy (CRITICAL)

**Core Principle:** Zero backend persistence. All state in URL hash.

**Rules:**
- **No Database:** Do not add any database (Postgres, Mongo, etc.).
- **No Logging:** Never log prompts, responses, or IP addresses to persistent storage.
- **No PII:** Store zero personally identifiable information.
- **Server-Side Secrets:** API keys in environment variables only.
- **LocalStorage:** Teacher API key stored in browser only (never sent to server).

### 8. Compression & URL State

**Workflow:**
1. State changes → Debounce (500ms)
2. Compress via `lib/compression.ts`
3. Update URL hash via `history.replaceState`
4. No page refresh

```typescript
// Pattern
import { compress } from '@/lib/compression';

function saveState(state: AppState) {
  const hash = compress(state);
  window.history.replaceState(null, '', `#${hash}`);
}
```

### 9. Content Library Standards

**Rules:**
- All content defined in `types/content-library.ts`.
- Content packs stored as JSON in `content/` directory.
- Dictionary generation via `scripts/generate-dict.ts`.
- Server-side content loading only (no client fetch from external URLs).

### 10. AI & Inference Strategy

**Provider Hierarchy:**
- **Primary:** Scaleway (EU-hosted for GDPR compliance)
- **Fallback:** OpenRouter (teacher BYOK only)

**Rules:**
- Use server-side proxy (`/api/chat`) - never expose API key to client.
- Stateless proxy: No request/response logging.
- Context injection: System prompt + mark scheme from server-side content.

### 11. Git & Version Control

**BANNED COMMANDS:**
- `git reset --hard` (unless explicitly authorized for cleanup)
- `git clean -fd`
- `git push --force` on shared branches

**Workflow:**
- Feature Branch (`feat/name`) → Pull Request → Main.
- Commit messages must be descriptive.

### 12. Testing

**Framework:** Vitest with jsdom environment

**Rules:**
- Write unit tests for all lib functions.
- Write component tests for key UI components.
- Test compression round-trips (compress → decompress).
- Run tests before committing: `pnpm test`

### 13. Build Verification

**Pre-commit Checklist:**
- [ ] TypeScript compiles: `npx tsc --noEmit`
- [ ] Linter passes: `pnpm lint`
- [ ] Tests pass: `pnpm test`
- [ ] Build succeeds: `pnpm build`
- [ ] No secrets in code

### 14. Environment Variables

Ensure these are set in `.env.local` and CI/CD:

```env
# Scaleway AI (student inference)
SCALEWAY_API_KEY=

# Optional: OpenRouter/Anthropic for teacher BYOK
OPENROUTER_API_KEY=
ANTHROPIC_API_KEY=

# Rate limiting (optional)
UPSTASH_REDIS_REST_URL=
UPSTASH_REDIS_REST_TOKEN=
```

---

## Quick Reference

| Do | Don't |
|---|---|
| Use `pnpm` | Use `npm` or `yarn` |
| Import from `lib/` | Create new clients in routes |
| Use CSS Variables | Hardcode Hex codes |
| `git reset --soft` | `git reset --hard` |
| Validate inputs (Zod) | Trust client inputs |
| Store state in URL hash | Use databases |
| Use Tailwind classes | Write custom CSS |
| Test compression | Skip testing |
| Keep components <250 lines | Giant component files |
| Use `cn()` for class merging | String concatenation |

---

## Commands

```bash
# Development
pnpm dev              # Start dev server
pnpm build            # Build for production
pnpm start            # Start production server
pnpm lint             # Run ESLint
pnpm test             # Run tests
pnpm generate-dict    # Generate compression dictionary
```

---

## Epic Structure

All stories follow the template in `epics/E1/E1.1.md`. Each story includes:
- User Story
- Acceptance Criteria
- Tasks
- Dev Notes
- Testing Requirements
- Dev Agent Record
- QA Results

When a story is completed:
1. Mark tasks complete in the story file
2. Move story to `epics/completed/`
3. Update status to "Done"
