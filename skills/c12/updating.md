# Updating Configuration with `updateConfig`

`updateConfig` provides an AST-aware utility for programmatically creating or updating JavaScript and TypeScript configuration files while preserving comments, formatting, and indentation.

---

## 1. Quick Example

```typescript
import { updateConfig } from "c12/update";

const result = await updateConfig({
  cwd: process.cwd(),
  configFile: "myApp.config",
  onUpdate(config) {
    // Modify AST directly
    config.port = 8080;
    config.features = config.features || {};
    config.features.auth = true;
  }
});

console.log("Updated config file:", result.configFile);
console.log("Was newly created?", result.created);
```

---

## 2. Preserving Existing Formatting & Comments

`updateConfig` uses `magicast` to modify the Abstract Syntax Tree (AST) rather than serializing plain JSON. This ensures:
- Existing comments are kept intact.
- Code style, quote preferences, and indentation are preserved.
- Default export structures (like `export default defineConfig({ ... })` or `export default { ... }`) are modified seamlessly.

---

## 3. Handling File Creation (`onCreate`)

If no configuration file exists, `updateConfig` can automatically create one:

```typescript
await updateConfig({
  cwd: process.cwd(),
  configFile: "myApp.config",
  createExtension: ".ts", // Creates myApp.config.ts if missing
  onCreate({ configFile }) {
    // Return template string, true for default 'export default {}\n', or false to abort
    return `import { defineConfig } from "myApp";\n\nexport default defineConfig({});\n`;
  },
  onUpdate(config) {
    config.theme = "dark";
  }
});
```

---

## 4. `UpdateConfigOptions` Reference

| Option | Type | Default | Description |
|---|---|---|---|
| `cwd` | `string` | *(Required)* | Project working directory containing the config file. |
| `configFile` | `string` | *(Required)* | Name or base name of the config file. |
| `createExtension` | `string` | `".ts"` | Extension to use when creating a new file (`.ts`, `.js`, `.mjs`, etc.). |
| `onUpdate` | `(config: any) => void \| Promise<void>` | `undefined` | Callback receiving the AST config object for modification. |
| `onCreate` | `(ctx: { configFile: string }) => string \| boolean \| Promise<...>` | `undefined` | Hook called before creating a new config file. Return template or `false` to abort. |
