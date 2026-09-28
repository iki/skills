# Oxfmt CLI Reference

`oxfmt` is a high-performance Prettier-compatible code formatter developed in Rust as part of the Oxc project.

---

## 1. Primary Output Modes

Unlike Prettier, `oxfmt` writes formatted files in place by default.

| Mode | Command | Description |
|---|---|---|
| **Write (Default)** | `npx oxfmt [PATH]` | Formats files in place. Equivalent to `oxfmt --write`. |
| **Check (CI)** | `npx oxfmt --check [PATH]` | Checks if files are already formatted. Returns exit code 1 if unformatted files exist. |
| **List Different** | `npx oxfmt --list-different [PATH]` | Prints names of files that would be modified, then exits. |
| **Stdin** | `cat file.ts \| npx oxfmt --stdin-filepath file.ts` | Reads code from stdin and outputs formatted code to stdout. |

### Common Usage Examples

```bash
# Format all supported files in current directory
npx oxfmt

# Format specific directories or glob patterns (always quote globs)
npx oxfmt 'src/**/*.{js,ts,jsx,tsx,json,md}'

# Exclude patterns with '!' prefix
npx oxfmt 'src/**/*.ts' '!src/**/*.generated.ts'

# Verify formatting in CI pipeline
npx oxfmt --check
```

---

## 2. Configuration & Initialization Modes

| Flag | Description |
|---|---|
| `--init` | Initializes a default `.oxfmtrc.json` configuration file in the current directory. |
| `--migrate prettier` | Reads existing Prettier configuration (`.prettierrc`, `prettier.config.js`, etc.) and migrates settings to `.oxfmtrc.json`. |
| `--migrate biome` | Reads existing Biome configuration (`biome.json`) and migrates formatting settings to `.oxfmtrc.json`. |
| `--lsp` | Starts the Oxfmt Language Server Protocol (LSP) server for editor integrations. |

```bash
# Migrate from Prettier to Oxfmt
npx oxfmt --migrate prettier

# Initialize new configuration
npx oxfmt --init
```

---

## 3. Configuration & Ignore Options

| Flag | Description |
|---|---|
| `-c, --config <PATH>` | Explicit path to `.oxfmtrc.json`, `.oxfmtrc.jsonc`, or config script (`.ts`, `.js`, `.mjs`, etc.). |
| `--disable-nested-config` | Disable automatic discovery of nested `.oxfmtrc.json` files in subdirectories. |
| `--ignore-path <PATH>` | Path to custom ignore file(s). Can be repeated. Defaults to `.gitignore` and `.prettierignore`. |
| `--with-node_modules` | Include the `node_modules/` directory (skipped by default). |

---

## 4. Runtime Options

| Flag | Description |
|---|---|
| `--threads <INT>` | Number of worker threads. Set to `1` to restrict to a single CPU core. Defaults to hardware concurrency. |
| `--no-error-on-unmatched-pattern` | Prevent non-zero exit when a provided glob pattern does not match any files. |
