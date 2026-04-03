Before I write every single file, I need the answer to one critical question that changes multiple skills fundamentally:

**Are you using Next.js App Router OR TanStack Router — or both, and how?**

Because `page-anatomy`, `routing-patterns`, `add-page` command — they're completely different files depending on this answer.

Also quick confirms:

1. **31 character limit** — is this for ALL text inputs across the app, or specific ones?
2. **Routes in constants** — do you already have a pattern like `ROUTES.DASHBOARD` or should I define that convention from scratch?
3. **`router.jsx`** — this sounds like TanStack Router's `createRouter`. Is your file actually `.jsx` or `.tsx`?

While you answer those, let me write every skill that is **stack-independent** right now — these don't change regardless of your answers.

---

I'll write them all. Starting now.

---

### `.claude/skills/styling-system/SKILL.md`

```md
---
name: styling-system
description: Use when writing className, applying styles, using Tailwind classes, 
             defining CSS variables, working with Shadcn themes, dark mode, 
             scrollbars, spacing, colors, or any visual styling decision.
---

# Styling System

## Core Rules — Never Violate

- **Never hardcode colors** — always use CSS variables or Tailwind tokens
- **Never use arbitrary values** like `w-[437px]` unless absolutely no token fits — document why
- **Never mix styling approaches** — no inline `style={{}}` for layout, only for truly dynamic computed values
- **Never use `!important`**
- **Class order**: layout → sizing → spacing → typography → color → border → shadow → animation → state modifiers

## Tailwind Usage

Use semantic class groupings, always in this order:
```tsx
// CORRECT
<div className="flex items-center gap-4 w-full px-4 py-2 text-sm font-medium text-foreground bg-background border border-border rounded-md shadow-sm hover:bg-accent transition-colors" />

// WRONG — random order
<div className="text-sm bg-background hover:bg-accent flex border-border items-center" />
```

## CSS Variables — index.css

All design tokens live in `index.css` under `:root` and `.dark`.

**Never define a color anywhere else.**

```css
:root {
  --background: 0 0% 100%;
  --foreground: 222.2 84% 4.9%;
  --primary: 221.2 83.2% 53.3%;
  /* etc — follow Shadcn convention */
}
```

When adding a new token:
1. Add to `:root`
2. Add dark mode value to `.dark`
3. Register in `tailwind.config.ts` under `extend.colors`
4. Never use the raw HSL value in components — only the variable

## Scrollbars — Always Invisible, Always Functional

Every scrollable container **must** have invisible scrollbars. Scrollbars provide function, not decoration.

```css
/* index.css — global, always present */
* {
  scrollbar-width: none; /* Firefox */
  -ms-overflow-style: none; /* IE/Edge */
}

*::-webkit-scrollbar {
  display: none; /* Chrome/Safari */
}
```

For any component that scrolls:
```tsx
// CORRECT
<div className="overflow-y-auto">

// WRONG — never show native scrollbar UI
<div className="overflow-y-scroll"> // only if you explicitly want visible scrollbar (you don't)
```

## Shadcn Theme Tokens

Always prefer Shadcn semantic tokens over raw Tailwind colors:

| Instead of | Use |
|---|---|
| `text-gray-900` | `text-foreground` |
| `bg-white` | `bg-background` |
| `bg-gray-100` | `bg-muted` |
| `text-gray-500` | `text-muted-foreground` |
| `border-gray-200` | `border-border` |
| `bg-blue-600` | `bg-primary` |
| `text-blue-600` | `text-primary` |
| `bg-red-50` | `bg-destructive/10` |

## Dark Mode

- Never hardcode dark mode with `dark:bg-gray-900` using raw colors
- Always use `dark:` modifier with semantic tokens only
- Component must look correct in both modes before PR

## Responsive Design

Mobile-first always:
```tsx
// CORRECT — mobile base, then up
<div className="w-full md:w-1/2 lg:w-1/3" />

// WRONG — desktop base
<div className="w-1/3 sm:w-full" />
```

Breakpoints: `sm:640px` `md:768px` `lg:1024px` `xl:1280px` `2xl:1536px`

## Fonts

Never import fonts inside components. Fonts are configured once in the root layout / `index.css`.

## What Belongs in CSS vs Tailwind

| In `index.css` | In `className` |
|---|---|
| CSS variables / tokens | Component-level styles |
| Global resets | Responsive variants |
| Scrollbar hiding | State modifiers |
| Font-face declarations | Layout composition |
| Keyframe animations | Spacing / sizing |
```

---

### `.claude/skills/add-component/SKILL.md`

```md
---
name: add-component
description: Use when creating, adding, scaffolding, or defining any new React component.
             Triggers on "new component", "create component", "add component", "build component".
---

# Adding a Component

## File Structure

Every component lives in its own folder:

```
src/components/<domain>/<ComponentName>/
├── index.tsx          ← the component
├── types.ts           ← props interface and any local types
└── index.ts           ← barrel export
```

For Shadcn components: they live in `src/components/ui/` — never modify them directly. Wrap them.

## Naming

- **Folders**: PascalCase — `ProductCard/`
- **Files**: as above — `index.tsx`, `types.ts`, `index.ts`
- **Component function**: PascalCase, matches folder name
- **Props interface**: `<ComponentName>Props` — e.g., `ProductCardProps`

## Props — Always Interface, Never Inline Type

```tsx
// types.ts — CORRECT
export interface ProductCardProps {
  id: string
  title: string
  price: number
  onSelect: (id: string) => void
  className?: string  // always optional className for composability
}

// WRONG — never inline, never type alias for props
const ProductCard = ({ id }: { id: string }) => { ... }
type ProductCardProps = { ... }  // use interface, not type
```

## Component Structure — Exact Order

```tsx
// index.tsx

import { cn } from '@/lib/utils'
import type { ProductCardProps } from './types'

// 1. Sub-components or constants local to this component (if any)

// 2. The component — named export only, never default export
export function ProductCard({ id, title, price, onSelect, className }: ProductCardProps) {
  // 3. Hooks first — in this order:
  //    - context hooks
  //    - store hooks (zustand)
  //    - query hooks
  //    - local state (useState)
  //    - refs (useRef)
  //    - derived values (useMemo)
  //    - effects last (useEffect)

  // 4. Handlers — prefixed with "handle"
  function handleSelect() {
    onSelect(id)
  }

  // 5. Early returns for loading/error states

  // 6. Main render
  return (
    <div className={cn('...your base classes', className)}>
      ...
    </div>
  )
}
```

## Barrel Export

```ts
// index.ts
export { ProductCard } from './index'
export type { ProductCardProps } from './types'
```

## Rules

- **Always** accept and spread `className` prop using `cn()` for composability
- **Never** use default exports for components
- **Never** put business logic inside a component — extract to hooks or utils
- **Never** call API functions directly inside components — always through TanStack Query
- **Never** access Zustand store directly in deeply nested components — pass via props or colocate the store hook at the right level
- **Always** add display name for debugging: `ProductCard.displayName = 'ProductCard'`
- Components over 200 lines are a signal to split — not a hard rule, use judgment
- No `any` — ever

## Shadcn Component Usage

When using a Shadcn component:
```tsx
// CORRECT — import from ui, use as-is or wrap
import { Button } from '@/components/ui/button'

// If customizing behavior — create a wrapper, don't modify the ui component
// src/components/common/ConfirmButton/index.tsx wraps Button with confirmation logic
```

## Checklist Before Finishing

- [ ] Props interface defined in `types.ts`
- [ ] Named export (not default)
- [ ] `className` prop accepted with `cn()`
- [ ] Hooks in correct order
- [ ] Handlers prefixed with `handle`
- [ ] Barrel export in `index.ts`
- [ ] No `any`
- [ ] No direct API calls
- [ ] displayName set
```

---

### `.claude/skills/typescript-strict-patterns/SKILL.md`

```md
---
name: typescript-strict-patterns
description: Use when writing types, interfaces, generics, enums, utility types, 
             type assertions, or any TypeScript-specific syntax. Triggers on 
             "type", "interface", "generic", "enum", "infer", "as", "satisfies".
---

# TypeScript Strict Patterns

## The Absolute Rules

- **`any` is banned.** If you think you need `any`, use `unknown` and narrow it.
- **No type assertions (`as SomeType`)** unless you can prove runtime safety — comment why.
- **No non-null assertions (`!`)** unless you can prove it — comment why.
- **Explicit return types on all functions** that are exported or non-trivial.
- **`strict: true`** in tsconfig — never loosen it.
- **`satisfies` over `as`** when you need to ensure shape without widening.

## Interface vs Type

```ts
// INTERFACE — for object shapes, component props, API responses
interface User {
  id: string
  email: string
  role: UserRole
}

// TYPE — for unions, intersections, computed types, primitives
type UserRole = 'admin' | 'editor' | 'viewer'
type ApiResponse<T> = { data: T; error: null } | { data: null; error: string }
type PartialUser = Pick<User, 'id' | 'email'>
```

## Enums — Never Use TypeScript Enums

They compile to objects, cause bundle bloat, and have footguns.

```ts
// WRONG
enum Direction { Up, Down }

// CORRECT — const object + type
const DIRECTION = {
  UP: 'up',
  DOWN: 'down',
} as const

type Direction = typeof DIRECTION[keyof typeof DIRECTION]
// Direction = 'up' | 'down'
```

## Discriminated Unions — Use for State

```ts
// CORRECT — every state is explicit and exhaustive
type RequestState<T> =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; error: string }

// Then narrow explicitly:
if (state.status === 'success') {
  // TypeScript knows state.data exists here
}
```

## Generics

```ts
// Good generic — constrained, purposeful
function getById<T extends { id: string }>(items: T[], id: string): T | undefined {
  return items.find(item => item.id === id)
}

// Bad generic — unconstrained, same as any
function process<T>(value: T): T { ... }  // What is T? Constrain it.
```

## Zod + TypeScript Inference

Never define a type AND a Zod schema for the same thing. Infer the type from Zod:

```ts
// CORRECT
const userSchema = z.object({
  id: z.string().uuid(),
  email: z.string().email(),
  role: z.enum(['admin', 'editor', 'viewer']),
})

type User = z.infer<typeof userSchema>  // Single source of truth

// WRONG — duplicating the shape
interface User { id: string; email: string }  // drift risk
const userSchema = z.object({ ... })
```

## Unknown over Any

```ts
// WRONG
function parseResponse(data: any) { return data.user }

// CORRECT
function parseResponse(data: unknown): User {
  return userSchema.parse(data)  // Zod validates and narrows
}
```

## Utility Types — Use Them

```ts
type CreateUserInput = Omit<User, 'id' | 'createdAt'>
type UpdateUserInput = Partial<Pick<User, 'email' | 'role'>>
type UserPreview = Readonly<Pick<User, 'id' | 'email'>>
```

## Function Signatures

```ts
// Always explicit return type on exports
export function formatPrice(cents: number): string {
  return `$${(cents / 100).toFixed(2)}`
}

// Arrow functions for simple pure transforms
const toUpperCase = (s: string): string => s.toUpperCase()

// Never use Function type
type Handler = (event: MouseEvent) => void  // CORRECT
type Handler = Function  // WRONG
```

## Import Types

Always use `import type` for type-only imports:
```ts
import type { User } from '@/types/user'
import { createUser } from '@/api/users'
```

## The `satisfies` Operator

```ts
// Use satisfies to validate shape while keeping inferred literal types
const config = {
  theme: 'dark',
  lang: 'en',
} satisfies Partial<AppConfig>
// config.theme is 'dark', not string
```
```

---

### `.claude/skills/tanstack-query-patterns/SKILL.md`

```md
---
name: tanstack-query-patterns
description: Use when writing useQuery, useSuspenseQuery, queryClient, queryKey, 
             prefetching, cache configuration, or any data fetching with TanStack Query.
             Triggers on "fetch", "useQuery", "queryKey", "cache", "stale", "server state".
---

# TanStack Query Patterns

## QueryKey Factory — Central, Always

Never write raw queryKey arrays inline. All keys live in one factory per domain:

```ts
// src/queries/keys/user.keys.ts
export const userKeys = {
  all: ['users'] as const,
  lists: () => [...userKeys.all, 'list'] as const,
  list: (filters: UserFilters) => [...userKeys.lists(), filters] as const,
  details: () => [...userKeys.all, 'detail'] as const,
  detail: (id: string) => [...userKeys.details(), id] as const,
}

// Usage
useQuery({ queryKey: userKeys.detail(userId), ... })
queryClient.invalidateQueries({ queryKey: userKeys.lists() })
```

## Query Function Structure

```ts
// src/queries/user.queries.ts
import { userKeys } from './keys/user.keys'
import { getUser, getUsers } from '@/api/user.api'
import type { UserFilters } from '@/types/user'

export const userQueries = {
  detail: (id: string) => ({
    queryKey: userKeys.detail(id),
    queryFn: () => getUser(id),
    staleTime: 5 * 60 * 1000,  // 5 min — define per query, not globally
  }),

  list: (filters: UserFilters) => ({
    queryKey: userKeys.list(filters),
    queryFn: () => getUsers(filters),
    staleTime: 60 * 1000,  // 1 min — lists go stale faster
  }),
}

// In component — clean, no duplication
const { data } = useQuery(userQueries.detail(userId))
```

## Stale Time Rules

| Data type | Stale time |
|---|---|
| User profile | 5 minutes |
| List / collection | 1 minute |
| Config / static | 30 minutes |
| Real-time critical | 0 |

Never use the global default staleTime of 0 — it causes waterfalls. Set per query.

## Suspense — Prefer useSuspenseQuery

```tsx
// CORRECT — works with React Suspense boundaries
const { data } = useSuspenseQuery(userQueries.detail(userId))
// data is always defined here — no null check needed

// Use useQuery only when you need isLoading/isError handling in-component
```

## Loading and Error States

```tsx
// With Suspense — handle at boundary, not in component
<Suspense fallback={<UserCardSkeleton />}>
  <ErrorBoundary fallback={<ErrorCard />}>
    <UserCard userId={userId} />
  </ErrorBoundary>
</Suspense>

// UserCard itself is clean:
function UserCard({ userId }: UserCardProps) {
  const { data } = useSuspenseQuery(userQueries.detail(userId))
  return <div>{data.name}</div>
}
```

## Select — Transform at Query Level

```ts
const { data: userName } = useQuery({
  ...userQueries.detail(userId),
  select: (user) => user.name,  // transform, don't do it in component
})
```

## Prefetching

```ts
// In loaders or on hover — not in render
await queryClient.prefetchQuery(userQueries.detail(userId))
```

## Invalidation Strategy

```ts
// After mutation — invalidate the right scope
// Too broad: invalidates everything
queryClient.invalidateQueries({ queryKey: userKeys.all })

// Too narrow: misses related lists
queryClient.invalidateQueries({ queryKey: userKeys.detail(id) })

// CORRECT: invalidate lists (they contain the updated item) + the detail
queryClient.invalidateQueries({ queryKey: userKeys.lists() })
queryClient.invalidateQueries({ queryKey: userKeys.detail(id) })
```

## What Goes in TanStack Query vs Zustand

| Server / async data | TanStack Query |
|---|---|
| UI state (sidebar open, selected tab) | Zustand |
| Form state | React Hook Form |
| Derived from server data | TanStack Query `select` |
| User session / auth | TanStack Query (with long staleTime) |

**Never copy server data into Zustand.** It creates two sources of truth.
```

---

### `.claude/skills/tanstack-mutations/SKILL.md`

```md
---
name: tanstack-mutations
description: Use when writing useMutation, form submissions, create/update/delete operations,
             optimistic updates, or any data writing operation.
             Triggers on "mutation", "submit", "create", "update", "delete", "optimistic".
---

# TanStack Mutations

## Basic Structure — Always

```ts
// src/mutations/user.mutations.ts
import { useMutation, useQueryClient } from '@tanstack/react-query'
import { userKeys } from '@/queries/keys/user.keys'
import { updateUser } from '@/api/user.api'
import type { UpdateUserInput } from '@/types/user'

export function useUpdateUser() {
  const queryClient = useQueryClient()

  return useMutation({
    mutationFn: (input: UpdateUserInput) => updateUser(input),

    onSuccess: (data, variables) => {
      // 1. Update the cache directly with the response
      queryClient.setQueryData(userKeys.detail(variables.id), data)
      // 2. Invalidate related lists
      queryClient.invalidateQueries({ queryKey: userKeys.lists() })
    },

    onError: (error, variables, context) => {
      // Rollback optimistic update if applicable
      // Log error to observability
    },
  })
}
```

## Optimistic Updates — When to Use

Use when the operation is low-risk and latency matters to UX (toggle, reorder, like):

```ts
export function useToggleFavorite() {
  const queryClient = useQueryClient()

  return useMutation({
    mutationFn: (id: string) => toggleFavorite(id),

    onMutate: async (id) => {
      // 1. Cancel in-flight queries
      await queryClient.cancelQueries({ queryKey: itemKeys.detail(id) })

      // 2. Snapshot current value
      const previous = queryClient.getQueryData(itemKeys.detail(id))

      // 3. Optimistically update
      queryClient.setQueryData(itemKeys.detail(id), (old: Item) => ({
        ...old,
        isFavorite: !old.isFavorite,
      }))

      // 4. Return snapshot for rollback
      return { previous }
    },

    onError: (error, id, context) => {
      // Rollback
      queryClient.setQueryData(itemKeys.detail(id), context?.previous)
    },

    onSettled: (data, error, id) => {
      // Always refetch after settle to ensure consistency
      queryClient.invalidateQueries({ queryKey: itemKeys.detail(id) })
    },
  })
}
```

## Error Handling

```ts
// In the mutation hook — handle at mutation level
onError: (error: ApiError) => {
  toast.error(error.message ?? 'Something went wrong')
  reportError(error)  // observability
},

// In the component — only handle UI-specific error state
const { mutate, isPending, isError } = useUpdateUser()
```

## Loading State in UI

```tsx
// CORRECT — use isPending, not isLoading (deprecated)
<Button disabled={isPending} onClick={() => mutate(input)}>
  {isPending ? 'Saving...' : 'Save'}
</Button>
```

## Mutation Colocation

Mutations live in `src/mutations/<domain>.mutations.ts` — not inside components.

One mutation = one exported hook = one file concern.

## Rules

- **Never** call API functions directly in `onClick` handlers — always `useMutation`
- **Always** invalidate or update cache on success — never leave stale data
- **Always** handle `onError` — never silently fail
- **Never** do optimistic updates on destructive operations (delete) — too risky
- **`isPending`** not `isLoading` — TanStack Query v5
```

---

### `.claude/skills/zustand-store-patterns/SKILL.md`

```md
---
name: zustand-store-patterns
description: Use when creating, modifying, or accessing Zustand stores. 
             Triggers on "store", "zustand", "global state", "useStore", 
             "persist", "slice", "state management".
---

# Zustand Store Patterns

## What Goes in Zustand

Zustand is for **UI state and client-side app state only.**

| Put in Zustand | Do NOT put in Zustand |
|---|---|
| Sidebar open/closed | Server data (use TanStack Query) |
| Selected tab / active view | Form values (use React Hook Form) |
| Toast notifications queue | User profile data |
| Modal open state + payload | API responses |
| Theme preference | Anything async |
| Wizard step progress | |
| Filters/sort that aren't in URL | |

## Slice Pattern — One File Per Domain

Never put everything in one store. Slice by domain:

```ts
// src/store/slices/ui.slice.ts
import type { StateCreator } from 'zustand'

export interface UiSlice {
  sidebarOpen: boolean
  setSidebarOpen: (open: boolean) => void
  toggleSidebar: () => void
}

export const createUiSlice: StateCreator<UiSlice> = (set) => ({
  sidebarOpen: true,
  setSidebarOpen: (open) => set({ sidebarOpen: open }),
  toggleSidebar: () => set((state) => ({ sidebarOpen: !state.sidebarOpen })),
})
```

## Root Store — Combine Slices

```ts
// src/store/index.ts
import { create } from 'zustand'
import { devtools, persist } from 'zustand/middleware'
import { createUiSlice, type UiSlice } from './slices/ui.slice'
import { createModalSlice, type ModalSlice } from './slices/modal.slice'

type AppStore = UiSlice & ModalSlice

export const useAppStore = create<AppStore>()(
  devtools(
    persist(
      (...args) => ({
        ...createUiSlice(...args),
        ...createModalSlice(...args),
      }),
      {
        name: 'app-store',
        partialize: (state) => ({
          sidebarOpen: state.sidebarOpen,  // only persist what needs to survive refresh
        }),
      }
    ),
    { name: 'AppStore' }  // DevTools label
  )
)
```

## Selectors — Always Select, Never Spread

```ts
// CORRECT — granular selector prevents unnecessary re-renders
const sidebarOpen = useAppStore((state) => state.sidebarOpen)
const toggleSidebar = useAppStore((state) => state.toggleSidebar)

// WRONG — subscribes to entire store, re-renders on any change
const store = useAppStore()
const { sidebarOpen, toggleSidebar } = useAppStore()  // also wrong
```

## Colocated Selectors

Export selectors alongside the slice:

```ts
// ui.slice.ts — add at bottom
export const uiSelectors = {
  sidebarOpen: (state: AppStore) => state.sidebarOpen,
}

// Usage
const sidebarOpen = useAppStore(uiSelectors.sidebarOpen)
```

## Immutability — Always Return New Objects

```ts
// CORRECT
set((state) => ({
  items: [...state.items, newItem],
}))

// WRONG — mutating state directly
set((state) => {
  state.items.push(newItem)  // never
})
```

## DevTools

Always wrap with `devtools()` — named, so Redux DevTools shows the store name clearly.

## Rules

- **Never** put async operations in Zustand actions — async lives in TanStack Query
- **Never** access the store outside of React components or other hooks (no store.getState() in components)
- **Always** use selectors — never destructure the whole store
- **Persist only what survives a page refresh** — don't persist derived or temporary state
- **One slice per domain** — never one massive store file
```

---

### `.claude/skills/form-patterns/SKILL.md`

```md
---
name: form-patterns
description: Use when building forms, inputs, validation, form submission, 
             or working with React Hook Form, Zod schemas, or Shadcn form components.
             Triggers on "form", "input", "validation", "submit", "field", "schema".
---

# Form Patterns

## The Stack

Every form uses this exact trio:
- **React Hook Form** — form state, validation trigger, submission
- **Zod** — schema definition and validation rules  
- **Shadcn Form components** — UI (`Form`, `FormField`, `FormItem`, `FormLabel`, `FormControl`, `FormMessage`)

Never mix. Never use uncontrolled inputs with manual validation. Never use state for form values.

## Input Length — 31 Character Limit

**All text inputs have a maximum of 31 visible characters width.**

This is enforced via a CSS class — not inline styles, not arbitrary values:

```css
/* index.css */
.input-standard {
  width: 100%;
  max-width: 31ch;
}
```

```tsx
/* Usage — always apply this class to FormControl inputs */
<Input className="input-standard" {...field} />
```

Every input in every form must be the same length. No exceptions unless explicitly designed otherwise. This creates visual rhythm and prevents layout chaos.

## Schema First — Always Define Zod Schema First

```ts
// src/schemas/user.schema.ts
import { z } from 'zod'

export const createUserSchema = z.object({
  name: z.string()
    .min(2, 'Name must be at least 2 characters')
    .max(50, 'Name must be under 50 characters')
    .trim(),
  email: z.string()
    .email('Please enter a valid email'),
  role: z.enum(['admin', 'editor', 'viewer'], {
    errorMap: () => ({ message: 'Please select a valid role' }),
  }),
})

export type CreateUserInput = z.infer<typeof createUserSchema>
```

## Form Component Structure

```tsx
// src/components/UserForm/index.tsx
import { useForm } from 'react-hook-form'
import { zodResolver } from '@hookform/resolvers/zod'
import { Form, FormControl, FormField, FormItem, FormLabel, FormMessage } from '@/components/ui/form'
import { Input } from '@/components/ui/input'
import { Button } from '@/components/ui/button'
import { createUserSchema, type CreateUserInput } from '@/schemas/user.schema'
import { useCreateUser } from '@/mutations/user.mutations'

export function UserForm() {
  const { mutate: createUser, isPending } = useCreateUser()

  const form = useForm<CreateUserInput>({
    resolver: zodResolver(createUserSchema),
    defaultValues: {
      name: '',
      email: '',
      role: 'viewer',
    },
  })

  function handleSubmit(data: CreateUserInput) {
    createUser(data, {
      onSuccess: () => form.reset(),
    })
  }

  return (
    <Form {...form}>
      <form onSubmit={form.handleSubmit(handleSubmit)} className="space-y-4">
        <FormField
          control={form.control}
          name="name"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Name</FormLabel>
              <FormControl>
                <Input
                  className="input-standard"
                  placeholder="Enter name"
                  {...field}
                />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />
        {/* repeat for other fields */}
        <Button type="submit" disabled={isPending}>
          {isPending ? 'Creating...' : 'Create User'}
        </Button>
      </form>
    </Form>
  )
}
```

## Rules

- **Schema file is separate** from the component — `src/schemas/<domain>.schema.ts`
- **`defaultValues` always defined** — never let fields be undefined on mount
- **`form.handleSubmit()`** always used — never `onSubmit` directly on `<form>`
- **`FormMessage`** always present under every field — never skip error display
- **`isPending` disables submit button** — always prevent double-submit
- **Never use `register()`** directly when using Shadcn Form — always `FormField` + `render`
- **Zod error messages are user-facing** — write them in plain English in the schema
- **All inputs same width** — `input-standard` class, always

## Select Inputs

```tsx
<FormField
  control={form.control}
  name="role"
  render={({ field }) => (
    <FormItem>
      <FormLabel>Role</FormLabel>
      <Select onValueChange={field.onChange} defaultValue={field.value}>
        <FormControl>
          <SelectTrigger className="input-standard">
            <SelectValue placeholder="Select role" />
          </SelectTrigger>
        </FormControl>
        <SelectContent>
          <SelectItem value="admin">Admin</SelectItem>
          <SelectItem value="editor">Editor</SelectItem>
          <SelectItem value="viewer">Viewer</SelectItem>
        </SelectContent>
      </Select>
      <FormMessage />
    </FormItem>
  )}
/>
```
```

---

### `.claude/skills/routing-patterns/SKILL.md`

```md
---
name: routing-patterns
description: Use when adding routes, creating pages, defining navigation, 
             using Link, navigate, useParams, useSearch, or any routing concern.
             Triggers on "route", "page", "navigate", "router", "URL", "path", "link".
---

# Routing Patterns

## Routes Are Defined in Constants — Always

**Never hardcode a path string anywhere except the constants file.**

```ts
// src/constants/routes.ts  ← single source of truth
export const ROUTES = {
  HOME: '/',
  
  AUTH: {
    LOGIN: '/login',
    REGISTER: '/register',
    FORGOT_PASSWORD: '/forgot-password',
  },

  DASHBOARD: {
    ROOT: '/dashboard',
    OVERVIEW: '/dashboard/overview',
    SETTINGS: '/dashboard/settings',
  },

  USERS: {
    LIST: '/users',
    DETAIL: (id: string) => `/users/${id}`,
    EDIT: (id: string) => `/users/${id}/edit`,
  },
} as const
```

Usage:
```tsx
// CORRECT
<Link to={ROUTES.USERS.DETAIL(userId)}>View User</Link>
navigate({ to: ROUTES.DASHBOARD.ROOT })

// WRONG — hardcoded path
<Link to="/users/123">View User</Link>
navigate({ to: '/dashboard' })
```

## Router Definition — router.tsx

All routes are registered in `src/router.tsx`. This is the only place routes are composed.

```tsx
// src/router.tsx
import { createRouter, createRootRoute, createRoute } from '@tanstack/react-router'
import { RootLayout } from '@/layouts/RootLayout'
import { DashboardPage } from '@/pages/Dashboard'
import { UsersListPage } from '@/pages/UsersList'
import { UserDetailPage } from '@/pages/UserDetail'

const rootRoute = createRootRoute({
  component: RootLayout,
})

const dashboardRoute = createRoute({
  getParentRoute: () => rootRoute,
  path: ROUTES.DASHBOARD.ROOT,
  component: DashboardPage,
})

const usersRoute = createRoute({
  getParentRoute: () => rootRoute,
  path: ROUTES.USERS.LIST,
  component: UsersListPage,
})

const userDetailRoute = createRoute({
  getParentRoute: () => usersRoute,
  path: '$id',
  component: UserDetailPage,
})

const routeTree = rootRoute.addChildren([
  dashboardRoute,
  usersRoute.addChildren([userDetailRoute]),
])

export const router = createRouter({ routeTree })
```

## Adding a New Route — Exact Steps

1. Add the path to `src/constants/routes.ts`
2. Create the page component in `src/pages/<PageName>/`
3. Register in `src/router.tsx`
4. Never skip step 1 — the constant must exist before the route

## Search Params — Always Typed

```ts
// Define the schema
const usersSearchSchema = z.object({
  page: z.number().default(1),
  search: z.string().optional(),
  role: z.enum(['admin', 'editor', 'viewer']).optional(),
})

const usersRoute = createRoute({
  ...
  validateSearch: usersSearchSchema,
})

// In component — fully typed
const { page, search } = useSearch({ from: usersRoute.id })
```

## Navigation

```tsx
import { Link, useNavigate } from '@tanstack/react-router'
import { ROUTES } from '@/constants/routes'

// Declarative
<Link to={ROUTES.USERS.DETAIL(id)}>View</Link>

// Programmatic
const navigate = useNavigate()
navigate({ to: ROUTES.DASHBOARD.ROOT })

// With search params
navigate({ to: ROUTES.USERS.LIST, search: { page: 2 } })
```

## Rules

- **Every path string lives in `ROUTES` constant** — no exceptions
- **`router.tsx` is the only place routes are composed** — no dynamic route registration elsewhere
- **Search params always have Zod schema** — never `as any` or untyped
- **Never use `window.location`** — always TanStack Router navigation
- **Route params always typed** — use `useParams({ from: routeId })`
```

---

### `.claude/skills/constants-registry/SKILL.md`

```md
---
name: constants-registry
description: Use when defining constants, configuration values, enums, magic numbers,
             string literals used in multiple places, or any hardcoded value.
             Triggers on "constant", "config", "enum", "magic number", "hardcoded".
---

# Constants and Registry

## The Rule

**If a value appears more than once, or has meaning beyond its literal value — it's a constant.**

No magic numbers. No magic strings. No inline configuration.

## File Structure

```
src/constants/
├── routes.ts         ← all route paths (see routing-patterns skill)
├── query-keys.ts     ← if not using key factories (prefer factories)
├── api.ts            ← API endpoints, base URLs, timeouts
├── ui.ts             ← UI constants (animation duration, breakpoints, limits)
├── validation.ts     ← validation limits, regex patterns
└── index.ts          ← barrel export
```

## Naming Convention

```ts
// SCREAMING_SNAKE_CASE for all constants
export const MAX_FILE_SIZE = 5 * 1024 * 1024  // 5MB — always add unit comment

export const API_ENDPOINTS = {
  USERS: '/api/users',
  USER_DETAIL: (id: string) => `/api/users/${id}`,
} as const

export const UI_CONSTANTS = {
  INPUT_MAX_WIDTH_CHARS: 31,       // the 31-char input width rule
  SIDEBAR_WIDTH: 256,
  TOAST_DURATION_MS: 3000,
  ANIMATION_DURATION_MS: 200,
  TABLE_DEFAULT_PAGE_SIZE: 20,
} as const

export const VALIDATION = {
  NAME_MIN: 2,
  NAME_MAX: 50,
  PASSWORD_MIN: 8,
  EMAIL_REGEX: /^[^\s@]+@[^\s@]+\.[^\s@]+$/,
} as const
```

## Always `as const`

```ts
// CORRECT — values are literal types, not widened
const STATUS = { ACTIVE: 'active', INACTIVE: 'inactive' } as const
type Status = typeof STATUS[keyof typeof STATUS]  // 'active' | 'inactive'

// WRONG — values are string, loses type safety
const STATUS = { ACTIVE: 'active', INACTIVE: 'inactive' }
```

## What Belongs Here

| Yes | No |
|---|---|
| API endpoint strings | Component-local one-off values |
| Timeout / limit numbers | Test fixtures |
| Regex patterns | Computed values (those are derived) |
| Route paths | Environment variables (those go in env config) |
| UI size tokens | Tailwind classes |
| Default values used across components | |

## Barrel Export

```ts
// constants/index.ts
export * from './routes'
export * from './api'
export * from './ui'
export * from './validation'
```

## Usage

```ts
import { UI_CONSTANTS, VALIDATION } from '@/constants'

// CORRECT
<Input maxLength={VALIDATION.NAME_MAX} />

// WRONG
<Input maxLength={50} />
```
```

---

### `.claude/skills/error-handling/SKILL.md`

```md
---
name: error-handling
description: Use when adding error boundaries, try-catch, error states, API error handling,
             error display, or any failure scenario handling.
             Triggers on "error", "catch", "boundary", "fallback", "failure", "exception".
---

# Error Handling

## Error Hierarchy

```
App Level ErrorBoundary (catches everything)
  └── Route Level ErrorBoundary (catches page errors)
        └── Feature Level ErrorBoundary (catches component subtrees)
              └── Inline error states (form errors, mutation errors)
```

## Error Boundary Placement

```tsx
// Root — catches catastrophic failures
// src/main.tsx or src/App.tsx
<ErrorBoundary fallback={<AppErrorPage />}>
  <App />
</ErrorBoundary>

// Route level — each page gets its own boundary
<Suspense fallback={<PageSkeleton />}>
  <ErrorBoundary fallback={<RouteErrorPage />}>
    <Outlet />
  </ErrorBoundary>
</Suspense>

// Feature level — for independently failable sections
<ErrorBoundary fallback={<WidgetErrorCard />}>
  <RevenueWidget />
</ErrorBoundary>
```

## API Error Type

Define one error type for all API errors:

```ts
// src/types/errors.ts
export interface ApiError {
  message: string
  code: string
  statusCode: number
  details?: Record<string, string[]>  // field-level validation errors
}

export function isApiError(error: unknown): error is ApiError {
  return (
    typeof error === 'object' &&
    error !== null &&
    'message' in error &&
    'statusCode' in error
  )
}
```

## Handling in Mutations

```ts
onError: (error: unknown) => {
  if (isApiError(error)) {
    if (error.statusCode === 422 && error.details) {
      // Field-level errors — set on form
      Object.entries(error.details).forEach(([field, messages]) => {
        form.setError(field as keyof FormData, { message: messages[0] })
      })
    } else {
      toast.error(error.message)
    }
  } else {
    toast.error('An unexpected error occurred')
  }
  reportError(error)  // always report
}
```

## Never Swallow Errors

```ts
// WRONG — silent failure
try {
  await doSomething()
} catch (e) {}

// WRONG — log and ignore
try {
  await doSomething()
} catch (e) {
  console.error(e)
}

// CORRECT — handle or report
try {
  await doSomething()
} catch (error) {
  reportError(error)
  throw error  // or handle appropriately
}
```

## User-Facing Error Messages

- Never show technical error messages to users
- Never show stack traces
- Always show actionable messages: "Couldn't save changes. Try again." not "500 Internal Server Error"
- Error messages live in constants or the Zod schema — not inline strings

## Rules

- **Every async operation has error handling** — no fire and forget
- **Every route has an error boundary** — no unhandled route errors
- **Every mutation has `onError`** — no silent failures
- **`reportError()` called on every caught error** — for observability
- **Never rethrow raw errors to UI** — always transform to user-friendly message
```

---

### `.claude/skills/page-anatomy/SKILL.md`

```md
---
name: page-anatomy
description: Use when creating a new page, adding a route component, building a view,
             or structuring the top-level component for any route.
             Triggers on "page", "view", "screen", "add page", "new page", "route component".
---

# Page Anatomy

## File Structure

```
src/pages/<PageName>/
├── index.tsx         ← the page component
├── types.ts          ← page-specific types (if any)
└── index.ts          ← barrel export
```

## Page Component Structure — Exact Order

```tsx
// src/pages/UsersListPage/index.tsx

// 1. Imports — external, then internal, then types
import { Suspense } from 'react'
import { ErrorBoundary } from 'react-error-boundary'
import { useSearch } from '@tanstack/react-router'
import { UsersTable } from '@/components/UsersTable'
import { PageHeader } from '@/components/PageHeader'
import { TableSkeleton } from '@/components/skeletons/TableSkeleton'
import { RouteErrorCard } from '@/components/errors/RouteErrorCard'

// 2. Page component — named export
export function UsersListPage() {
  // 3. Route hooks first (params, search)
  const { page, search } = useSearch({ from: '/users' })

  // 4. Store state (zustand) — if needed
  
  // 5. NO data fetching here — data fetching belongs in child components
  //    Page is the boundary and layout owner, not the data owner

  // 6. Derived state

  // 7. Handlers — if page-level handlers exist

  // 8. Render — always Suspense + ErrorBoundary wrapping data components
  return (
    <div className="flex flex-col gap-6 p-6">
      <PageHeader
        title="Users"
        description="Manage your team members"
      />

      <Suspense fallback={<TableSkeleton />}>
        <ErrorBoundary fallback={<RouteErrorCard />}>
          <UsersTable page={page} search={search} />
        </ErrorBoundary>
      </Suspense>
    </div>
  )
}
```

## Data Fetching — Never at Page Level

The page component does NOT call `useQuery`. It sets up boundaries and layout. Data fetching happens in the components that need the data.

```tsx
// WRONG — page fetches data
function UsersListPage() {
  const { data } = useQuery(userQueries.list())  // NO
  return <UsersTable users={data} />
}

// CORRECT — data-owning component fetches its own data
function UsersListPage() {
  return (
    <Suspense fallback={<TableSkeleton />}>
      <UsersTable />  // UsersTable internally calls useQuery
    </Suspense>
  )
}
```

## Page Layout Primitives

```tsx
// Standard page wrapper — consistent padding and spacing
<div className="flex flex-col gap-6 p-6">

// Two-column layout
<div className="grid grid-cols-1 lg:grid-cols-[280px_1fr] gap-6">

// Full-height page
<div className="flex flex-col h-full overflow-hidden">
  <PageHeader ... />
  <div className="flex-1 overflow-y-auto">  {/* scrollable content */}
    ...
  </div>
</div>
```

## Scroll Behavior

Scrollable areas are always the content container, never the page itself:

```tsx
// Page does not scroll
<div className="flex flex-col h-full overflow-hidden">
  <PageHeader />                        {/* fixed height */}
  <div className="flex-1 overflow-y-auto">  {/* this scrolls, invisibly */}
    <Content />
  </div>
</div>
```

## Rules

- **Pages own layout and boundaries** — not data
- **Every page has Suspense + ErrorBoundary** around async children
- **No `useQuery` at page level** — push data fetching down
- **Pages are thin** — if logic grows, extract to a hook `useUsersPage.ts`
- **Named export only** — no default export
- **Consistent padding** — `p-6` standard, never varies per page without design reason
```

---

### `.claude/skills/api-layer/SKILL.md`

```md
---
name: api-layer
description: Use when writing API functions, HTTP requests, axios configuration,
             API client setup, or connecting to backend endpoints.
             Triggers on "api", "endpoint", "fetch", "axios", "http", "request", "client".
---

# API Layer

## Structure

```
src/api/
├── client.ts           ← axios instance, interceptors, base config
├── user.api.ts         ← user domain API functions
├── product.api.ts      ← product domain API functions
└── index.ts            ← barrel export
```

## API Client — One Instance

```ts
// src/api/client.ts
import axios from 'axios'
import type { AxiosError, InternalAxiosRequestConfig } from 'axios'
import { ROUTES } from '@/constants/routes'

export const apiClient = axios.create({
  baseURL: process.env.NEXT_PUBLIC_API_URL,
  timeout: 10_000,
  headers: {
    'Content-Type': 'application/json',
  },
})

// Request interceptor — attach auth token
apiClient.interceptors.request.use((config: InternalAxiosRequestConfig) => {
  const token = getAuthToken()  // from your auth store/cookie
  if (token) {
    config.headers.Authorization = `Bearer ${token}`
  }
  return config
})

// Response interceptor — normalize errors
apiClient.interceptors.response.use(
  (response) => response.data,  // unwrap data
  (error: AxiosError<ApiError>) => {
    const apiError: ApiError = {
      message: error.response?.data?.message ?? 'Something went wrong',
      code: error.response?.data?.code ?? 'UNKNOWN',
      statusCode: error.response?.status ?? 500,
      details: error.response?.data?.details,
    }

    // Redirect on auth failure
    if (apiError.statusCode === 401) {
      navigate({ to: ROUTES.AUTH.LOGIN })
    }

    return Promise.reject(apiError)
  }
)
```

## API Functions — Pure, Typed

```ts
// src/api/user.api.ts
import { apiClient } from './client'
import type { User, CreateUserInput, UpdateUserInput, UserFilters } from '@/types/user'

export async function getUsers(filters: UserFilters): Promise<User[]> {
  return apiClient.get('/users', { params: filters })
}

export async function getUser(id: string): Promise<User> {
  return apiClient.get(`/users/${id}`)
}

export async function createUser(input: CreateUserInput): Promise<User> {
  return apiClient.post('/users', input)
}

export async function updateUser(input: UpdateUserInput): Promise<User> {
  return apiClient.patch(`/users/${input.id}`, input)
}

export async function deleteUser(id: string): Promise<void> {
  return apiClient.delete(`/users/${id}`)
}
```

## Rules

- **One `apiClient` instance** — never create `axios.create()` again elsewhere
- **API functions are pure** — they only take input, call the API, return typed output
- **No business logic in API functions** — no transforms, no UI concerns
- **All endpoint strings come from constants** — never hardcode in api functions (or at minimum, never duplicate)
- **All functions are typed** — input type, return type, both explicit
- **Response is unwrapped in the interceptor** — API functions return `T`, not `AxiosResponse<T>`
- **Errors are normalized in the interceptor** — components and mutations receive `ApiError`, not raw axios errors
- **Never call `apiClient` directly in components** — always through TanStack Query/Mutation hooks
```

---

### `.claude/skills/add-feature-checklist/SKILL.md`

```md
---
name: add-feature-checklist
description: Use when implementing a new feature, adding a major capability, or 
             building something that spans multiple files and layers.
             Triggers on "add feature", "implement", "new feature", "build", "create feature".
---

# Add Feature Checklist

When adding any feature, follow this exact sequence. Do not skip steps. Do not reorder.

## The Sequence

### Step 1 — Types First
```
src/types/<domain>.ts
```
- Define all interfaces for this feature
- Infer from Zod schemas where applicable
- No `any`

### Step 2 — Zod Schemas
```
src/schemas/<domain>.schema.ts
```
- Define validation schemas for all inputs
- Write user-friendly error messages
- Infer TypeScript types from schemas

### Step 3 — Constants
```
src/constants/
```
- Add route paths to `routes.ts`
- Add any new string/number constants
- Add to barrel export

### Step 4 — API Functions
```
src/api/<domain>.api.ts
```
- Pure typed functions
- Use `apiClient`
- Follow api-layer patterns

### Step 5 — Query Keys
```
src/queries/keys/<domain>.keys.ts
```
- Define key factory
- All keys for this domain

### Step 6 — Queries
```
src/queries/<domain>.queries.ts
```
- Query objects using key factory
- Stale times defined per query

### Step 7 — Mutations
```
src/mutations/<domain>.mutations.ts
```
- One hook per mutation operation
- Cache invalidation defined
- Error handling defined

### Step 8 — Components
```
src/components/<Feature>/
```
- Follow add-component patterns
- No API calls directly
- Types in types.ts

### Step 9 — Page
```
src/pages/<FeaturePage>/
```
- Register route in router.tsx
- Add route constant
- Suspense + ErrorBoundary wrapping

### Step 10 — Register Route
```
src/router.tsx
src/constants/routes.ts
```
- Add to ROUTES constant first
- Then register in router

### Step 11 — Tests
```
src/<domain>/__tests__/
```
- Unit tests for utils and hooks
- Component tests for critical interactions

## Checklist Before Marking Done

- [ ] All types defined, no `any`
- [ ] Zod schema with user-friendly errors
- [ ] Route constant defined (if new page)
- [ ] Route registered in router.tsx
- [ ] API functions typed and pure
- [ ] QueryKey factory updated
- [ ] Stale times set per query
- [ ] Mutations have onError handlers
- [ ] Mutations invalidate correct cache keys
- [ ] Components follow naming + structure patterns
- [ ] All inputs have `input-standard` class
- [ ] Scrollable areas have invisible scrollbars
- [ ] Page has Suspense + ErrorBoundary
- [ ] No hardcoded strings (use constants)
- [ ] TypeScript compiles with no errors
- [ ] Prettier passes
- [ ] ESLint passes

## What Claude Must Never Do

- Skip the types step and write components first
- Hardcode route strings anywhere except `routes.ts`
- Put `useQuery` in page-level components
- Create a second axios instance
- Leave `onError` undefined on mutations
- Use `any` to get unstuck
```

---

### `.claude/skills/performance-patterns/SKILL.md`

```md
---
name: performance-patterns
description: Use when optimizing renders, using useMemo, useCallback, React.memo,
             lazy loading, code splitting, or any performance concern.
             Triggers on "performance", "memo", "re-render", "optimize", "lazy", "split".
---

# Performance Patterns

## The Golden Rule

**Measure before optimizing.** Don't add `useMemo`/`useCallback` speculatively. Profile first with React DevTools.

## When to Use useMemo

```tsx
// YES — expensive computation
const sortedData = useMemo(
  () => data.sort((a, b) => b.createdAt - a.createdAt),
  [data]
)

// YES — referential stability for object passed to memoized child
const config = useMemo(() => ({ theme, lang }), [theme, lang])

// NO — cheap computation, not worth the overhead
const fullName = useMemo(() => `${first} ${last}`, [first, last])
// Just do: const fullName = `${first} ${last}`

// NO — dependency on everything
const result = useMemo(() => compute(a, b, c, d), [a, b, c, d])
// If everything changes, memo saves nothing
```

## When to Use useCallback

```tsx
// YES — passed to memoized child component
const handleSelect = useCallback((id: string) => {
  setSelected(id)
}, [])  // stable, no deps

// YES — used as useEffect dependency
useEffect(() => {
  fetchData()
}, [fetchData])  // fetchData should be useCallback

// NO — used only in this component's render
const handleClick = () => { setOpen(true) }  // no useCallback needed
```

## React.memo

```tsx
// YES — component re-renders often with same props
export const UserRow = React.memo(function UserRow({ user, onSelect }: UserRowProps) {
  return ...
})

// When to NOT use React.memo
// - Component is cheap to render
// - Props change every render anyway
// - Not a list item
```

## Code Splitting — Lazy Load Heavy Pages

```tsx
// src/router.tsx — lazy load page components
import { lazy, Suspense } from 'react'

const DashboardPage = lazy(() => import('@/pages/DashboardPage').then(m => ({ default: m.DashboardPage })))
const AnalyticsPage = lazy(() => import('@/pages/AnalyticsPage').then(m => ({ default: m.AnalyticsPage })))

// Always wrap lazy components in Suspense
<Suspense fallback={<PageSkeleton />}>
  <DashboardPage />
</Suspense>
```

## List Rendering

```tsx
// Always use stable keys — never index
{users.map((user) => (
  <UserRow key={user.id} user={user} />  // CORRECT
))}

{users.map((user, index) => (
  <UserRow key={index} ... />  // WRONG — causes reconciliation bugs
))}
```

## State Colocation

Keep state as close to where it's used as possible. Don't lift state unnecessarily.

```tsx
// WRONG — dialog state in parent for no reason
function ParentPage() {
  const [dialogOpen, setDialogOpen] = useState(false)
  return <UserDialog open={dialogOpen} onClose={() => setDialogOpen(false)} />
}

// CORRECT — dialog manages its own open state
function UserDialog() {
  const [open, setOpen] = useState(false)
  return ...
}
```

## TanStack Query — Already Handles Caching

Do not cache TanStack Query results in `useMemo` or state. The query cache is the cache.

```tsx
// WRONG
const { data } = useQuery(userQueries.list())
const memoizedData = useMemo(() => data, [data])  // pointless

// CORRECT
const { data } = useQuery(userQueries.list())
// data is already cached by TanStack Query
```
```

---

### `.claude/skills/accessibility-standards/SKILL.md`

```md
---
name: accessibility-standards
description: Use when writing any UI component, interactive element, form, modal,
             navigation, or anything a user interacts with.
             Triggers on "button", "form", "modal", "menu", "dialog", "interactive", "click", "a11y".
---

# Accessibility Standards

## The Non-Negotiables

Every component ships accessible. Accessibility is not a post-launch checkbox.

## Semantic HTML First

```tsx
// CORRECT — semantic elements
<button onClick={handleSave}>Save</button>
<nav aria-label="Main navigation">
<main>
<header>
<article>
<section aria-labelledby="section-title">

// WRONG — div soup
<div onClick={handleSave}>Save</div>  // not keyboard accessible
<div className="nav">
<div className="main">
```

## Interactive Elements

```tsx
// Every interactive element must be keyboard accessible
// Buttons for actions
<button type="button" onClick={handleAction}>Action</button>

// Links for navigation  
<Link to={ROUTES.DASHBOARD.ROOT}>Dashboard</Link>

// Never: <div onClick={...}> without role and keyboard handling
```

## ARIA Labels

```tsx
// Icon-only buttons MUST have aria-label
<button aria-label="Close dialog" onClick={handleClose}>
  <XIcon className="h-4 w-4" />
</button>

// Form inputs — use FormLabel (Shadcn) or explicit htmlFor
<label htmlFor="email-input">Email</label>
<input id="email-input" type="email" />

// Images — always alt text
<img src={avatar} alt={`${user.name} profile photo`} />
// Decorative images
<img src={decoration} alt="" aria-hidden="true" />
```

## Focus Management

```tsx
// Modals — focus must move into modal on open, return on close
// Shadcn Dialog handles this automatically — use it

// Avoid focus traps outside of modals/drawers
// After dynamic content loads, move focus to the new content if user action caused it
```

## Color Contrast

- Text on background: minimum 4.5:1 ratio
- Large text (18px+): minimum 3:1
- Never rely on color alone to convey meaning
- Always pair color with an icon or text label

```tsx
// WRONG — error shown only in red color
<span className="text-destructive">{errorMessage}</span>

// CORRECT — icon + color + text
<span className="flex items-center gap-1 text-destructive">
  <AlertCircleIcon className="h-4 w-4" aria-hidden="true" />
  {errorMessage}
</span>
```

## Loading States

```tsx
// Screen reader announcement for loading
<div aria-live="polite" aria-busy={isLoading}>
  {isLoading ? <Skeleton /> : <Content />}
</div>
```

## Rules

- **Every button has visible focus ring** — never `outline-none` without a `focus-visible:` replacement
- **Every form input has a label** — visible or `sr-only`, never unlabeled
- **Every icon-only button has `aria-label`**
- **Every image has `alt` text** — empty string for decorative
- **No `div` or `span` with `onClick`** unless `role` and keyboard handling added
- **Shadcn components are accessible by default** — don't break their ARIA by overriding
```

---

Now I need your answer on **Next.js vs TanStack Router** before I write:
- `page-anatomy` (already written above assumes TanStack — verify this is right)
- The `settings.json` hooks file
- The `/add-feature` and `/add-page` command files

Also — do you want me to write the `settings.json` with the hooks (branch protection, prettier, tsc) and the commands folder next?
