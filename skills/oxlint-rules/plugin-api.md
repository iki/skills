# Oxlint Plugin API Reference

Oxlint supports custom JavaScript and TypeScript plugins via `@oxlint/plugins`. Plugins can be authored directly in TypeScript with complete type safety.

---

## 1. Package Installation

```bash
npm install @oxlint/plugins
```

---

## 2. Defining Plugins and Rules

The `@oxlint/plugins` package provides `definePlugin` and `defineRule` to ensure proper type checking:

```typescript
import { definePlugin, defineRule } from '@oxlint/plugins';

const noDebugger = defineRule({
  meta: {
    type: 'problem',
    docs: {
      description: 'Disallow the use of debugger'
    },
    fixable: 'code',
    messages: {
      unexpected: 'Unexpected debugger statement.'
    }
  },
  create(context) {
    return {
      DebuggerStatement(node) {
        context.report({
          node,
          messageId: 'unexpected',
          fix: (fixer) => fixer.remove(node)
        });
      }
    };
  }
});

export default definePlugin({
  meta: {
    name: 'custom-plugin'
  },
  rules: {
    'no-debugger': noDebugger
  }
});
```

---

## 3. Rule Lifecycle: `create` vs `createOnce`

Oxlint supports two rule execution models:

### 1. `create(context)` (Standard ESLint-Compatible)
The `create` function is executed once per linted file.
- `context.filename`, `context.sourceCode`, and file contents are immediately available.
- State inside the `create` closure is local to that specific file.

```typescript
import { defineRule } from '@oxlint/plugins';

export const myRule = defineRule({
  create(context) {
    // Called once per file
    const filename = context.filename;

    return {
      Identifier(node) {
        // Handle AST nodes
      }
    };
  }
});
```

### 2. `createOnce(context)` (High-Performance Oxlint Extension)
The `createOnce` function is called **only once per process run**, eliminating repeated setup overhead across thousands of files.

In `createOnce`, the returned visitor can include lifecycle hooks:
- **`before()`**: Runs before a file is traversed. If `before()` returns `false`, Oxlint completely skips AST traversal for this rule on that file!
- **`after()`**: Runs after a file has finished AST traversal.

```typescript
import { defineRule } from '@oxlint/plugins';

export const fastRule = defineRule({
  createOnce(context) {
    // Executed once for the entire lint process run

    return {
      before() {
        // Skip rule execution on test files
        if (context.filename.endsWith('.test.ts')) {
          return false;
        }
      },
      CallExpression(node) {
        // Traverse matching nodes in eligible files
      },
      after() {
        // Clean up or finalize per-file diagnostics
      }
    };
  }
});
```

---

## 4. ESLint Compatibility (`eslintCompatPlugin`)

If you write rules utilizing the high-performance `createOnce` API with `before`/`after` hooks, you can use `eslintCompatPlugin` so the exact same plugin functions in both Oxlint and standard ESLint:

```typescript
import { eslintCompatPlugin } from '@oxlint/plugins';

export default eslintCompatPlugin({
  meta: { name: 'my-plugin' },
  rules: {
    'fast-rule': fastRule
  }
});
```

---

## 5. Registering Plugins in `oxlint.config.ts`

Custom plugins can be loaded in `oxlint.config.ts` via relative file paths or installed npm package names:

```typescript
import { defineConfig } from 'oxlint';
import myCustomPlugin from './plugins/my-custom-plugin.ts';

export default defineConfig({
  jsPlugins: [
    myCustomPlugin,              // Direct plugin object import
    './plugins/another-plugin.ts',// Relative path
    'oxlint-plugin-community'    // NPM package name
  ],
  rules: {
    'custom-plugin/no-debugger': 'error',
    'another-plugin/rule-name': 'warn'
  }
});
```
