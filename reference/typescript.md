# TypeScript Guide

Complete guide to type-safe routing with unplugin-vue-router.

## Overview

unplugin-vue-router generates a `typed-router.d.ts` file that provides full TypeScript support for Vue Router.

### Generated Types

When you run the dev server, the plugin generates:

- `RouteNamedMap`: All route records with full type information
- Enhanced `useRoute()`, `useRouter()`, `$route`, `$router` types
- Typed route location objects
- Typed route parameters

## Setup

### 1. Include Generated Types

Add to `tsconfig.json`:

```json
{
  "include": ["src/**/*.ts", "src/**/*.vue", "./typed-router.d.ts"],
  "compilerOptions": {
    "moduleResolution": "Bundler"
  }
}
```

### 2. Add Client Types

In `env.d.ts` (or create one):

```ts
/// <reference types="vite/client" />
/// <reference types="unplugin-vue-router/client" />
```

Or in `tsconfig.json`:

```json
{
  "compilerOptions": {
    "types": ["unplugin-vue-router/client"]
  }
}
```

### 3. Run Dev Server

Generate types by running:

```bash
npm run dev
```

The `typed-router.d.ts` file is created automatically.

## Basic Usage

### Import Generated Routes

```ts
import { routes } from 'vue-router/auto-routes'
import { createRouter, createWebHistory } from 'vue-router'

const router = createRouter({
  history: createWebHistory(),
  routes, // ✅ Fully typed
})

export default router
```

### Typed Navigation

```ts
import { useRouter } from 'vue-router'

const router = useRouter()

// ✅ Fully typed - autocomplete and validation
router.push({
  name: 'users-id',
  params: { id: '123' },
})

// ❌ Type error: missing required param
router.push({
  name: 'users-id',
})

// ❌ Type error: invalid param name
router.push({
  name: 'users-id',
  params: { userId: '123' },
})
```

### Typed Route Location

```ts
import { type RouteLocationRaw } from 'vue-router'

// ✅ Type-safe route location
const toUser: RouteLocationRaw = {
  name: 'users-id',
  params: { id: '123' },
}
```

## Typed useRoute()

### Automatic Typing (Recommended)

Enable Volar plugin for automatic typing:

**tsconfig.json:**

```json
{
  "compilerOptions": {
    "rootDir": "."
  },
  "vueCompilerOptions": {
    "plugins": ["unplugin-vue-router/volar/sfc-typed-router"]
  }
}
```

**In page components:**

```vue
<script setup lang="ts">
// ✅ Automatically typed based on current route
const route = useRoute()

// Type-safe access to params
const userId = route.params.id // type: string

// Type-safe access to meta
const title = route.meta.title // type: unknown (custom)
</script>
```

**Benefits:**

- No manual casting needed
- Full autocomplete
- Refactoring support
- Type-safe across entire application

### Manual Typing

When Volar plugin is not enabled:

```vue
<script setup lang="ts">
// Manually specify route name
const route = useRoute('/users/[id]')

const userId = route.params.id // type: string
</script>
```

### Route Name Generic

Pass route name as generic for typing:

```ts
import { useRoute } from 'vue-router'

// ✅ Includes child route typings
const route = useRoute('users-id')

// Access params
const id = route.params.id // type: string

// Access route info
const name = route.name // type: 'users-id'
const path = route.path // type: '/users/:id'
```

### Type Casting (Not Recommended)

```ts
import { type RouteLocationNormalizedLoaded } from 'vue-router'

// ❌ Not recommended - excludes child routes
const route = useRoute() as RouteLocationNormalizedLoaded<'/users/[id]'>

// ❌ Better - includes child routes
const route = useRoute<'/users/[id]'>()

// ✅ Best - automatic with Volar plugin
const route = useRoute()
```

## Route Record Types

### RouteNamedMap Interface

The generated `RouteNamedMap` contains all route records:

```ts
export interface RouteNamedMap {
  home: RouteRecordInfo<
    'home', // route name
    '/', // route path
    Record<string, never>, // raw params
    Record<string, never>, // normalized params
    never // children routes
  >
  'users-id': RouteRecordInfo<
    'users-id',
    '/users/:id',
    { id: ParamValue<true> }, // raw params (accept number/string)
    { id: ParamValue<false> }, // normalized params (strings)
    'users-id-edit' // child route name
  >
  // ... more routes
}
```

### RouteRecordInfo Type

```ts
interface RouteRecordInfo<
  TName extends RouteNamedMapKey,
  TPath extends string,
  TRawParams,
  TNormalizedParams,
  TChildren extends RouteNamedMapKey | never,
> {
  name: TName
  path: TPath
  params: TRawParams
  normalizedParams: TNormalizedParams
  children: TChildren
}
```

### ParamValue Types

```ts
type ParamValue<Raw extends boolean> = Raw extends true
  ? string | number | boolean | (string | number | boolean)[]
  : string | string[]
```

- `ParamValue<true>`: Accepts original values (strings, numbers, booleans)
- `ParamValue<false>`: Normalized to strings (as they appear in URLs)

## Manual Type Augmentation

### Adding Dynamic Routes

For routes added at runtime, augment `RouteNamedMap`:

```ts
import type { RouteNamedMap } from 'vue-router/auto-routes'

declare module 'vue-router/auto-routes' {
  import type { RouteRecordInfo, ParamValue } from 'vue-router'

  export interface RouteNamedMap {
    'custom-dynamic': RouteRecordInfo<
      'custom-dynamic',
      '/custom/:id',
      { id: ParamValue<true> },
      { id: ParamValue<false> },
      never
    >
  }
}
```

### Augmenting with Children

```ts
declare module 'vue-router/auto-routes' {
  export interface RouteNamedMap {
    parent: RouteRecordInfo<
      'parent',
      '/parent',
      Record<string, never>,
      Record<string, never>,
      'parent-child' // Child route name
    >
    'parent-child': RouteRecordInfo<
      'parent-child',
      '/parent/child',
      Record<string, never>,
      Record<string, never>,
      never
    >
  }
}
```

### \_RouteFileInfoMap Augmentation

For Volar plugin integration:

```ts
declare module 'vue-router/auto-routes' {
  export interface _RouteFileInfoMap {
    '/custom/[id].vue': {
      routes: 'custom-dynamic' | 'custom-dynamic-child'
      views: 'default'
    }
  }
}
```

## Advanced Usage

### Type-Safe Router Links

```vue
<template>
  <!-- ✅ Type-safe with name -->
  <RouterLink :to="{ name: 'users-id', params: { id: '123' } }"> User </RouterLink>

  <!-- ✅ Type-safe with path -->
  <RouterLink to="/users/123"> User </RouterLink>
</template>
```

### Type-Safe Programmatic Navigation

```ts
// ✅ Type-safe navigation
await router.push({
  name: 'users-id',
  params: { id: '123' },
})

// ✅ Type-safe replace
await router.replace({
  name: 'home',
})

// ✅ Type-safe navigation guard
router.beforeEach((to, from, next) => {
  if (to.name === 'users-id') {
    // to.params.id is typed as string
  }
  next()
})
```

### Type-S Route Guards

```ts
// ✅ Type-safe navigation guards
router.beforeEach(to => {
  if (to.name === 'admin') {
    // Check authentication
    if (!isAuthenticated()) {
      return { name: 'login' }
    }
  }
})
```

### Extract Route Types

```ts
import type { RouteNamedMap } from 'vue-router/auto-routes'

// Extract route type by name
type UserRoute = RouteNamedMap['users-id']

// Type: 'users-id'
type UserName = UserRoute['name']

// Type: '/users/:id'
type UserPath = UserRoute['path']

// Type: { id: ParamValue<true> }
type UserParams = UserRoute['params']
```

## Nuxt Integration

### Enable Typed Pages

**nuxt.config.ts:**

```ts
export default defineNuxtConfig({
  experimental: {
    typedPages: true,
  },
})
```

### Manual Volar Plugin

```ts
export default defineNuxtConfig({
  typescript: {
    tsConfig: {
      vueCompilerOptions: {
        plugins: ['unplugin-vue-router/volar/sfc-typed-router'],
      },
    },
  },
})
```

## Common Patterns

### Type-Safe Component Props

```vue
<script setup lang="ts">
interface Props {
  routeName: keyof RouteNamedMap
}

const props = defineProps<Props>()

// Type-safe route lookup
const route = useRoute(props.routeName)
</script>
```

### Type-Safe Route List

```ts
import { routes } from 'vue-router/auto-routes'

// Extract all route names
type AllRouteNames = (typeof routes)[number]['name']

// Create typed route list
const publicRoutes: AllRouteNames[] = ['home', 'about', 'contact']
```

### Type-Safe Breadcrumbs

```ts
import { useRoute } from 'vue-router'

const route = useRoute()

// Type-safe route name matching
const isUserRoute = route.name === 'users-id'
const isAdminRoute = route.name?.startsWith('admin')
```

## Troubleshooting

### Types Not Generated

**Problem:** `typed-router.d.ts` doesn't exist

**Solution:**

1. Run dev server: `npm run dev`
2. Verify plugin is configured correctly
3. Check `dts` option points to valid path

### Type Errors in Route Names

**Problem:** Route names show as red squiggles

**Solution:**

1. Regenerate types by restarting dev server
2. Clear cache: `rm -rf node_modules/.vite`
3. Verify `typed-router.d.ts` is in `tsconfig.json` include

### Missing Autocomplete

**Problem:** No autocomplete for route names

**Solution:**

1. Enable Volar plugin in `vueCompilerOptions.plugins`
2. Ensure `"moduleResolution": "Bundler"` in tsconfig
3. Restart TypeScript server in your IDE

### Incorrect Param Types

**Problem:** Params typed as `unknown`

**Solution:**

1. Ensure Volar plugin is enabled
2. Restart dev server after adding new routes
3. Check route file naming conventions

### Route Name Not Found

**Problem:** Type error: route name doesn't exist

**Solution:**

1. Verify file exists in `routesFolder`
2. Check file naming follows conventions
3. Restart dev server to regenerate types

## Best Practices

### 1. Enable Volar Plugin

Always enable `sfc-typed-router` for automatic typing:

```json
{
  "vueCompilerOptions": {
    "plugins": ["unplugin-vue-router/volar/sfc-typed-router"]
  }
}
```

### 2. Use Route Names over Paths

Prefer route names for navigation:

```ts
// ✅ Type-safe
router.push({ name: 'users-id', params: { id: '123' } })

// ❌ Not type-safe
router.push('/users/123')
```

### 3. Commit Generated Types

Commit `typed-router.d.ts` to repository:

- Easier code reviews
- Faster CI builds
- Better IDE performance

### 4. Type-Safe Route Guards

Use route name checks in guards:

```ts
router.beforeEach(to => {
  // ✅ Type-safe
  if (to.name === 'admin' || to.name?.startsWith('admin-')) {
    // Admin route logic
  }
})
```

### 5. Augment Dynamic Routes

Add types for runtime routes:

```ts
declare module 'vue-router/auto-routes' {
  export interface RouteNamedMap {
    'dynamic-route': RouteRecordInfo<...>
  }
}
```

## Type Safety Benefits

### 1. Compile-Time Error Detection

Catch route errors before runtime:

```ts
// ❌ Caught at compile time
router.push({ name: 'users-id' })
// Error: Property 'params' is missing

// ❌ Caught at compile time
router.push({ name: 'users-id', params: { userId: '123' } })
// Error: Property 'userId' does not exist
```

### 2. Autocomplete

Get route names and params as you type:

```ts
router.push({
  name: 'u|', // Shows all routes starting with 'u'
})
```

### 3. Refactoring Support

Rename routes safely with IDE refactoring:

```ts
// Rename 'users-id' to 'user-detail' everywhere
// All references update automatically
```

### 4. Documentation

Types serve as living documentation:

```ts
// Hover to see route structure
const route = useRoute('users-id')
// Shows:
// - name: 'users-id'
// - path: '/users/:id'
// - params: { id: ParamValue<true> }
```
