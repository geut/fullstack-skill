---
title: OXC Tooling Configuration
impact: MEDIUM
tags: [tooling, linting, formatting, oxc]
---

# OXC Tooling Configuration

Use Oxlint and Oxfmt as fast, Rust-based alternatives to ESLint and Prettier for linting and formatting. This dramatically improves performance while maintaining code quality.

## Why

- **Performance**: 50-100x faster than ESLint and Prettier
- **Drop-in Replacement**: Compatible configs, minimal migration effort
- **Unified Tooling**: Single toolchain for linting and formatting
- **TypeScript Support**: Excellent TypeScript and JSX/TSX support
- **IDE Integration**: VSCode extension with auto-fix on save

## Oxlint Configuration

Configure Oxlint for comprehensive code quality checks.

```json
// Bad: ESLint with many plugins (slow)
// .eslintrc.json
{
  "extends": [
    "eslint:recommended",
    "@typescript-eslint/recommended",
    "plugin:react/recommended",
    "plugin:react-hooks/recommended"
  ],
  "plugins": ["@typescript-eslint", "react", "react-hooks"],
  "rules": {
    "no-console": "warn"
  }
}

// Good: Fast Oxlint with built-in plugins
// .oxlintrc.json
{
  "$schema": "https://raw.githubusercontent.com/oxc-project/oxc/main/npm/oxlint/configuration_schema.json",
  "plugins": [
    "eslint",
    "import",
    "node",
    "oxc",
    "promise",
    "typescript",
    "unicorn"
  ],
  "rules": {
    "no-console": "error",
    "no-unused-vars": "error",
    "no-duplicate-imports": "error",
    "no-shadow": "warn",
    "prefer-const": "warn",
    "eqeqeq": ["error", "always"]
  },
  "settings": {
    "import/resolver": {
      "typescript": true
    }
  }
}
```

## Frontend-Specific Oxlint Config

Add React and JSX-specific rules for frontend projects.

```json
// .oxlintrc.json for Astro/React projects
{
  "plugins": [
    "eslint",
    "import",
    "jsx-a11y",
    "node",
    "oxc",
    "promise",
    "react",
    "react-perf",
    "typescript",
    "unicorn"
  ],
  "rules": {
    "no-console": "error",
    "no-unused-vars": "error",
    "no-duplicate-imports": "error",
    "no-shadow": "warn",
    "react/no-array-index-key": "warn",
    "react/no-children-prop": "error",
    "react/no-children-to-array": "warn",
    "react/no-class-component": "warn",
    "react/no-context-provider": "warn",
    "react/no-default-props": "error",
    "react/no-danger": "warn",
    "react/no-direct-mutation-state": "error",
    "react/no-duplicate-key": "error",
    "react/no-forward-ref": "warn",
    "react/no-implicit-key": "error",
    "react/no-missing-context-display-name": "warn",
    "react/no-missing-key": "error",
    "react/no-prop-types": "error",
    "react/no-redundant-should-component-update": "error",
    "react/no-render-return-value": "error",
    "react/no-set-state-in-component-did-mount": "warn",
    "react/no-set-state-in-component-did-update": "warn",
    "react/no-set-state-in-component-will-update": "warn",
    "react/no-string-refs": "error",
    "react/no-unsafe-component-will-receive-props": "error",
    "react/no-unsafe-target-blank": "error",
    "react/no-unstable-context-value": "warn",
    "react/no-unstable-default-props": "warn",
    "react/no-use-context": "warn",
    "react/no-useless-forward-ref": "error",
    "react/no-void-dom-elements-no-children": "error",
    "react/no-will-update-set-state": "warn",
    "react/only-exports-component": "warn",
    "jsx-a11y/alt-text": "warn",
    "jsx-a11y/anchor-has-content": "warn",
    "jsx-a11y/anchor-is-valid": "warn",
    "jsx-a11y/aria-activedescendant-has-tabindex": "warn",
    "jsx-a11y/aria-props": "error",
    "jsx-a11y/aria-proptypes": "error",
    "jsx-a11y/aria-role": "error",
    "jsx-a11y/aria-unsupported-elements": "warn",
    "jsx-a11y/autocomplete-valid": "warn",
    "jsx-a11y/click-events-have-key-events": "warn",
    "jsx-a11y/control-has-associated-label": "warn",
    "jsx-a11y/heading-has-content": "warn",
    "jsx-a11y/html-has-lang": "warn",
    "jsx-a11y/iframe-has-title": "warn",
    "jsx-a11y/img-redundant-alt": "warn",
    "jsx-a11y/label-has-associated-control": "warn",
    "jsx-a11y/lang": "error",
    "jsx-a11y/media-has-caption": "warn",
    "jsx-a11y/mouse-events-have-key-events": "warn",
    "jsx-a11y/no-access-key": "warn",
    "jsx-a11y/no-aria-hidden-on-focusable": "error",
    "jsx-a11y/no-autofocus": "warn",
    "jsx-a11y/no-distracting-elements": "warn",
    "jsx-a11y/no-interactive-element-to-noninteractive-role": "warn",
    "jsx-a11y/no-noninteractive-element-interactions": "warn",
    "jsx-a11y/no-noninteractive-element-to-interactive-role": "warn",
    "jsx-a11y/no-noninteractive-tabindex": "warn",
    "jsx-a11y/no-redundant-roles": "warn",
    "jsx-a11y/no-static-element-interactions": "warn",
    "jsx-a11y/prefer-tag-over-role": "warn"
  }
}
```

## Oxfmt Configuration

Configure Oxfmt for consistent code formatting.

```json
// Bad: Prettier config (slow)
// .prettierrc
{
  "semi": false,
  "singleQuote": true,
  "jsxSingleQuote": true,
  "tabWidth": 2,
  "trailingComma": "all"
}

// Good: Fast Oxfmt with experimental features
// .oxfmtrc.json
{
  "semi": false,
  "singleQuote": true,
  "jsxSingleQuote": true,
  "trailingComma": "all",
  "experimentalSortImports": {
    "enabled": true,
    "newlinesBetween": true,
    "groupOrder": [
      "builtin",
      "external",
      "internal",
      "parent",
      "sibling",
      "index",
      "object",
      "type"
    ],
    "internalPattern": ["@/"]
  },
  "experimentalSortPackageJson": {
    "enabled": true
  }
}
```

## Package Scripts

Set up package.json scripts for consistent usage.

```json
{
  "scripts": {
    "lint": "oxlint",
    "lint:fix": "oxlint --fix",
    "format": "oxfmt",
    "format:check": "oxfmt --check"
  }
}
```

## Pre-commit Hooks

Configure lint-staged with Husky for pre-commit quality checks.

```json
// package.json
{
  "scripts": {
    "prepare": "husky"
  },
  "lint-staged": {
    "*.{js,ts,tsx,jsx,json}" : "bun run lint:fix"
  }
}
```

```bash
# .husky/pre-commit
bun x lint-staged
```

## VSCode Integration

Configure VSCode for optimal OXC tooling experience.

```json
// .vscode/settings.json
{
  "typescript.experimental.useTsgo": true,
  "editor.defaultFormatter": "oxc.oxc-vscode",
  "editor.formatOnSave": true,
  "editor.codeActionsOnSave": {
    "source.fixAll": "always"
  },
  "oxc.typeAware": true,
  "[astro]": {
    "editor.defaultFormatter": "oxc.oxc-vscode"
  },
  "[typescript]": {
    "editor.defaultFormatter": "oxc.oxc-vscode"
  },
  "[typescriptreact]": {
    "editor.defaultFormatter": "oxc.oxc-vscode"
  }
}
```

```json
// .vscode/extensions.json
{
  "recommendations": [
    "oxc.oxc-vscode"
  ]
}
```

## Lint Ignore Patterns

Exclude files that shouldn't be linted.

```
# .oxlintignore
dist/
node_modules/
*.config.js
*.config.ts
.drizzle/
*.generated.ts
```

## Rules

1. Use Oxlint as the primary linter instead of ESLint for 50-100x better performance
2. Use Oxfmt as the primary formatter instead of Prettier with same performance gains
3. Configure no semicolons (`"semi": false`) and single quotes (`"singleQuote": true`)
4. Enable experimental import sorting with newlines between groups
5. Use `@/` pattern for internal imports in sort configuration
6. Enable experimental package.json sorting for consistent dependency ordering
7. Set `no-console` to error level in production code
8. Include React/JSX specific plugins for frontend projects
9. Enable type-aware linting (`"oxc.typeAware": true`) in VSCode for better analysis
10. Use lint-staged with pre-commit hooks to prevent committing unlinted code
11. Install the OXC VSCode extension for in-editor linting and formatting
12. Configure `editor.formatOnSave` and `editor.codeActionsOnSave` for auto-fix on save
