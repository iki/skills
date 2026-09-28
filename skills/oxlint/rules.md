# Oxlint Rules and Directives Reference

This document covers rule namespaces, directive comments for suppressing lints, and inspecting registered rules in Oxlint.

---

## 1. Rule Namespaces & Plugins

Oxlint supports standard ESLint core rules and multiple plugin namespaces natively:

| Namespace / Plugin | Example Rules | Notes |
|---|---|---|
| **Core (ESLint)** | `no-debugger`, `no-unused-vars`, `eqeqeq`, `no-var` | Standard JS rules without namespace prefix |
| **`typescript/`** | `typescript/no-explicit-any`, `typescript/ban-types` | Compatible with `@typescript-eslint/*` names |
| **`oxc/`** | `oxc/no-accumulating-spread`, `oxc/double-comparisons` | Performance and correctness rules unique to Oxc |
| **`react/`** | `react/jsx-key`, `react/no-direct-mutation-state` | React component and JSX linting |
| **`react-hooks/`** | `react-hooks/rules-of-hooks`, `react-hooks/exhaustive-deps` | React Hook rules |
| **`unicorn/`** | `unicorn/prefer-node-protocol`, `unicorn/no-null` | Powerful modern JavaScript guidelines |
| **`import/`** | `import/no-cycle`, `import/no-duplicates` | ESM module import and export structure |
| **`jsx-a11y/`** | `jsx-a11y/alt-text`, `jsx-a11y/anchor-is-valid` | Accessibility validation |
| **`nextjs/`** | `nextjs/no-img-element`, `nextjs/no-head-element` | Next.js best practices |
| **`jest/` / `vitest/`**| `jest/no-disabled-tests`, `vitest/expect-expect` | Unit and integration test assertions |
| **`promise/`** | `promise/param-names`, `promise/always-return` | Promise chain handling |
| **`node/`** | `node/no-exports-assign`, `node/handle-callback-err` | Node.js runtime checks |

---

## 2. Inline Suppression Directives

Oxlint supports both native `oxlint-disable` comments and standard `eslint-disable` comments for full backward compatibility.

### Disable for Next Line
Suppresses rule violations occurring on the immediately following line:

```javascript
// oxlint-disable-next-line no-debugger
debugger;

// Disable multiple rules
// oxlint-disable-next-line no-console, eqeqeq
console.log(1 == "1");

// ESLint syntax is also supported
// eslint-disable-next-line typescript/no-explicit-any
const data: any = response;
```

### Disable for Current Line
Suppresses violations on the current line:

```javascript
debugger; // oxlint-disable-line no-debugger
```

### Disable Block or Entire File
Disables rules from the comment onward, or until re-enabled:

```javascript
/* oxlint-disable */
// All rules disabled for this section

/* oxlint-enable */
// Rules re-enabled

/* oxlint-disable no-console, no-alert */
console.log('Testing');
alert('Message');
/* oxlint-enable no-console, no-alert */
```

### Unused Directive Detection

To prevent stale suppression comments from lingering in the codebase:

```bash
# Report unused disable comments
npx oxlint --report-unused-disable-directives
```

---

## 3. Querying Available Rules

You can inspect the full database of rules supported by your installed version of Oxlint:

```bash
# Print list of all rules and whether they are enabled
npx oxlint --rules

# Filter rules by name
npx oxlint --rules | grep -i "react"
```
