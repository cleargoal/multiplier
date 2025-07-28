# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Vue 3 project called "multiplier" built with Vite, using modern Vue composition API and Pinia for state management. The project follows standard Vue 3 conventions with Single File Components (SFC).

## Development Commands

```bash
# Install dependencies
pnpm install
pnpm approve-builds  # for pnpm 10+

# Development server with hot reload
pnpm dev

# Build for production
pnpm build

# Preview production build
pnpm preview

# Run unit tests (Vitest)
pnpm test:unit

# Run end-to-end tests (Nightwatch)
pnpm test:e2e
# Run E2E tests on specific browser
pnpm test:e2e --env chrome
# Run specific test file
pnpm test:e2e tests/e2e/example.js
# Run E2E tests in debug mode
pnpm test:e2e --debug

# Lint and fix code
pnpm lint

# Format code
pnpm format
```

## Architecture

### Tech Stack
- **Frontend Framework**: Vue 3 with Composition API
- **Build Tool**: Vite 7.0
- **State Management**: Pinia
- **Testing**: Vitest (unit tests) + Nightwatch (E2E tests)
- **Linting**: ESLint with Vue plugin
- **Formatting**: Prettier

### Project Structure
- `src/main.js` - Application entry point, sets up Vue app with Pinia
- `src/App.vue` - Root component
- `src/components/` - Reusable Vue components
- `src/stores/` - Pinia stores for state management
- `src/assets/` - Static assets (CSS, images)
- `tests/e2e/` - End-to-end tests using Nightwatch
- `src/components/__tests__/` - Unit tests for components

### State Management
- Uses Pinia with Composition API syntax
- Example store in `src/stores/counter.js` demonstrates reactive state, computed values, and actions

### Vite Configuration
- Path alias `@` points to `src/` directory
- Configured with Vue, Vue JSX, Vue DevTools, and Nightwatch plugins

### Testing Setup
- **Unit Tests**: Vitest with jsdom environment for component testing
- **E2E Tests**: Nightwatch configured for Firefox (default), Chrome, Safari, and Edge
- E2E tests run against development server on port 5173 (or 4173 in CI)

### Code Style
- ESLint configured with Vue essential rules and Vitest plugin
- Prettier integration for consistent formatting
- Global ignores for dist, coverage directories