# Oxlint Configuration Reference

Oxlint supports both static JSON/JSONC configuration files (`.oxlintrc.json`, `.oxlintrc.jsonc`) and programmatic TypeScript/JavaScript configuration files (`oxlint.config.ts`, `oxlint.config.js`).

---

## 1. File Formats & Discovery

Oxlint automatically searches for the following configuration files in the current working directory:
1. `oxlint.config.ts` / `oxlint.config.mts`
2. `oxlint.config.js` / `oxlint.config.mjs`
3. `.oxlintrc.json`
4. `.oxlintrc.jsonc`

Nested configurations in subdirectories are automatically discovered and merged for files within those directories, unless `--disable-nested-config` is passed.

---

## 2. Programmatic Configuration (`oxlint.config.ts`)

`oxlint.config.ts` provides complete TypeScript type safety and autocompletion using `defineConfig`:

```typescript
import { defineConfig } from 'oxlint';

export default defineConfig({
  categories: {
    correctness: 'error',
    suspicious: 'warn',
    pedantic: 'off'
  },
  plugins: ['react', 'unicorn', 'import', 'jsx-a11y'],
  rules: {
    'no-unused-vars': 'error',
    'no-console': ['warn', { allow: ['warn', 'error'] }],
    'react/jsx-key': 'error',
    'import/no-cycle': 'error'
  },
  env: {
    browser: true,
    node: true,
    es2024: true
  },
  globals: {
    AudioWorkletGlobalScope: 'readonly'
  },
  ignorePatterns: [
    'dist/**',
    'coverage/**',
    '**/*.generated.ts'
  ],
  overrides: [
    {
      files: ['**/*.test.ts', '**/*.spec.ts'],
      rules: {
        'no-console': 'off'
      }
    }
  ]
});
```

---

## 3. Static JSON Configuration (`.oxlintrc.json`)

```json
{
  "$schema": "./node_modules/oxlint/configuration_schema.json",
  "categories": {
    "correctness": "error",
    "suspicious": "warn"
  },
  "plugins": [
    "react",
    "typescript",
    "unicorn"
  ],
  "rules": {
    "eqeqeq": "error",
    "no-debugger": "error",
    "typescript/no-explicit-any": "warn"
  },
  "env": {
    "browser": true,
    "node": true
  },
  "ignorePatterns": [
    "build/**"
  ]
}
```

---

## 4. Key Configuration Fields

### `categories`
Bulk severity control for rule categories. Values can be `'error'`, `'warn'`, or `'off'`:
- `correctness`: Enabled by default (`error`). Detects bugs and broken code.
- `suspicious`: Potential bugs, unhandled states, or suspicious syntax.
- `pedantic`: Strict guidelines, type narrowing checks, clean code requirements.
- `perf`: Performance and optimization recommendations.
- `style`: Formatting-adjacent and stylistic conventions.
- `restriction`: Disallowing specific syntax features (e.g. `eval`, `var`).
- `nursery`: Experimental rules in development.

### `rules`
Explicit configuration for individual rules. Overrides category-level settings:
- `'off'` or `'allow'`: Disable the rule.
- `'warn'`: Report as warning.
- `'error'` or `'deny'`: Report as error.
- `['error', options]`: Enable with custom options.

### `plugins` (Built-in Native Plugins)
Array of native plugin identifiers:
`'react'`, `'unicorn'`, `'typescript'`, `'oxc'`, `'import'`, `'jsdoc'`, `'jest'`, `'vitest'`, `'jsx-a11y'`, `'nextjs'`, `'react-perf'`, `'promise'`, `'node'`, `'vue'`.

### `jsPlugins` (Custom JavaScript / TypeScript Plugins)
Load external or local custom JS plugins:

```typescript
export default defineConfig({
  jsPlugins: [
    './plugins/my-local-plugin.ts',
    'oxlint-plugin-custom'
  ],
  rules: {
    'my-local-plugin/rule-name': 'error'
  }
});
```

### `extends`
Inherit from other configurations.
- In `oxlint.config.ts`, directly import and spread/pass plugin configs:
  ```typescript
  import { defineConfig } from 'oxlint';
  import customPlugin from 'oxlint-plugin-custom';

  export default defineConfig({
    extends: [customPlugin.configs.recommended]
  });
  ```
- In `.oxlintrc.json`, reference file paths or presets.

### `overrides`
Apply custom rules or plugin settings to matching file patterns:

```typescript
export default defineConfig({
  overrides: [
    {
      files: ['scripts/**/*.js'],
      env: { node: true },
      rules: {
        'no-console': 'off'
      }
    },
    {
      files: ['src/components/**/*.tsx'],
      rules: {
        'react/prop-types': 'off'
      }
    }
  ]
});
```

### `settings`
Shared configuration passed to rule listeners (e.g. JSX components, React versions):

```typescript
export default defineConfig({
  settings: {
    react: {
      version: 'detect'
    },
    jsxA11y: {
      components: {
        CustomButton: 'button'
      }
    }
  }
});
```

### `options`
Internal engine flags:
```typescript
export default defineConfig({
  options: {
    typeAware: true // Enable type-aware linting
  }
});
```
