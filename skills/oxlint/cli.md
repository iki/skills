# Oxlint CLI Reference

`oxlint` is a high-performance linter for JavaScript and TypeScript designed to run in milliseconds without mandatory configuration.

---

## 1. Basic Invocation

```bash
# Lint current directory
npx oxlint

# Lint specific directory or files
npx oxlint src/
npx oxlint src/index.ts src/utils.ts

# Lint with glob patterns (quote patterns to prevent shell expansion)
npx oxlint 'src/**/*.ts' '!src/**/*.test.ts'
```

---

## 2. Rule and Category Severity Flags

Flags accumulate from left to right. Categories and individual rules can be set to:
- `-D`, `--deny <NAME>`: Emit error (exit code 1 on violation)
- `-W`, `--warn <NAME>`: Emit warning
- `-A`, `--allow <NAME>`: Disable / suppress the rule or category

### Available Categories

| Category | Description | Default Status |
|---|---|---|
| `correctness` | Code that is outright wrong, buggy, or useless | Enabled (`deny`) |
| `suspicious` | Code that is likely wrong or error-prone | Off |
| `pedantic` | Strict lints with occasional false positives | Off |
| `perf` | Performance optimizations | Off |
| `style` | Idiomatic code style conventions | Off |
| `restriction` | Restrictions on language or library features | Off |
| `nursery` | Experimental rules under active development | Off |
| `all` | All categories except `nursery` | Off |

### Examples

```bash
# Enable suspicious and pedantic rules as warnings
npx oxlint -W suspicious -W pedantic

# Deny all correctness and suspicious rules, but allow debugger
npx oxlint -D correctness -D suspicious -A no-debugger

# Enable all categories as warnings, but deny a specific rule
npx oxlint -W all -D no-var
```

---

## 3. Fixing Problems

Oxlint supports multiple levels of automated code fixing:

| Flag | Description |
|---|---|
| `--fix` | Fix safe issues automatically. Only unfixed issues remain in output. |
| `--fix-suggestions` | Apply auto-fixable suggestions (may alter program behavior). |
| `--fix-dangerously` | Apply dangerous fixes and suggestions. |

```bash
# Automatically fix safe violations in place
npx oxlint --fix

# Fix safe violations and apply suggestions
npx oxlint --fix --fix-suggestions
```

---

## 4. Built-in Plugin Flags

Oxlint includes fast native implementations of popular ESLint plugins. Plugins can be toggled via CLI:

| Flag | Default | Description |
|---|---|---|
| `--disable-oxc-plugin` | Enabled | Disable unique Oxlint rules. |
| `--disable-typescript-plugin` | Enabled | Disable TypeScript rules. |
| `--disable-unicorn-plugin` | Enabled | Disable Unicorn rules. |
| `--react-plugin` | Disabled | Enable React rules. |
| `--react-perf-plugin` | Disabled | Enable React rendering performance rules. |
| `--import-plugin` | Disabled | Enable ESM import/export validation. |
| `--jsdoc-plugin` | Disabled | Enable JSDoc formatting and syntax checks. |
| `--jsx-a11y-plugin` | Disabled | Enable JSX accessibility checks. |
| `--nextjs-plugin` | Disabled | Enable Next.js specific rules. |
| `--jest-plugin` | Disabled | Enable Jest test rules. |
| `--vitest-plugin` | Disabled | Enable Vitest test rules. |
| `--promise-plugin` | Disabled | Enable Promise usage rules. |
| `--node-plugin` | Disabled | Enable Node.js usage rules. |
| `--vue-plugin` | Disabled | Enable Vue.js rules. |

```bash
# Enable React and JSX accessibility rules
npx oxlint --react-plugin --jsx-a11y-plugin
```

---

## 5. Type-Aware Linting & Type-Checking

Oxlint can perform type-aware linting (rules requiring TypeScript type information) and standalone TypeScript type-checking:

| Flag | Description |
|---|---|
| `--type-aware` | Enables rules requiring type information (e.g. `@typescript-eslint/no-floating-promises`). Automatically discovers `tsconfig.json`. |
| `--type-check` | Performs experimental TypeScript compiler type checking alongside linting. Requires `--type-aware`. |
| `--type-check-only` | Runs only TypeScript compiler type checking diagnostics without lint rules. |

```bash
# Run type-aware rules
npx oxlint --type-aware

# Run type-aware linting + compiler type checking
npx oxlint --type-aware --type-check
```

---

## 6. Output and Reporting Options

### Output Formats (`--format <FORMAT>`)

Supported formats: `default`, `stylish`, `json`, `unix`, `checkstyle`, `github`, `gitlab`, `junit`, `sarif`, `agent`.

```bash
# GitHub Actions annotations
npx oxlint --format github

# Machine-readable JSON output
npx oxlint --format json

# SARIF output for security scanners
npx oxlint --format sarif > oxlint.sarif
```

### Warning Control

| Flag | Description |
|---|---|
| `--quiet` | Suppress warning output; report only errors. |
| `--deny-warnings` | Exit with a non-zero code if any warnings are emitted. |
| `--max-warnings <INT>` | Exit with an error if warnings exceed the threshold. |

---

## 7. Configuration and Discovery Flags

| Flag | Description |
|---|---|
| `-c, --config <PATH>` | Explicit path to `.oxlintrc.json`, `.oxlintrc.jsonc`, or `oxlint.config.ts`. |
| `--init` | Interactively initialize an Oxlint configuration. |
| `--disable-nested-config` | Disable automatic discovery of nested config files in subdirectories. |
| `--tsconfig <PATH>` | Explicit path to `tsconfig.json` (use with caution; auto-discovery is preferred). |
| `--ignore-path <PATH>` | Specify custom ignore files (can be repeated). Defaults to `.gitignore` and `.eslintignore`. |
| `--rules` | List all available registered rules and their active status. |
| `--lsp` | Start the language server for IDE integrations. |
| `--threads <INT>` | Number of worker threads (defaults to logical CPU core count). |
| `--debug files` | Print list of files that will be linted without executing rules. |
| `--debug timings` | Print per-rule execution time breakdown. |
