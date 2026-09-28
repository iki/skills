---
name: oxlint-rules
description: >-
  Author custom JavaScript/TypeScript lint rules and plugins for oxlint using @oxlint/plugins.
  Use when creating custom oxlint rules, implementing create/createOnce visitors, AST traversal,
  diagnostic reporting, autofixes, suggestions, and testing rules with RuleTester.
---

# oxlint-rules Skill

This skill provides comprehensive guidance for authoring, testing, and packaging custom JavaScript and TypeScript lint rules and plugins for Oxlint using the `@oxlint/plugins` API.

---

## ⚡ Quick Reference

### 1. Defining a Custom Rule and Plugin

```typescript
import { definePlugin, defineRule } from '@oxlint/plugins';

const noMathRandom = defineRule({
  meta: {
    type: 'suggestion',
    docs: {
      description: 'Disallow Math.random in favor of crypto.getRandomValues'
    },
    fixable: 'code',
    messages: {
      useCrypto: 'Avoid Math.random(); use crypto.getRandomValues instead.'
    }
  },
  create(context) {
    return {
      MemberExpression(node) {
        if (
          !node.computed &&
          node.object.type === 'Identifier' &&
          node.object.name === 'Math' &&
          node.property.type === 'Identifier' &&
          node.property.name === 'random'
        ) {
          context.report({
            node,
            messageId: 'useCrypto',
            fix: (fixer) => fixer.replaceText(node, 'crypto.getRandomValues')
          });
        }
      }
    };
  }
});

export default definePlugin({
  meta: {
    name: 'security-rules'
  },
  rules: {
    'no-math-random': noMathRandom
  }
});
```

### 2. High-Performance `createOnce` API

Oxlint introduces `createOnce` to execute rule initialization once per process instead of once per file. It supports `before()` and `after()` hooks:

```typescript
import { defineRule } from '@oxlint/plugins';

export const skipTestFilesRule = defineRule({
  createOnce(context) {
    return {
      before() {
        // Skip rule execution on test files completely
        if (context.filename.endsWith('.test.ts')) {
          return false;
        }
      },
      DebuggerStatement(node) {
        context.report({ node, message: 'Unexpected debugger' });
      }
    };
  }
});
```

### 3. Registering the Plugin in `oxlint.config.ts`

```typescript
import { defineConfig } from 'oxlint';
import securityRules from './plugins/security-rules.ts';

export default defineConfig({
  jsPlugins: [securityRules],
  rules: {
    'security-rules/no-math-random': 'error'
  }
});
```

### 4. Unit Testing with `RuleTester`

```typescript
import { RuleTester } from 'oxlint';
import plugin from './security-rules.ts';

const ruleTester = new RuleTester();

ruleTester.run('no-math-random', plugin.rules['no-math-random'], {
  valid: [
    'crypto.getRandomValues(arr);',
    'Math.floor(1.5);'
  ],
  invalid: [
    {
      code: 'const val = Math.random();',
      errors: [{ messageId: 'useCrypto' }],
      output: 'const val = crypto.getRandomValues;'
    }
  ]
});
```

---

## 📚 References

The following guides are located directly alongside this skill file:

- [Plugin API Reference](file:///c:/_/skills/skills/oxlint-rules/plugin-api.md): `@oxlint/plugins` functions (`definePlugin`, `defineRule`, `eslintCompatPlugin`), `create` vs `createOnce` lifecycle hooks (`before`, `after`), and config registration.
- [Context and AST Reference](file:///c:/_/skills/skills/oxlint-rules/context-and-ast.md): The `context` object, querying tokens, comments, and scopes via `SourceCode`, and ESTree AST node traversal.
- [Diagnostics and Fixes Reference](file:///c:/_/skills/skills/oxlint-rules/fixes-and-suggestions.md): Violation reporting, `messageId` template interpolation, automated code repairs with `Fixer`, and interactive suggestions.
- [Testing Rules Reference](file:///c:/_/skills/skills/oxlint-rules/testing-rules.md): Unit testing custom rules using Oxlint's `RuleTester` across Vitest, Jest, and Node test runner.
