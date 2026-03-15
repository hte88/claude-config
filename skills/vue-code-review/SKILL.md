---
name: vue-code-review
description: Strict code review for Vue.js 3, Nuxt 3, and TypeScript projects. Use this skill when the user asks for code review, correction, improvement, or shares Vue/Nuxt/TypeScript code for analysis. Enforces strict TypeScript typing (no 'any'), Vue.js best practices, and provides complete corrected code with concrete examples.
---

# Vue Code Review

Review Vue.js 3, Nuxt 3, and TypeScript code with strict standards enforcement and complete corrected output.

## Core Principles

**Always provide complete corrected code** - Never return partial code or just the modified sections. Return the entire file with all corrections applied.

**Challenge and correct systematically** - Point out errors, anti-patterns, and suboptimal approaches directly. Provide better solutions with concrete code examples.

**Use examples for every explanation** - Illustrate every concept or correction with real code snippets.

## Stack and Standards

### Technology Stack
- Vue.js 3 (Composition API)
- Nuxt 3
- Nuxt UI
- TailwindCSS
- VueUse
- Vee-Validate
- Zod
- TypeScript (strict mode)
- Vitest

### TypeScript Rules (Strict Enforcement)

**Rule 1: `any` is FORBIDDEN**

Replace `any` with proper typing. No exceptions.

Bad:
```typescript
const data: any = fetchData()
function process(input: any) { }
```

Good:
```typescript
const data: User = fetchData()
function process(input: unknown) {
  if (isUser(input)) {
    // Type guard narrows unknown to User
  }
}
```

**Rule 2: Leverage type inference**

Don't explicitly type when TypeScript can infer correctly.

Bad:
```typescript
const count: number = ref(0)
const name: string = 'John'
```

Good:
```typescript
const count = ref(0) // inferred as Ref<number>
const name = 'John'  // inferred as string
```

**Rule 3: Prefer `type` over `interface`**

Use `type` by default. Reserve `interface` only when you need declaration merging or `extends`.

Bad:
```typescript
interface User {
  id: string
  name: string
}
```

Good:
```typescript
type User = {
  id: string
  name: string
}
```

**Rule 4: Use `as const` for static arrays**

If an array is hardcoded and never mutated, add `as const` for narrower type inference.

Bad:
```typescript
const roles = ['admin', 'editor', 'viewer']
// type: string[]
```

Good:
```typescript
const roles = ['admin', 'editor', 'viewer'] as const
// type: readonly ['admin', 'editor', 'viewer']
```

**Rule 5: Use type guards for narrowing**

When working with `unknown`, use type guards to narrow types safely.

```typescript
function isUser(value: unknown): value is User {
  return typeof value === 'object' &&
         value !== null &&
         'id' in value
}
```

### Vue.js Rules

**Rule 1: Script setup ordering**

Follow this order in `<script setup>` **sauf si l'ordre d'exécution impose autrement** (ex: un ref utilisé par un composable doit être déclaré avant) :

```typescript
<script setup lang="ts">
// 1. Types / Interfaces
// 2. Props / Emits / Model
// 3. Injections (useRoute, useI18n, stores, composables...)
// 4. Refs / Reactive state
// 5. Computed
// 6. Watch
// 7. Functions
// 8. Lifecycle hooks (onMounted, etc.)
// 9. defineExpose
</script>
```

Si une dépendance d'exécution crée un conflit avec cet ordre, privilégier le bon fonctionnement du code et commenter pourquoi l'ordre dévie.

**Rule 2: Prefer `function` over `const` for function declarations**

Bad:
```typescript
const handleClick = () => {
  // logic
}
```

Good:
```typescript
function handleClick() {
  // logic
}
```

**Rule 2: Use Composition API consistently**

Avoid Options API patterns. Use `<script setup>` with Composition API.

**Rule 3: Destructure props and use `defineProps` with TypeScript**

```typescript
type Props = {
  userId: string
  isActive?: boolean
}

const props = defineProps<Props>()
```

**Rule 4: Use computed for derived state**

Bad:
```typescript
const fullName = ref('')
watch([firstName, lastName], () => {
  fullName.value = `${firstName.value} ${lastName.value}`
})
```

Good:
```typescript
const fullName = computed(() =>
  `${firstName.value} ${lastName.value}`
)
```

## Review Process

When reviewing code, follow this structure:

1. **Identify all issues** - List TypeScript violations, Vue.js anti-patterns, performance problems, maintainability issues
2. **Provide complete corrected code** - Return the entire file with all corrections applied
3. **Comment critical changes** - Add brief inline comments explaining non-obvious fixes
4. **Suggest architectural improvements** - If the approach can be fundamentally improved, explain the better pattern

## Common Anti-Patterns to Flag

### TypeScript Anti-Patterns

**Using `any` anywhere**
```typescript
// WRONG - immediately flag and fix
const result: any = await fetch()

// CORRECT
type ApiResponse = { data: User[] }
const result: ApiResponse = await fetch()
```

**Not handling `unknown` properly**
```typescript
// WRONG
function process(input: unknown) {
  return input.id // Error: Property 'id' does not exist
}

// CORRECT
function process(input: unknown) {
  if (isValidInput(input)) {
    return input.id
  }
  throw new Error('Invalid input')
}
```

### Vue.js Anti-Patterns

**Mutating props directly**
```typescript
// WRONG
const props = defineProps<{ count: number }>()
props.count++ // Error

// CORRECT
const localCount = ref(props.count)
localCount.value++
```

**Not using composables for reusable logic**
```typescript
// WRONG - logic duplicated across components
const loading = ref(false)
async function fetchData() {
  loading.value = true
  // fetch logic
  loading.value = false
}

// CORRECT - extract to composable
// composables/useFetch.ts
export function useFetch<T>(url: string) {
  const loading = ref(false)
  const data = ref<T>()

  async function fetch() {
    loading.value = true
    try {
      data.value = await $fetch<T>(url)
    } finally {
      loading.value = false
    }
  }

  return { loading, data, fetch }
}
```

**Excessive watchers instead of computed**
```typescript
// WRONG
const total = ref(0)
watch([price, quantity], () => {
  total.value = price.value * quantity.value
})

// CORRECT
const total = computed(() => price.value * quantity.value)
```

### Performance Anti-Patterns

**Unnecessary re-renders from reactive overhead**
```typescript
// WRONG - entire object is reactive
const state = reactive({
  largeArray: [],
  config: { /* complex object */ }
})

// CORRECT - only make reactive what needs reactivity
const largeArray = ref([])
const config = readonly({ /* complex object */ })
```

**Not using `shallowRef` for large data structures**
```typescript
// WRONG for large immutable data
const bigData = ref(hugeDataSet)

// CORRECT
const bigData = shallowRef(hugeDataSet)
```

## Output Format

When providing corrected code:

1. Return the COMPLETE file, not excerpts
2. Add comments only for non-obvious changes
3. Maintain original formatting style
4. Keep the response concise - let the code speak

Example review output:

```typescript
// components/UserProfile.vue
<script setup lang="ts">
type Props = {
  userId: string
}

const props = defineProps<Props>()

// Changed from 'any' to proper typing
type UserData = {
  id: string
  name: string
  email: string
}

// Changed from const to function
async function loadUser() {
  const { data } = await useFetch<UserData>(`/api/users/${props.userId}`)
  return data.value
}

const user = await loadUser()
</script>
```

## References

For detailed patterns and architectural guidance, see `references/patterns.md`.
