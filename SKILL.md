---
name: unplugin-vue-router-skill
description: File-based typed routing for Vue 3 with automatic route generation from file structure. Use when: (1) Setting up or configuring unplugin-vue-router in a Vue project, (2) Understanding file-based routing structure and naming conventions, (3) Implementing type-safe routes with TypeScript, (4) Extending or modifying routes with custom configuration, (5) Troubleshooting routing issues or optimizing route organization
version: 1.0.0
license: MIT
---

# Unplugin Vue Router

## Overview

Enable automatic file-based routing with full TypeScript type safety for Vue 3 projects. This plugin generates routes from your file structure and provides typed navigation, eliminating the need to manually maintain route arrays.

## Core Concepts

### What It Does

- **Automatic route generation**: Creates routes from `src/pages` directory structure
- **Type safety**: Generates `typed-router.d.ts` for full TypeScript support
- **Build-time processing**: Routes generated during build, no runtime overhead
- **Framework agnostic**: Works with Vite, Rollup, Webpack, and esbuild
- **Vue Router 4+**: Requires Vue Router >= 4.4.0

### When to Use

- Setting up a new Vue 3 project with routing
- Migrating from manual route configuration to file-based routing
- Adding TypeScript type safety to existing Vue Router setup
- Configuring custom route patterns or layouts
- Troubleshooting route generation or type issues

## Quick Start

### Installation

```bash
npm i -D unplugin-vue-router
```

### Basic Configuration

Add the plugin **before** Vue plugin in your Vite config:

```ts
// vite.config.ts
import VueRouter from 'unplugin-vue-router/vite'
import Vue from '@vitejs/plugin-vue'

export default defineConfig({
  plugins: [
    VueRouter({
      /* options */
    }),
    Vue(), // ⚠️ Vue must be placed after VueRouter()
  ],
})
```

### TypeScript Setup

1. Run dev server to generate types: `npm run dev`
2. Add generated types to `tsconfig.json`:

```json
{
  "include": ["./typed-router.d.ts"],
  "compilerOptions": {
    "moduleResolution": "Bundler"
  }
}
```

3. Add client types to `env.d.ts` (or create one):

```ts
/// <reference types="vite/client" />
/// <reference types="unplugin-vue-router/client" />
```

4. Import and use generated routes:

```ts
import { createRouter, createWebHistory } from 'vue-router'
import { routes } from 'vue-router/auto-routes'

const router = createRouter({
  history: createWebHistory(),
  routes,
})
```

## File-Based Routing

### Directory Structure

Routes are generated from `src/pages` by default. File path determines route path:

```
src/pages/
├── index.vue          → /
├── about.vue          → /about
├── users/
│   ├── index.vue      → /users
│   └── [id].vue       → /users/:id
└── blog/
    └── [slug].vue     → /blog/:slug
```

### Key Rules

#### Index Routes

- `index.vue` (lowercase) generates empty path
- `src/pages/index.vue` → `/`
- `src/pages/users/index.vue` → `/users`

#### Dynamic Routes

- Wrap params in brackets: `[param]`
- `src/pages/users/[id].vue` → `/users/:id`
- Multiple params: `product_[skuId]_[desc].vue` → `/product_:skuId_:desc`
- Optional params: `[[param]]` → `:param?`
- Repeatable params: `[slugs]+.vue` → `/:slugs+`
- Optional + repeatable: `[[slugs]]+.vue` → `/:slugs*`

#### Catch-All Routes

- Prepend `...` to param name
- `src/pages/[...path].vue` → `/:path(.*)`
- `src/pages/articles/[...path].vue` → `/articles/:path(.*)`

#### Nested Routes

- Create `.vue` file alongside folder with same name
- Child route renders in parent's `<RouterView>`

```
src/pages/
├── users.vue          → /users (parent)
└── users/
    └── index.vue      → renders in users.vue <RouterView>
```

#### Named Routes

- All generated routes have a `name` property
- Auto-generated from file path
- Customize with `getRouteName()` option

## Advanced Features

### Route Groups

Organize files without affecting URLs using `(group)` syntax:

```
src/pages/
├── (admin)/
│   ├── dashboard.vue   → /dashboard
│   └── settings.vue    → /settings
└── (user)/
    ├── profile.vue      → /profile
    └── orders.vue      → /orders
```

### Nested Routes Without Layouts

Use `.` to create URL nesting without UI nesting:

```
src/pages/
├── users.vue
└── users.create.vue    → /users/create (not nested)
```

### Named Views

Use `@` suffix for named views:

```
src/pages/
├── index.vue           → default view
└── index@aux.vue      → auxiliary view
```

## Configuration Options

See [Configuration Reference](references/configuration.md) for complete options.

Key options:

- `routesFolder`: Array of folders to scan (default: `['src/pages']`)
- `extensions`: File extensions to consider (default: `['.vue']`)
- `dts`: Path for generated types (default: `'./typed-router.d.ts'`)
- `importMode`: Route import mode - `'sync'` | `'async'`
- `pathParser.dotNesting`: Parse `file.[id]` as `file/:id` (default: `true`)
- `extendRoute`: Function to modify individual routes
- `beforeWriteFiles`: Function to modify routes before writing

### Multiple Route Folders

```ts
VueRouter({
  routesFolder: [
    'src/pages',
    {
      src: 'src/admin/routes',
      path: 'admin/',
    },
    {
      src: 'src/docs',
      path: 'docs/:lang/',
    },
  ],
})
```

## Type Safety

### Typed Navigation

Import routes from `vue-router/auto-routes` for full type safety:

```ts
import { routes } from 'vue-router/auto-routes'

// Fully typed - autocomplete and validation
router.push({ name: 'users-id', params: { id: '123' } })
```

### Typed useRoute()

Configure Volar plugin for automatic typing in page components:

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

Then in page components:

```ts
// Automatically typed without manual casting
const route = useRoute()
route.params.id // ✅ Type-safe access
```

### Manual Route Typing

For dynamically added routes, augment `RouteNamedMap`:

```ts
declare module 'vue-router/auto-routes' {
  export interface RouteNamedMap {
    'custom-route': RouteRecordInfo<
      'custom-route',
      '/custom-path',
      { id: ParamValue<true> },
      { id: ParamValue<false> },
      never
    >
  }
}
```

## SSR Support

Mark `vue-router` as `noExternal` in development:

```ts
export default defineConfig(({ mode }) => ({
  ssr: {
    noExternal: mode === 'development' ? ['vue-router'] : [],
  },
  plugins: [VueRouter(), Vue()],
}))
```

## Extending Routes

### Route Block

Add `<route>` block in SFC to customize routes:

```vue
<template>
  <div>Page</div>
</template>

<script setup lang="ts">
definePage({
  meta: {
    requiresAuth: true,
  },
})
</script>
```

### Extend Routes Option

Modify routes via `extendRoute()`:

```ts
VueRouter({
  async extendRoute(route) {
    if (route.name === 'index') {
      route.meta!.title = 'Home'
    }
    return route
  },
})
```

## Troubleshooting

### Types Not Generated

- Ensure dev server is running: `npm run dev`
- Check `typed-router.d.ts` is in `tsconfig.json` include
- Verify `moduleResolution` is set to `"Bundler"`

### Route Not Found

- Check file naming follows conventions (lowercase `index.vue`)
- Verify file is in `routesFolder` directory
- Ensure file extension matches `extensions` config

### Type Errors

- Run dev server to regenerate types after file changes
- Clear build cache: delete `node_modules/.vite` folder
- Verify `unplugin-vue-router/client` is referenced

### Routes Not Updating

- Restart dev server
- Check for caching in build tools
- Verify plugin order (VueRouter before Vue)

## Resources

For detailed configuration and advanced usage, see:

- [Configuration Reference](reference/configuration.md)
- [Route Patterns](reference/route-patterns.md)
- [TypeScript Guide](reference/typescript.md)

## Notes

- Experimental APIs (like Data Loaders) may change
- Official documentation: https://uvr.esm.is
- Requires Vue Router >= 4.4.0
- Works best with TypeScript strict mode
