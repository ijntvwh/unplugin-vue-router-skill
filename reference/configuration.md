# Configuration Reference

Complete reference for all unplugin-vue-router configuration options.

## Overview

```ts
import VueRouter from 'unplugin-vue-router/vite'

VueRouter({
  // Configuration options
})
```

## Options

### routesFolder

Type: `RoutesFolderOption[] | string[]`

Default: `['src/pages']`

Define which folders to scan for route files.

**String format:**

```ts
VueRouter({
  routesFolder: ['src/pages', 'src/admin'],
})
```

**Object format:**

```ts
VueRouter({
  routesFolder: [
    'src/pages', // simple string
    {
      src: 'src/admin/routes',
      path: 'admin/', // URL prefix
      extensions: ['.vue'], // override global extensions
      filePatterns: ['**/*'], // include patterns
      exclude: [], // exclude patterns
    },
  ],
})
```

**Path rules:**

- Always has trailing slash, never leading slash
- Can contain route parameters
- Can omit trailing slash for custom prefixes

### extensions

Type: `string[]`

Default: `['.vue']`

File extensions to consider as route pages.

```ts
VueRouter({
  extensions: ['.vue', '.md'], // Support markdown pages
})
```

Can be overridden per folder in `routesFolder`.

### filePatterns

Type: `string[] | ((patterns: string[]) => string[])`

Default: `['**/*']`

Glob patterns for files to include.

```ts
VueRouter({
  filePatterns: ['**/*.vue', '**/*.md'],
})
```

Can be a function to filter default patterns.

### exclude

Type: `(path: string) => boolean | string[]`

Default: `[]`

Files or directories to exclude from scanning.

```ts
VueRouter({
  exclude: [
    '**/_*.vue', // Exclude underscore prefixed files
    '**/components/', // Exclude component directories
  ],
})
```

Or as a function:

```ts
VueRouter({
  exclude: path => path.includes('.test.vue'),
})
```

### dts

Type: `string`

Default: `'./typed-router.d.ts'`

Path for generated TypeScript type definitions.

```ts
VueRouter({
  dts: './src/types/router.d.ts',
})
```

Generated file is created when dev or build server runs.

### getRouteName

Type: `(routeNode: TreeNode) => string`

Default: Function that generates names from file paths

Custom function to generate route names.

```ts
VueRouter({
  getRouteName: routeNode => {
    // Generate custom names
    return routeNode.filePath.replace('src/pages/', '').replace('.vue', '').replace(/\//g, '-').toLowerCase()
  },
})
```

**TreeNode type:**

```ts
interface TreeNode {
  filePath: string
  name: string
  children: TreeNode[]
  // ... other properties
}
```

### routeBlockLang

Type: `'json5' | 'yaml' | 'json' | 'yml'`

Default: `'json5'`

Language for `<route>` custom block parsing.

```ts
VueRouter({
  routeBlockLang: 'yaml', // Use YAML for route blocks
})
```

### importMode

Type: `'sync' | 'async' | ((name: string) => 'sync' | 'async')`

Default: `'async'`

How to import route components.

**String:**

```ts
VueRouter({
  importMode: 'sync', // All routes synchronous
})
```

**Function:**

```ts
VueRouter({
  importMode: routePath => {
    // Async for nested routes, sync for others
    return routePath.includes('/layout') ? 'sync' : 'async'
  },
})
```

### root

Type: `string`

Default: `process.cwd()`

Base directory for resolving paths.

```ts
VueRouter({
  root: '/path/to/project',
})
```

### pathParser

Type: `{ dotNesting: boolean }`

Default: `{ dotNesting: true }`

Options for parsing file paths to route paths.

#### dotNesting

Type: `boolean`

Default: `true`

Whether `file.[id]` should be parsed as `file/:id`.

```ts
VueRouter({
  pathParser: {
    dotNesting: false, // Disable dot nesting
  },
})
```

### extendRoute

Type: `(route: RouteRecordRaw, parent: RouteRecordRaw | null) => void | Promise<void> | RouteRecordRaw`

Function to modify individual routes during generation.

```ts
VueRouter({
  async extendRoute(route, parent) {
    // Add meta information
    if (route.name === 'users-id') {
      route.meta = {
        ...route.meta,
        requiresAuth: true,
        roles: ['admin'],
      }
    }

    // Add redirect
    if (route.name === 'home' && route.path === '/') {
      route.redirect = '/dashboard'
    }

    // Add aliases
    if (route.name === 'profile') {
      route.alias = ['/me', '/account']
    }

    return route
  },
})
```

### beforeWriteFiles

Type: `(rootRoute: RouteRecordRaw) => void | Promise<RouteRecordRaw>`

Function to modify all routes before writing to file.

```ts
VueRouter({
  async beforeWriteFiles(rootRoute) {
    // Add global middleware
    rootRoute.beforeEach = (to, from, next) => {
      // Global guard logic
      next()
    }

    // Apply transformations to entire route tree
    const addAuthGuard = route => {
      if (route.path.startsWith('/admin')) {
        route.meta = {
          ...route.meta,
          requiresAuth: true,
        }
      }
      if (route.children) {
        route.children.forEach(addAuthGuard)
      }
    }

    rootRoute.children?.forEach(addAuthGuard)

    return rootRoute
  },
})
```

## Full Example

```ts
import VueRouter from 'unplugin-vue-router/vite'
import Vue from '@vitejs/plugin-vue'

export default defineConfig({
  plugins: [
    VueRouter({
      // Multiple route folders with prefixes
      routesFolder: [
        {
          src: 'src/pages',
          path: '',
        },
        {
          src: 'src/admin',
          path: 'admin/',
          extensions: ['.vue'],
        },
      ],

      // File configuration
      extensions: ['.vue'],
      filePatterns: ['**/*'],
      exclude: ['**/_*.vue'],

      // Type generation
      dts: './src/types/typed-router.d.ts',

      // Route naming
      getRouteName: routeNode => {
        return routeNode.filePath.replace('src/', '').replace('.vue', '').replace(/\//g, '-')
      },

      // Route blocks
      routeBlockLang: 'json5',

      // Import mode
      importMode: 'async',

      // Path parsing
      pathParser: {
        dotNesting: true,
      },

      // Route extension
      async extendRoute(route) {
        // Add page titles from filename
        if (route.meta && !route.meta.title) {
          const title = String(route.name)
            .split('-')
            .map(word => word.charAt(0).toUpperCase() + word.slice(1))
            .join(' ')
          route.meta.title = title
        }
        return route
      },
    }),
    Vue(),
  ],
})
```

## Framework-Specific Configuration

### Vite

```ts
// vite.config.ts
import VueRouter from 'unplugin-vue-router/vite'
import Vue from '@vitejs/plugin-vue'

export default defineConfig({
  plugins: [VueRouter(), Vue()],
})
```

### Rollup

```js
// rollup.config.js
import VueRouter from 'unplugin-vue-router/rollup'

export default {
  plugins: [VueRouter()],
}
```

### Webpack

```js
// webpack.config.js
module.exports = {
  plugins: [require('unplugin-vue-router/webpack')()],
}
```

### Vue CLI

```js
// vue.config.js
module.exports = {
  configureWebpack: {
    plugins: [require('unplugin-vue-router/webpack')()],
  },
}
```

### esbuild

```js
// esbuild.config.js
import { build } from 'esbuild'
import VueRouter from 'unplugin-vue-router/esbuild'

build({
  plugins: [VueRouter()],
})
```

## SSR Configuration

For SSR frameworks, mark `vue-router` as `noExternal`:

```ts
export default defineConfig(({ mode }) => ({
  ssr: {
    noExternal: mode === 'development' ? ['vue-router'] : [],
  },
  plugins: [VueRouter(), Vue()],
}))
```
