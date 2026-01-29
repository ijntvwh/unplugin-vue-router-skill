# Route Patterns Reference

Complete guide to file-based routing patterns and conventions.

## Basic Patterns

### Static Routes

Simple file → simple path:

```
File:                    Generated Path:
about.vue               → /about
profile.vue             → /profile
dashboard.vue           → /dashboard
settings/profile.vue    → /settings/profile
admin/users/list.vue    → /admin/users/list
```

### Index Routes

`index.vue` generates empty path (similar to `index.html`):

```
File:                    Generated Path:
index.vue               → /
about/index.vue         → /about
users/index.vue         → /users
admin/index.vue         → /admin
```

**Rules:**

- Must be all lowercase `index.vue`
- Can appear at any directory level
- Creates route at parent directory path

## Dynamic Routes

### Required Parameters

Wrap param name in brackets `[param]`:

```
File:                    Generated Path:
users/[id].vue          → /users/:id
products/[sku].vue       → /products/:sku
posts/[slug].vue         → /posts/:slug
```

**Usage:**

```vue
<script setup lang="ts">
const route = useRoute()
const id = route.params.id // type: string
</script>
```

### Multiple Parameters

Multiple params in filename:

```
File:                               Generated Path:
product_[skuId]_[desc].vue          → /product_:skuId_:desc
user_[id]_[action].vue              → /user_:id_:action
blog_[year]_[month]_[slug].vue      → /blog_:year_:month_:slug
```

### Optional Parameters

Double brackets `[[param]]` for optional params:

```
File:                    Generated Path:
users/[[id]].vue         → /users/:id?
products/[[sku]].vue      → /products/:sku?
blog/[[page]].vue        → /blog/:page?
```

**Behavior:**

- Matches both `/users` and `/users/123`
- Param value is `undefined` when not provided

### Repeatable Parameters

Add `+` after closing bracket `[slugs]+`:

```
File:                    Generated Path:
articles/[slugs]+.vue    → /articles/:slugs+
tags/[ids]+.vue          → /tags/:ids+
```

**Usage:**

```ts
// Matches: /articles/a/b/c
const slugs = route.params.slugs // type: string[]
```

### Optional + Repeatable

Combine both `[[slugs]]+`:

```
File:                       Generated Path:
articles/[[slugs]]+.vue      → /articles/:slugs*
```

**Behavior:**

- Matches `/articles` (empty array)
- Matches `/articles/a` (one item)
- Matches `/articles/a/b/c` (multiple items)

## Nested Routes

### Basic Nesting

Create `.vue` file alongside folder with same name:

```
Structure:
src/pages/
├── users.vue              → Parent component
└── users/
    └── index.vue          → Child (renders in users.vue)

Generated Routes:
{
  path: '/users',
  component: () => import('src/pages/users.vue'),
  children: [
    { path: '', component: () => import('src/pages/users/index.vue') }
  ]
}
```

**users.vue:**

```vue
<template>
  <div>
    <h1>Users</h1>
    <RouterView />
    <!-- Child routes render here -->
  </div>
</template>
```

### Nested Layouts

Multiple levels of nesting:

```
Structure:
src/pages/
├── admin.vue
├── admin/
│   ├── index.vue
│   └── users/
│       ├── index.vue
│       └── [id].vue

Generated Routes:
{
  path: '/admin',
  component: () => import('src/pages/admin.vue'),
  children: [
    {
      path: '',
      component: () => import('src/pages/admin/index.vue'),
      children: [
        { path: '', component: () => import('src/pages/admin/users/index.vue') },
        { path: ':id', component: () => import('src/pages/admin/users/[id].vue') }
      ]
    }
  ]
}
```

### Dynamic Nested Parents

Parent names can also be dynamic:

```
Structure:
src/pages/
├── [userType].vue
└── [userType]/
    └── index.vue

Generated Routes:
{
  path: '/:userType',
  component: () => import('src/pages/[userType].vue'),
  children: [
    { path: '', component: () => import('src/pages/[userType]/index.vue') }
  ]
}
```

## Route Groups

### Basic Groups

Parentheses `(group)` for organization without URL impact:

```
Structure:
src/pages/
├── (admin)/
│   ├── dashboard.vue     → /dashboard
│   ├── settings.vue      → /settings
│   └── users/
│       └── index.vue     → /users
└── (public)/
    ├── about.vue         → /about
    └── contact.vue      → /contact
```

**Use cases:**

- Group routes by layout
- Organize by feature or permission
- Keep related files together

### Group Index

`(group).vue` acts as index for group:

```
Structure:
src/pages/
├── admin/
│   ├── (dashboard).vue   → /admin (acts as index)
│   └── settings.vue     → /admin/settings
```

**Equivalent to:**

```
src/pages/
├── admin/
│   ├── index.vue        → /admin
│   └── settings.vue     → /admin/settings
```

### Nested Groups

Groups can be nested:

```
Structure:
src/pages/
├── (auth)/
│   ├── (login)/
│   │   └── index.vue   → /
│   └── (register)/
│       └── index.vue    → /
```

## URL Nesting Without UI Nesting

### Dot Notation

Use `.` to create URL nesting without parent component nesting:

```
Structure:
src/pages/
├── users.vue
├── users/
│   ├── index.vue        → /users (nested in users.vue)
│   └── [id].vue        → /users/:id (nested in users.vue)
├── users.create.vue     → /users/create (NOT nested)
├── users.edit.vue       → /users/edit (NOT nested)
└── users.delete.vue     → /users/delete (NOT nested)
```

**Generated Routes:**

```js
;[
  {
    path: '/users',
    component: () => import('src/pages/users.vue'),
    children: [
      { path: '', component: () => import('src/pages/users/index.vue') },
      { path: ':id', component: () => import('src/pages/users/[id].vue') },
    ],
  },
  {
    path: '/users/create',
    component: () => import('src/pages/users.create.vue'),
  },
  {
    path: '/users/edit',
    component: () => import('src/pages/users.edit.vue'),
  },
]
```

**Use case:**
Add sibling routes without nesting in parent's `<RouterView>`.

## Named Views

### Basic Named Views

Use `@name` suffix for named views:

```
Structure:
src/pages/
├── index.vue                → default view
├── index@sidebar.vue       → named view 'sidebar'
└── index@header.vue        → named view 'header'
```

**Generated Route:**

```js
{
  path: '/',
  component: {
    default: () => import('src/pages/index.vue'),
    sidebar: () => import('src/pages/index@sidebar.vue'),
    header: () => import('src/pages/index@header.vue'),
  }
}
```

**Usage in parent:**

```vue
<template>
  <RouterView name="header" />
  <RouterView name="default" />
  <RouterView name="sidebar" />
</template>
```

### Mix Named and Default

Having both `index.vue` and `index@aux.vue`:

```
Structure:
src/pages/
├── index.vue            → default view
└── index@aux.vue       → auxiliary view

Generated Route:
{
  path: '/',
  component: {
    default: () => import('src/pages/index.vue'),
    aux: () => import('src/pages/index@aux.vue'),
  }
}
```

**Note:** You don't need to name files `index@default.vue`. Default is implicit.

## Catch-All Routes

### Basic Catch-All

Prepend `...` for catch-all:

```
File:                    Generated Path:
[...path].vue            → /:path(.*)
```

**Behavior:**

- Matches any path
- Use for 404 pages

### Scoped Catch-All

Catch-all within a specific path:

```
File:                            Generated Path:
admin/[...path].vue              → /admin/:path(.*)
docs/[...page].vue               → /docs/:page(.*)
blog/[category]/[...page].vue    → /blog/:category/:page(.*)
```

**Use case:**
Handle 404s for specific route sections.

## Complex Examples

### Multi-Level Nested with Dynamic Params

```
Structure:
src/pages/
├── blog.vue
├── blog/
│   ├── index.vue
│   ├── [slug].vue
│   └── [slug]/
│       ├── edit.vue
│       └── comments/
│           ├── index.vue
│           └── [id].vue

Generated Routes:
{
  path: '/blog',
  component: () => import('src/pages/blog.vue'),
  children: [
    {
      path: '',
      component: () => import('src/pages/blog/index.vue'),
    },
    {
      path: ':slug',
      component: () => import('src/pages/blog/[slug].vue'),
      children: [
        { path: 'edit', component: () => import('src/pages/blog/[slug]/edit.vue') },
        {
          path: 'comments',
          children: [
            { path: '', component: () => import('src/pages/blog/[slug]/comments/index.vue') },
            { path: ':id', component: () => import('src/pages/blog/[slug]/comments/[id].vue') }
          ]
        }
      ]
    }
  ]
}
```

### Admin Dashboard with Groups

```
Structure:
src/pages/
├── (dashboard)/
│   ├── index.vue            → /
│   └── stats.vue            → /stats
├── (admin)/
│   ├── index.vue            → /admin
│   ├── users.vue            → /admin/users
│   ├── users.vue            → Parent
│   ├── users/
│   │   ├── index.vue        → /admin/users (nested)
│   │   ├── [id].vue         → /admin/users/:id (nested)
│   │   ├── [id].edit.vue    → /admin/users/:id/edit (NOT nested)
│   │   └── [id].delete.vue  → /admin/users/:id/delete (NOT nested)
└── (settings)/
    ├── index.vue            → /settings
    └── account.vue          → /settings/account
```

## File Naming Rules

### Allowed Characters

- Lowercase letters: `a-z`
- Numbers: `0-9`
- Hyphens: `-`
- Underscores: `_`
- Dots: `.` (for nesting)
- Brackets: `[]` (for params)

### Forbidden Patterns

- Uppercase in `index.vue` (must be lowercase)
- Leading/trailing dots (except for nesting)
- Consecutive hyphens or underscores
- Special characters: `!@#$%^&*()+=`

### Best Practices

1. **Use kebab-case** for multi-word names: `user-profile.vue`
2. **Be descriptive**: `user-profile-settings.vue` > `settings.vue`
3. **Group logically**: Use route groups for organization
4. **Avoid deep nesting**: Max 3-4 levels preferred
5. **Consistent patterns**: Similar routes, similar structure

## Pattern Comparison

| Pattern                           | URL             | Type                  | Use Case           |
| --------------------------------- | --------------- | --------------------- | ------------------ |
| `index.vue`                       | `/`             | Static                | Home page          |
| `about.vue`                       | `/about`        | Static                | Simple page        |
| `[id].vue`                        | `/:id`          | Dynamic param         | Detail pages       |
| `[[id]].vue`                      | `/:id?`         | Optional param        | Optional filters   |
| `[slugs]+.vue`                    | `/:slugs+`      | Repeatable param      | Category chains    |
| `[[slugs]]+.vue`                  | `/:slugs*`      | Optional + repeatable | Complex navigation |
| `[...path].vue`                   | `/:path(.*)`    | Catch-all             | 404 pages          |
| `parent.vue` + `parent/child.vue` | `/parent`       | Nested                | Layout system      |
| `parent.child.vue`                | `/parent/child` | Non-nested            | Sibling routes     |
| `(group)/page.vue`                | `/page`         | Route group           | Organization       |
| `page@named.vue`                  | `/`             | Named view            | Multi-view layouts |
