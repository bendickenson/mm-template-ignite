# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Ignite is a battle-tested React Native boilerplate CLI maintained by Infinite Red. The project consists of two main parts:
1. **CLI (`src/`)**: The Ignite CLI tool that scaffolds React Native apps
2. **Boilerplate (`boilerplate/`)**: The React Native app template that gets generated

## Development Commands

### CLI Development

```bash
# Build the CLI from TypeScript
yarn build

# Type check without emitting files
yarn typecheck

# Compile TypeScript (watch mode)
yarn build:watch

# Run tests
yarn test
yarn watch              # Watch mode
yarn coverage           # With coverage

# Linting and formatting
yarn lint
yarn format:check
yarn format:write

# Clean build artifacts
yarn clean

# Test local changes to Ignite CLI
node bin/ignite new TestApp
node bin/ignite generate component MyComponent
```

### Generated Boilerplate App

These commands are available in apps created with Ignite (the `boilerplate/` folder structure):

```bash
# Start development
yarn start              # Start Expo dev server
yarn ios                # Run on iOS simulator
yarn android            # Run on Android emulator

# Testing
yarn test               # Run Jest tests
yarn test:watch         # Watch mode
yarn test:maestro       # E2E tests with Maestro

# Code quality
yarn compile            # TypeScript type checking
yarn lint               # Run ESLint with auto-fix
yarn lint:check         # Check without fixing

# Build commands (EAS)
yarn build:ios:sim      # iOS simulator build
yarn build:android:sim  # Android simulator build
yarn build:ios:prod     # Production iOS build
yarn build:android:prod # Production Android build
```

## Architecture

### CLI Architecture (`src/`)

The CLI is built with Gluegun and organized as follows:

- **`src/cli.ts`**: Entry point that creates the Gluegun CLI runtime
- **`src/commands/`**: CLI commands (new, generate, doctor, rename, etc.)
  - `new.ts`: Scaffolds new React Native apps
  - `generate.ts`: Runs code generators (component, screen, navigator, etc.)
  - `doctor.ts`: Diagnoses environment issues
  - `rename.ts`: Renames the app
- **`src/tools/`**: Shared utilities
  - `generators.ts`: Core generator logic and template rendering (EJS)
  - `react-native.ts`: React Native specific tooling
  - `packager.ts`: Package manager operations (npm/yarn)
  - `markup.ts`: Code markup/parsing utilities
  - `pretty.ts`: CLI output formatting

### Boilerplate Architecture (`boilerplate/`)

The generated React Native app follows this structure:

- **`app/`**: Main application code
  - **`components/`**: Reusable UI components (Button, Card, Header, Icon, ListItem, Screen, Text, TextField, Toggle, etc.)
  - **`screens/`**: Screen components
  - **`navigators/`**: React Navigation configuration
    - `AppNavigator.tsx`: Main app navigation
    - `DemoNavigator.tsx`: Demo screens navigation
    - `navigationUtilities.ts`: Navigation helpers
  - **`theme/`**: Theming system with dark mode support
    - `colors.ts` / `colorsDark.ts`: Color palettes
    - `spacing.ts` / `spacingDark.ts`: Spacing scales
    - `typography.ts`: Font definitions
    - `context.tsx`: Theme provider and hooks
  - **`i18n/`**: Internationalization (i18next) with RTL support
  - **`utils/`**: Utility functions (storage, formatDate, etc.)
  - **`services/`**: API clients and external services
  - **`config/`**: App configuration
  - **`context/`**: React Context providers (AuthContext, etc.)
  - **`devtools/`**: Reactotron configuration
- **`ignite/templates/`**: Local generator templates
  - `app-icon/`, `component/`, `navigator/`, `screen/`, `splash-screen/`

### Generators

Generators create new code from templates in `boilerplate/ignite/templates/`:

```bash
npx ignite-cli generate component MyComponent
npx ignite-cli generate screen MyScreen
npx ignite-cli generate navigator MyNavigator
```

Generators support:
- `--dir`: Override output directory
- `--case`: Control filename casing (auto, pascal, kebab, snake, none)
- `--update`: Update generator templates from latest Ignite

## Key Technologies

- **TypeScript 5**: Strict typing throughout
- **React Native 0.81** + **React 19**
- **Expo SDK 54**: Modular development platform
- **React Navigation 7**: Navigation framework
- **Reanimated 4**: Animations
- **MMKV**: Fast key-value storage
- **apisauce**: HTTP client (Axios wrapper)
- **Jest**: Testing framework
- **date-fns 4**: Date manipulation
- **Gluegun**: CLI framework (for Ignite CLI itself)
- **sharp**: Image processing (for app icons)

## Testing

- **CLI**: Jest tests in `test/` directory
- **Boilerplate**: Jest + React Testing Library
  - Component tests live alongside components (e.g., `Text.test.tsx`)
  - Test utilities in `app/devtools/`
- **E2E**: Maestro tests in `.maestro/flows/`

## Important Patterns

### Theme System
The boilerplate uses a context-based theming system with automatic dark mode support. Colors, spacing, and typography are defined in separate theme files and accessed via hooks:

```typescript
import { useAppTheme } from "@/theme"
const { themed, theme } = useAppTheme()
```

### Component Structure
Components are self-contained with inline TypeScript types and use the theme system. Most components support presets for common variants.

### Generator Templates
Templates use EJS syntax and support frontmatter for configuration (output directory, filename casing, etc.). The `generators.ts` file handles template rendering and file operations.

### Demo Code Removal
The boilerplate includes demo screens. Use these commands to remove them:
- `npx ignite-cli remove-demo`: Remove demo screens but keep markup
- `npx ignite-cli remove-demo-markup`: Remove all demo-related code

## Release Process

Ignite uses semantic-release for automated versioning:
- Commit messages must follow conventional commits format
- PRs are merged with proper semantic prefixes (feat:, fix:, etc.)
- Releases are automated via CircleCI
- See `docs/contributing/Releasing-Ignite.md` for details

## Node Version

Requires Node >= 20.0.0 (specified in `.nvmrc` and `package.json` engines)
