---
name: oxfmt
description: >-
  Format code and configure oxfmt, the high-performance Prettier-compatible code formatter from Oxc.
  Use when formatting files, checking formatting in CI, configuring .oxfmtrc.json, setting up
  import/package/tailwind sorting, and migrating from Prettier or Biome.
---

# oxfmt Skill

`oxfmt` is a high-speed code formatter developed in Rust as part of the Oxc suite. Designed as a drop-in, zero-dependency alternative to Prettier, it runs orders of magnitude faster while formatting files in place by default and maintaining Prettier compatibility.

---

## ⚡ Quick Reference

### Running Oxfmt

```bash
# Format entire codebase in place (default behavior)
npx oxfmt

# Format specific directories or globs
npx oxfmt src/
npx oxfmt 'src/**/*.{ts,tsx,json,md}'

# Verify formatting in CI without modifying files
npx oxfmt --check

# List files that differ from formatted output
npx oxfmt --list-different

# Initialize default .oxfmtrc.json
npx oxfmt --init

# Migrate from existing Prettier configuration
npx oxfmt --migrate prettier

# Migrate from existing Biome configuration
npx oxfmt --migrate biome
```

---

## ⚙️ Configuration (`.oxfmtrc.json`)

```json
{
  "$schema": "./node_modules/oxfmt/configuration_schema.json",
  "printWidth": 100,
  "tabWidth": 2,
  "useTabs": false,
  "semi": true,
  "singleQuote": true,
  "trailingComma": "all",
  "bracketSpacing": true,
  "arrowParens": "always",
  "sortImports": true,
  "sortPackageJson": true,
  "sortTailwindcss": true
}
```

---

## 🚀 Key Advantages Over Prettier

1. **Instant Execution**: Formats thousands of files in milliseconds with multithreaded Rust performance.
2. **Built-in Plugins**: Includes native import sorting (`sortImports`), `package.json` key sorting (`sortPackageJson`), and Tailwind CSS class sorting (`sortTailwindcss`) out of the box without extra Node.js plugins.
3. **Write-First Default**: Defaults to formatting files in place without needing `--write`.
4. **Shared Ignore Engine**: Automatically respects `.gitignore`, `.prettierignore`, and `.editorconfig`.

---

## 📚 References

The following guides are located directly alongside this skill file:

- [CLI Reference](file:///c:/_/skills/skills/oxfmt/cli.md): Output modes (`write`, `--check`, `--list-different`), migration commands, stdin formatting, ignore paths, and runtime flags.
- [Configuration Reference](file:///c:/_/skills/skills/oxfmt/config.md): Formatting parameters, quotes, semicolons, line wrapping, file overrides, ignore patterns, and built-in sorting extensions.
