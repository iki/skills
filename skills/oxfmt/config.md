# Oxfmt Configuration Reference

Oxfmt reads configuration primarily from `.oxfmtrc.json` or `.oxfmtrc.jsonc`. It also respects standard `.editorconfig` settings and `.prettierignore` files.

---

## 1. Example `.oxfmtrc.json`

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
  "endOfLine": "lf",
  "sortImports": true,
  "sortPackageJson": true,
  "sortTailwindcss": true,
  "ignorePatterns": [
    "dist/**",
    "coverage/**",
    "**/*.min.js"
  ],
  "overrides": [
    {
      "files": ["*.md", "*.mdx"],
      "options": {
        "proseWrap": "always"
      }
    }
  ]
}
```

---

## 2. Formatting Options

### Core Layout Options

| Option | Type | Default | Description |
|---|---|---|---|
| `printWidth` | `number` | `80` | Line length threshold for wrapping. |
| `tabWidth` | `number` | `2` | Number of spaces per indentation level. |
| `useTabs` | `boolean` | `false` | Indent with tabs instead of spaces. |
| `endOfLine` | `"lf" \| "crlf" \| "cr"` | `"lf"` | Line ending characters to apply. |
| `insertFinalNewline`| `boolean` | `true` | Ensure files end with a trailing newline. |

### Quotes and Punctuation

| Option | Type | Default | Description |
|---|---|---|---|
| `semi` | `boolean` | `true` | Print semicolons at the ends of statements. |
| `singleQuote` | `boolean` | `false` | Use single quotes (`'`) instead of double quotes (`"`). |
| `jsxSingleQuote` | `boolean` | `false` | Use single quotes in JSX attributes. |
| `quoteProps` | `"as-needed" \| "consistent" \| "preserve"` | `"as-needed"` | Quotes around object literal property keys. |
| `trailingComma` | `"all" \| "es5" \| "none"` | `"all"` | Trailing commas in multi-line lists/tuples/objects. |

### Syntax & Spacing

| Option | Type | Default | Description |
|---|---|---|---|
| `bracketSpacing` | `boolean` | `true` | Spaces inside object literal brackets `{ foo: bar }`. |
| `bracketSameLine` | `boolean` | `false` | Place `>` of multi-line HTML/JSX element at end of last line. |
| `arrowParens` | `"always" \| "avoid"` | `"always"` | Include parentheses around a sole arrow function parameter `(x) => x`. |
| `singleAttributePerLine` | `boolean` | `false` | Enforce single HTML/JSX attribute per line. |
| `objectWrap` | `"preserve" \| "collapse"` | `"preserve"` | How object literal line wrapping is maintained. |
| `experimentalOperatorPosition` | `"start" \| "end"` | `"end"` | Place operators at line start (`start`) or end (`end`) on wrap. |

### HTML & Framework Options

| Option | Type | Default | Description |
|---|---|---|---|
| `htmlWhitespaceSensitivity` | `"css" \| "strict" \| "ignore"` | `"css"` | Global whitespace sensitivity for HTML, Vue, and Angular. |
| `vueIndentScriptAndStyle` | `boolean` | `false` | Indent code inside `<script>` and `<style>` in Vue files. |
| `embeddedLanguageFormatting` | `"auto" \| "off"` | `"auto"` | Format code embedded in Markdown, HTML, or template strings. |
| `proseWrap` | `"always" \| "never" \| "preserve"` | `"preserve"` | How prose text is wrapped in Markdown. |

---

## 3. Oxfmt Extensions (Built-in Optimizations)

Oxfmt incorporates features that traditionally required external Prettier plugins:

### `sortImports`
Automatically sorts and groups import declarations according to standard ECMAScript conventions.
```json
{
  "sortImports": true
}
```

### `sortPackageJson`
Automatically organizes and sorts `package.json` keys according to standard community specifications (e.g. `name`, `version`, `scripts`, `dependencies`).
```json
{
  "sortPackageJson": true
}
```

### `sortTailwindcss`
Automatically sorts Tailwind CSS utility classes inside `class` and `className` attributes without requiring the `prettier-plugin-tailwindcss` npm dependency.
```json
{
  "sortTailwindcss": true
}
```

### `jsdoc`
Formats JSDoc comment blocks and alignments.
```json
{
  "jsdoc": true
}
```

---

## 4. Scopes and Overrides

Configure distinct options for specific file types or subdirectories:

```json
{
  "singleQuote": true,
  "overrides": [
    {
      "files": ["*.json", "*.json5"],
      "options": {
        "tabWidth": 4
      }
    },
    {
      "files": ["docs/**/*.md"],
      "options": {
        "printWidth": 120,
        "proseWrap": "always"
      }
    }
  ]
}
```

---

## 5. Ignoring Files

Files can be excluded from formatting using:
1. `ignorePatterns` array inside `.oxfmtrc.json`:
   ```json
   {
     "ignorePatterns": ["dist/**", "generated/**"]
   }
   ```
2. `.prettierignore` or `.oxfmtignore` files placed in project root.
3. Standard `.gitignore` file (respected by default).
