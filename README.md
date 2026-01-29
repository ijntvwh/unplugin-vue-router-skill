# unplugin-vue-router-skill

A skill for [unplugin-vue-router](https://github.com/posva/unplugin-vue-router) - file-based typed routing for Vue 3.

## About

This skill provides comprehensive guidance for setting up and using **unplugin-vue-router**, a plugin that enables automatic file-based routing with full TypeScript type safety for Vue 3 projects.

## What This Skill Covers

### Core Features

- ✅ **Quick Setup**: Installation and basic configuration for Vite, Rollup, Webpack, and esbuild
- ✅ **File-Based Routing**: Automatic route generation from file structure
- ✅ **Type Safety**: Full TypeScript support with typed navigation and route parameters
- ✅ **Route Patterns**: Static, dynamic, nested, optional, repeatable, and catch-all routes
- ✅ **Advanced Features**: Route groups, named views, URL nesting without UI nesting
- ✅ **Configuration**: Complete reference for all plugin options
- ✅ **SSR Support**: Server-side rendering configuration
- ✅ **Troubleshooting**: Common issues and solutions

### Documentation Sections

**Main Guide (SKILL.md)**

- Quick start and installation
- Basic file-based routing concepts
- Configuration overview
- TypeScript type safety introduction
- Common troubleshooting

**References**

- **configuration.md**: Complete configuration options with examples
- **route-patterns.md**: All route patterns and naming conventions
- **typescript.md**: Type-safe routing, Volar plugin setup, advanced typing patterns

## Installation

1. Download `unplugin-vue-router-skill.skill` from the [Releases](https://github.com/ijntvwh/unplugin-vue-router-skill) section
2. Install using Claude Desktop:
   - Open Claude Desktop settings
   - Go to Skills section
   - Click "Add Skill"
   - Select the downloaded `.skill` file

## Usage

This skill automatically triggers when:

- Setting up or configuring unplugin-vue-router in a Vue project
- Understanding file-based routing structure and naming conventions
- Implementing type-safe routes with TypeScript
- Extending or modifying routes with custom configuration
- Troubleshooting routing issues or optimizing route organization

### Example Conversations

**Setup Help**

> User: "How do I set up unplugin-vue-router in my Vite project?"

**Route Patterns**

> User: "What's the file naming convention for nested routes?"

**Type Safety**

> User: "How do I get type-safe route parameters?"

**Troubleshooting**

> User: "My route names aren't showing up in autocomplete. What's wrong?"

## Resources

- [unplugin-vue-router Documentation](https://uvr.esm.is)
- [GitHub Repository](https://github.com/posva/unplugin-vue-router)
- [Vue Router Documentation](https://router.vuejs.org/)

## Version

Based on unplugin-vue-router v0.19.2

## License

MIT

### Project Structure

```
unplugin-vue-router-skill/
├── SKILL.md              # Main skill documentation (required)
├── reference/            # Additional documentation (optional)
│   ├── configuration.md
│   ├── route-patterns.md
│   └── typescript.md
├── README.md             # GitHub documentation
├── LICENSE               # MIT License
└── .gitignore
```

## Author

Created to help developers leverage file-based routing with type safety in Vue 3 projects.
