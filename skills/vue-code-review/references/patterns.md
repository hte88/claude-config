# Advanced Vue.js & TypeScript Patterns

This reference contains advanced patterns, composable structures, and architectural guidance for Vue 3 + Nuxt 3 + TypeScript projects.

## Composable Patterns

### Generic Data Fetching Composable

```typescript
// composables/useApi.ts
export function useApi<T>(url: MaybeRefOrGetter<string>) {
  const data = ref<T>()
  const error = ref<Error>()
  const loading = ref(false)

  async function execute() {
    loading.value = true
    error.value = undefined

    try {
      data.value = await $fetch<T>(toValue(url))
    } catch (e) {
      error.value = e instanceof Error ? e : new Error(String(e))
    } finally {
      loading.value = false
    }
  }

  return { data, error, loading, execute }
}

// Usage
const { data, loading, execute } = useApi<User[]>('/api/users')
await execute()
```

### Pagination Composable

```typescript
// composables/usePagination.ts
interface PaginationOptions {
  initialPage?: number
  pageSize?: number
}

interface PaginatedData<T> {
  items: T[]
  total: number
  page: number
  pageCount: number
}

export function usePagination<T>(
  fetchFn: (page: number, size: number) => Promise<PaginatedData<T>>,
  options: PaginationOptions = {}
) {
  const page = ref(options.initialPage ?? 1)
  const pageSize = ref(options.pageSize ?? 10)
  const items = ref<T[]>([])
  const total = ref(0)
  const loading = ref(false)

  const pageCount = computed(() => Math.ceil(total.value / pageSize.value))

  async function load() {
    loading.value = true
    try {
      const result = await fetchFn(page.value, pageSize.value)
      items.value = result.items
      total.value = result.total
    } finally {
      loading.value = false
    }
  }

  function nextPage() {
    if (page.value < pageCount.value) {
      page.value++
      load()
    }
  }

  function prevPage() {
    if (page.value > 1) {
      page.value--
      load()
    }
  }

  return {
    items,
    page,
    pageSize,
    total,
    pageCount,
    loading,
    nextPage,
    prevPage,
    load
  }
}
```

### Form State Management with Vee-Validate + Zod

```typescript
// composables/useUserForm.ts
import { z } from 'zod'
import { toTypedSchema } from '@vee-validate/zod'
import { useForm } from 'vee-validate'

const userSchema = z.object({
  email: z.string().email('Email invalide'),
  password: z.string().min(8, 'Minimum 8 caractères'),
  age: z.number().min(18, 'Âge minimum 18 ans')
})

type UserFormData = z.infer<typeof userSchema>

export function useUserForm() {
  const { handleSubmit, errors, values, resetForm } = useForm({
    validationSchema: toTypedSchema(userSchema),
    initialValues: {
      email: '',
      password: '',
      age: 18
    }
  })

  const onSubmit = handleSubmit(async (values: UserFormData) => {
    await $fetch('/api/users', {
      method: 'POST',
      body: values
    })
  })

  return {
    onSubmit,
    errors,
    values,
    resetForm
  }
}
```

## TypeScript Patterns

### Discriminated Unions for API Responses

```typescript
type ApiResponse<T> =
  | { success: true; data: T }
  | { success: false; error: string }

async function fetchUser(id: string): Promise<ApiResponse<User>> {
  try {
    const data = await $fetch<User>(`/api/users/${id}`)
    return { success: true, data }
  } catch (error) {
    return {
      success: false,
      error: error instanceof Error ? error.message : 'Unknown error'
    }
  }
}

// Usage with type narrowing
const response = await fetchUser('123')
if (response.success) {
  console.log(response.data.name) // TypeScript knows 'data' exists
} else {
  console.error(response.error) // TypeScript knows 'error' exists
}
```

### Generic Component Props with Constraints

```typescript
// components/DataTable.vue
interface DataTableProps<T extends Record<string, unknown>> {
  items: T[]
  columns: Array<keyof T>
  onRowClick?: (item: T) => void
}

const props = defineProps<DataTableProps<User>>()

// TypeScript ensures columns are valid User keys
// props.columns can only contain 'id', 'name', 'email', etc.
```

### Type Guards for Runtime Validation

```typescript
interface User {
  id: string
  name: string
  email: string
}

function isUser(value: unknown): value is User {
  if (typeof value !== 'object' || value === null) {
    return false
  }

  const obj = value as Record<string, unknown>

  return (
    typeof obj.id === 'string' &&
    typeof obj.name === 'string' &&
    typeof obj.email === 'string'
  )
}

// Usage
async function loadUser(data: unknown) {
  if (!isUser(data)) {
    throw new Error('Invalid user data')
  }

  // data is now typed as User
  console.log(data.email)
}
```

## Nuxt-Specific Patterns

### Server API Routes with Type Safety

```typescript
// server/api/users/[id].get.ts
export default defineEventHandler(async (event) => {
  const id = getRouterParam(event, 'id')

  if (!id) {
    throw createError({
      statusCode: 400,
      message: 'User ID required'
    })
  }

  const user = await db.users.findUnique({ where: { id } })

  if (!user) {
    throw createError({
      statusCode: 404,
      message: 'User not found'
    })
  }

  return user
})

// Client-side usage with automatic typing
const { data: user } = await useFetch(`/api/users/${userId}`)
// user is automatically typed based on the return type
```

### Middleware with Type Safety

```typescript
// middleware/auth.ts
export default defineNuxtRouteMiddleware((to, from) => {
  const authStore = useAuthStore()

  if (!authStore.isAuthenticated) {
    return navigateTo({
      path: '/login',
      query: { redirect: to.fullPath }
    })
  }
})
```

### Plugin Pattern for Global State

```typescript
// plugins/auth.ts
export default defineNuxtPlugin(() => {
  const user = useState<User | null>('user', () => null)
  const isAuthenticated = computed(() => user.value !== null)

  async function login(email: string, password: string) {
    const response = await $fetch<{ user: User }>('/api/auth/login', {
      method: 'POST',
      body: { email, password }
    })
    user.value = response.user
  }

  function logout() {
    user.value = null
    navigateTo('/login')
  }

  return {
    provide: {
      auth: {
        user: readonly(user),
        isAuthenticated,
        login,
        logout
      }
    }
  }
})

// Usage in components
const { $auth } = useNuxtApp()
console.log($auth.isAuthenticated)
```

## State Management Patterns

### Pinia Store with TypeScript

```typescript
// stores/user.ts
import { defineStore } from 'pinia'

interface UserState {
  currentUser: User | null
  users: User[]
  loading: boolean
}

export const useUserStore = defineStore('user', () => {
  const currentUser = ref<User | null>(null)
  const users = ref<User[]>([])
  const loading = ref(false)

  const isAuthenticated = computed(() => currentUser.value !== null)

  async function fetchUsers() {
    loading.value = true
    try {
      users.value = await $fetch<User[]>('/api/users')
    } finally {
      loading.value = false
    }
  }

  function setCurrentUser(user: User | null) {
    currentUser.value = user
  }

  return {
    currentUser,
    users,
    loading,
    isAuthenticated,
    fetchUsers,
    setCurrentUser
  }
})
```

## Performance Patterns

### Virtual Scrolling for Large Lists

```typescript
// composables/useVirtualList.ts
interface VirtualListOptions {
  itemHeight: number
  containerHeight: number
}

export function useVirtualList<T>(
  items: Ref<T[]>,
  options: VirtualListOptions
) {
  const scrollTop = ref(0)

  const visibleItems = computed(() => {
    const { itemHeight, containerHeight } = options
    const startIndex = Math.floor(scrollTop.value / itemHeight)
    const endIndex = Math.ceil(
      (scrollTop.value + containerHeight) / itemHeight
    )

    return items.value.slice(startIndex, endIndex).map((item, i) => ({
      item,
      index: startIndex + i,
      top: (startIndex + i) * itemHeight
    }))
  })

  function onScroll(event: Event) {
    scrollTop.value = (event.target as HTMLElement).scrollTop
  }

  const totalHeight = computed(() =>
    items.value.length * options.itemHeight
  )

  return {
    visibleItems,
    totalHeight,
    onScroll
  }
}
```

### Debounced Search

```typescript
// composables/useSearch.ts
export function useSearch<T>(
  searchFn: (query: string) => Promise<T[]>,
  delay = 300
) {
  const query = ref('')
  const results = ref<T[]>([])
  const loading = ref(false)

  const debouncedSearch = useDebounceFn(async (q: string) => {
    if (!q.trim()) {
      results.value = []
      return
    }

    loading.value = true
    try {
      results.value = await searchFn(q)
    } finally {
      loading.value = false
    }
  }, delay)

  watch(query, (newQuery) => {
    debouncedSearch(newQuery)
  })

  return {
    query,
    results,
    loading
  }
}
```

## Testing Patterns

### Component Testing with Vitest

```typescript
// components/UserCard.test.ts
import { describe, it, expect } from 'vitest'
import { mount } from '@vue/test-utils'
import UserCard from './UserCard.vue'

describe('UserCard', () => {
  it('renders user information', () => {
    const user = {
      id: '1',
      name: 'John Doe',
      email: 'john@example.com'
    }

    const wrapper = mount(UserCard, {
      props: { user }
    })

    expect(wrapper.text()).toContain('John Doe')
    expect(wrapper.text()).toContain('john@example.com')
  })

  it('emits delete event on button click', async () => {
    const user = {
      id: '1',
      name: 'John Doe',
      email: 'john@example.com'
    }

    const wrapper = mount(UserCard, {
      props: { user }
    })

    await wrapper.find('[data-test="delete-btn"]').trigger('click')

    expect(wrapper.emitted('delete')).toBeTruthy()
    expect(wrapper.emitted('delete')?.[0]).toEqual([user.id])
  })
})
```

### Composable Testing

```typescript
// composables/useCounter.test.ts
import { describe, it, expect } from 'vitest'
import { useCounter } from './useCounter'

describe('useCounter', () => {
  it('increments counter', () => {
    const { count, increment } = useCounter()

    expect(count.value).toBe(0)
    increment()
    expect(count.value).toBe(1)
  })

  it('decrements counter', () => {
    const { count, decrement } = useCounter(5)

    expect(count.value).toBe(5)
    decrement()
    expect(count.value).toBe(4)
  })
})
```
