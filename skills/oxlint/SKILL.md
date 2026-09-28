---
name: oxlint
description: >-
  Run and configure oxlint, the high-performance JavaScript/TypeScript linter from Oxc.
  Use when running lint checks, fixing code issues, configuring .oxlintrc.json or oxlint.config.ts,
  setting up rules, categories, plugins, and CLI options.
---

# oxlint Skill

`oxlint` is an ultra-fast JavaScript and TypeScript linter from the Oxc project designed as a drop-in, zero-config enhancement or replacement for ESLint. It runs in milliseconds without requiring pre-configuration.

---

## ⚡ Quick Reference

### Running Oxlint

```bash
# Run on current directory (zero config required)
npx oxlint

# Run and automatically apply safe fixes
npx oxlint --fix

# Enable specific rule categories (-D for error, -W for warning, -A for allow)
npx oxlint -D correctness -W suspicious -W pedantic

# Enable native plugins via CLI
npx oxlint --react-plugin --import-plugin --jsx-a11y-plugin

# Type-aware linting (rules requiring TypeScript types)
npx oxlint --type-aware

# Check only errors in CI with failure on warnings
npx oxlint --deny-warnings --format github
```

---

## ⚙️ Configuration Setup

### Option A: `oxlint.config.ts` (Recommended)

```typescript
import { defineConfig } from 'oxlint';

export default defineConfig({
  categories: {
    correctness: 'error',
    suspicious: 'warn'
  },
  plugins: ['react', 'unicorn', 'typescript', 'import'],
  rules: {
    'no-debugger': 'error',
    'no-unused-vars': 'error',
    'unicorn/prefer-node-protocol': 'error'
  },
  env: {
    browser: true,
    node: true
  },
  ignorePatterns: ['dist/**', 'coverage/**']
});
```

### Option B: `.oxlintrc.json`

```json
{
  "$schema": "./node_modules/oxlint/configuration_schema.json",
  "categories": {
    "correctness": "error",
    "suspicious": "warn"
  },
  "plugins": ["react", "typescript"],
  "rules": {
    "eqeqeq": "error"
  }
}
```

---

## 🚫 Inline Suppressions

Oxlint supports both `oxlint-disable` and `eslint-disable` comments:

```typescript
// oxlint-disable-next-line no-debugger
debugger;

// oxlint-disable-next-line no-console, eqeqeq
console.log(1 == "1");

/* oxlint-disable no-alert */
alert("Hello");
/* oxlint-enable no-alert */
```

---

## 📚 References

The following guides are located directly alongside this skill file:

- [CLI Reference](file:///c:/_/skills/skills/oxlint/cli.md): All CLI flags, category toggles (`-D`, `-W`, `-A`), output formatters (`github`, `json`, `sarif`), `--fix`, `--type-aware`, and debugging options.
- [Configuration Reference](file:///c:/_/skills/skills/oxlint/config.md): Complete options for `oxlint.config.ts` (`defineConfig`) and `.oxlintrc.json`, including categories, rule options, overrides, env, and JS plugins.
- [Rules Reference](file:///c:/_/skills/skills/oxlint/rules.md): Rule namespaces (`typescript/`, `react/`, `unicorn/`, `import/`, `oxc/`), inline disable directives, and querying available rules with `--rules`.
