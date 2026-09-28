---
name: c12
description: >-
  Load, watch, and update layered configuration files in JavaScript and TypeScript projects using c12.
  Use when integrating configuration loading (loadConfig), multi-file formats (.ts, .js, .json, .yaml, .toml),
  dotenv environment loading, schema validation, live config watching (watchConfig), and AST updates (updateConfig).
---

# c12 Skill

`c12` is a smart, zero-config configuration loader for JavaScript and TypeScript projects created by the UnJS ecosystem. It unifies configuration loading from files, RC files, `package.json`, environment variables, and remote Git repositories with deep merging, schema validation, live watching, and AST-preserving updates.

---

## ⚡ Quick Reference

### 1. Basic Configuration Loading (`loadConfig`)

```typescript
import { loadConfig } from "c12";

export interface MyConfig {
  port: number;
  host: string;
  theme?: string;
}

const { config, configFile, layers } = await loadConfig<MyConfig>({
  name: "myApp", // Resolves myApp.config.{ts,js,json,yaml,toml}, .myApprc, package.json "myApp"
  defaults: {
    port: 3000,
    host: "localhost"
  }
});

console.log("Active port:", config.port);
console.log("Loaded from:", configFile);
```

### 2. Multi-Environment Configuration File

`c12` natively resolves environment-specific blocks matching `process.env.NODE_ENV` or custom `envName`:

```typescript
// myApp.config.ts
export default {
  port: 3000,
  host: "localhost",
  debug: false,

  $development: {
    debug: true
  },

  $production: {
    port: 8080
  },

  $env: {
    staging: {
      host: "staging.example.com"
    }
  }
};
```

### 3. Schema Validation with Standard Schema (Zod / Valibot)

```typescript
import { loadConfig } from "c12";
import { z } from "zod";

const ConfigSchema = z.object({
  port: z.number().default(3000),
  host: z.string().default("localhost"),
  apiKey: z.string()
});

const { config } = await loadConfig({
  name: "myApp",
  schema: ConfigSchema // Standard Schema v1 (Zod, Valibot, ArkType)
});
```

### 4. Dotenv Loading with Interpolation & Container Secrets

```typescript
import { setupDotenv } from "c12";

// Standalone loading with variable interpolation and _FILE secret resolution
await setupDotenv({
  cwd: process.cwd(),
  fileName: [".env", ".env.local"],
  interpolate: true,
  resolveFileSecrets: true // Reads /run/secrets/db_pass from DB_PASS_FILE
});

// Or enable directly inside loadConfig:
const { config } = await loadConfig({
  name: "myApp",
  dotenv: true
});
```

### 5. Live Watching & Hot Module Replacement (`watchConfig`)

```typescript
import { watchConfig } from "c12";

const watcher = await watchConfig({
  name: "myApp",
  onUpdate({ newConfig, oldConfig, getDiff }) {
    console.log("Configuration updated!");
    const diffs = getDiff();
    console.log("Diffs:", diffs);
  }
});

// Stop watching
// await watcher.unwatch();
```

### 6. Programmatically Updating Config Files (`updateConfig`)

Uses AST modification via `magicast` to preserve comments and indentation:

```typescript
import { updateConfig } from "c12/update";

await updateConfig({
  cwd: process.cwd(),
  configFile: "myApp.config",
  onUpdate(config) {
    config.port = 8080;
    config.features = config.features || {};
    config.features.auth = true;
  }
});
```

---

## 🏗️ Resolution Precedence (High → Low)

1. **`overrides`**: Directly passed in options.
2. **Main config file**: `${name}.config.${ext}` in CWD or `.config/`.
3. **RC files**: `.${name}rc` in CWD or workspace root.
4. **`package.json`**: `"myApp"` property.
5. **`defaultConfig`**: Dynamic or resolvable config in options.
6. **Extended layers (`extends`)**: Base configs inherited recursively.
7. **`defaults`**: Fallback defaults in options.

---

## 📚 References

The following guides are located directly alongside this skill file:

- [Loading Reference](file:///c:/_/skills/skills/c12/loading.md): Complete guide to `loadConfig`, file formats, layer merging (`defu`), environment overrides (`$env`), remote layers (`giget`), schema validation, and `defineConfig`.
- [Dotenv Reference](file:///c:/_/skills/skills/c12/dotenv.md): Built-in environment variable handling with `setupDotenv`/`loadDotenv`, variable interpolation syntax, defaults, and container file secret resolution.
- [Watching Reference](file:///c:/_/skills/skills/c12/watching.md): Real-time config watching with `watchConfig`, lifecycle hooks (`onWatch`, `onUpdate`, `acceptHMR`), debouncing, and deep diffing.
- [Updating Reference](file:///c:/_/skills/skills/c12/updating.md): AST-preserving configuration modification using `updateConfig` (`magicast`) and automatic file scaffolding.