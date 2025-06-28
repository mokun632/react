# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the React library repository - a monorepo containing React core, DOM renderer, server components, and the entire React ecosystem. The project uses Yarn workspaces with packages in `/packages/*` and has a sophisticated build system supporting multiple release channels and deployment targets.

## Common Commands

### Build System
```bash
yarn build                          # Build all packages for all release channels
yarn build-for-devtools            # Build packages needed for React DevTools
yarn build-for-flight-dev          # Build for React Server Components development
node scripts/rollup/build.js <packages> --type=<bundle-types>  # Build specific packages
```

### Testing
```bash
yarn test                          # Run all tests with Jest
yarn test-stable                   # Test stable release channel
yarn test-www                      # Test www-modern release channel 
yarn test-build-devtools          # Test DevTools build
yarn test-dom-fixture             # Test DOM fixtures in /fixtures/dom
node scripts/jest/jest-cli.js --release-channel=<channel>  # Test specific release channel
```

### Code Quality
```bash
yarn lint                         # Run ESLint with React-specific rules
yarn flow                         # Flow type checking
yarn prettier                     # Format changed files
yarn prettier-all                # Format all files
```

### Development Utilities
```bash
yarn extract-errors               # Extract error codes for minification
yarn download-build               # Download experimental builds from CI
yarn flags                        # Show available feature flags
```

## Architecture Overview

### Monorepo Structure
- **Core Packages**: `react`, `react-dom`, `react-reconciler`, `scheduler`, `shared`
- **Server Rendering**: `react-server`, `react-server-dom-*` (webpack/parcel/turbopack/esm variants)
- **Development Tools**: `react-devtools-*`, `react-test-renderer`, `eslint-plugin-react-hooks`
- **Specialized Renderers**: `react-native-renderer`, `react-art`, `react-noop-renderer`

### Build System (Rollup-based)
- **Bundle Types**: NODE_DEV, NODE_PROD, ESM_DEV, ESM_PROD, FB_WWW, RN_OSS, etc.
- **Release Channels**: `stable`, `experimental`, `www-modern`, `www-classic`
- **Feature Flags**: Compile-time and runtime flags via `ReactFeatureFlags`
- **Configuration**: `/scripts/rollup/` contains all build logic, bundles, and plugins

### Testing Infrastructure
- **Jest Configuration**: Multiple configs in `/scripts/jest/` for different environments
- **Test Types**: Unit tests, integration tests, fixtures, DevTools regression tests
- **Custom Matchers**: React-specific test utilities in `/scripts/jest/matchers/`

## Key Development Patterns

### Fiber Architecture
React uses the Fiber reconciler with advanced features:
- Time-slicing and priority-based scheduling
- Suspense and concurrent rendering
- Host config pattern for platform abstraction

### Feature Flag System
- Flags defined in `ReactFeatureFlags.js` per package
- Different flag values for different release channels
- Use `__EXPERIMENTAL__` and `__DEV__` for conditional code

### Server Components Integration
- Server-side rendering with streaming capabilities
- Flight protocol for server-client communication
- Multiple bundler integrations (webpack, parcel, turbopack, esm)

### Platform Abstraction
- Host configs define platform-specific behavior
- Separate renderers for DOM, React Native, testing, etc.
- Shared reconciler logic across all platforms

## React Compiler
- Located in `/compiler/` directory with separate build system
- Experimental optimization tool for automatic memoization
- Has its own playground app and test suite
- Build with `./scripts/react-compiler/build-compiler.sh`

## Custom Directories
- `/learning-log/`: Documentation of React internals and architecture
- `/memory-bank/`: Development context and progress tracking
- `/fixtures/`: Real-world integration test applications

## Build Output
- **Development builds**: `/build/node_modules/` (unminified with dev warnings)
- **Production builds**: Minified and optimized via Closure Compiler
- **Package specific**: Each package has its own build artifacts