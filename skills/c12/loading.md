# Loading Configuration with `loadConfig`

`loadConfig` is the core entry point of `c12`. It discovers, parses, validates, and merges configuration files across multiple formats, layers, environments, and remote sources.

---

## 1. Quick Example

```typescript
import { loadConfig } from "c12";

export interface AppConfig {
  port: number;
  host: string;
  debug?: boolean;
}

const { config, configFile, layers, cwd } = await loadConfig<AppConfig>({
  name: "myApp",
  defaults: {
    port: 3000,
    host: "localhost"
  }
});

console.log("Loaded configuration:", config);
console.log("Config file path:", configFile);
```

---

## 2. File Resolution & Supported Formats

When given `name: "myApp"`, `c12` automatically resolves configuration from:
1. `${cwd}/${name}.config.${ext}` or `${cwd}/config.${ext}`
2. `${cwd}/.config/${name}.config.${ext}` or `${cwd}/.config/${name}.${ext}`
3. RC files: `${cwd}/.${name}rc`, workspace root RC, or `~/.${name}rc` (if `globalRc: true`)
4. Nearest `package.json` under key `"myApp"`

### Supported Extensions (`SUPPORTED_EXTENSIONS`)

`c12` parses both executable and static configuration formats without extra boilerplate:
- **TypeScript & JavaScript**: `.ts`, `.mts`, `.cts`, `.js`, `.mjs`, `.cjs`
- **JSON**: `.json`, `.jsonc` (with comments and trailing commas), `.json5`
- **YAML**: `.yaml`, `.yml`
- **TOML**: `.toml`

---

## 3. Resolution Priority (Highest to Lowest)

When multiple configuration sources exist, `c12` merges them using `defu` according to the following strict order of precedence:

1. **`overrides`**: Object passed directly via `options.overrides`.
2. **Main config file**: `${name}.config.${ext}` in CWD or `.config/`.
3. **RC file**: `.${name}rc` in CWD or workspace.
4. **`package.json`**: `"myApp"` property in project `package.json`.
5. **`defaultConfig`**: Dynamic or resolvable config passed to `options.defaultConfig`.
6. **Extended layers (`extends`)**: Base configs inherited recursively.
7. **`defaults`**: Fallback defaults passed to `options.defaults`.

---

## 4. Environment-Specific Overrides

`c12` allows defining environment-specific blocks directly within the configuration file using `$` prefixes:

```typescript
// myApp.config.ts
export default {
  endpoint: "https://api.example.com",
  logging: false,

  $development: {
    endpoint: "http://localhost:8000",
    logging: true
  },

  $test: {
    endpoint: "http://127.0.0.1:9000"
  },

  $production: {
    logging: false
  },

  $env: {
    staging: {
      endpoint: "https://staging-api.example.com"
    }
  }
};
```

### Controlling the Environment

- By default, `envName` reads `process.env.NODE_ENV`.
- Pass a custom environment name or priority array:
  ```typescript
  await loadConfig({
    name: "myApp",
    envName: process.env.APP_ENV || "development"
  });
  ```
- By default, all `$` keys (`$development`, `$env`, etc.) are stripped from the final `config` object (`omit$Keys: true`).

---

## 5. Layered Configurations (`extends`)

Configurations can extend local files, installed packages, or remote Git repositories:

```typescript
// myApp.config.ts
export default {
  extends: [
    // 1. Relative local config
    "./base.config.ts",

    // 2. Installed npm package preset
    "my-shared-preset",

    // 3. Remote Git repository (requires optional peer dependency `giget`)
    "github:org/repo#v1.0.0"
  ],

  // Local overrides apply on top of extended layers
  theme: "dark"
};
```

### Remote Layer Options (`giget`)

```typescript
export default {
  extends: [
    ["github:my-org/shared-configs/packages/app", {
      auth: process.env.GITHUB_TOKEN,
      install: true // Automatically install layer dependencies with nypm
    }]
  ]
};
```

---

## 6. Schema Validation (Standard Schema v1)

`c12` supports [Standard Schema](https://standardschema.dev) (Zod, Valibot, ArkType) out of the box. The validated output replaces `config`, applying defaults and transformations:

```typescript
import { loadConfig } from "c12";
import { z } from "zod";

const AppSchema = z.object({
  port: z.number().default(3000),
  host: z.string().default("localhost"),
  features: z.array(z.string()).default([])
});

const { config } = await loadConfig({
  name: "myApp",
  schema: AppSchema
});

// config is strictly typed as { port: number; host: string; features: string[] }
console.log(config.port);
```

If validation fails, `loadConfig` throws a formatted error detailing invalid paths and error messages.

---

## 7. Dynamic Configuration Functions

Config files can export a function that receives a context object:

```typescript
// myApp.config.ts
export default (context) => {
  return {
    port: context.isDev ? 3000 : 8080,
    apiUrl: context.apiUrl
  };
};
```

Pass the context during loading:

```typescript
await loadConfig({
  name: "myApp",
  context: {
    isDev: process.env.NODE_ENV !== "production",
    apiUrl: "https://api.internal"
  }
});
```

---

## 8. TypeScript Types & `defineConfig`

Create a typed `defineConfig` helper for your library users:

```typescript
// In your library:
import { createDefineConfig } from "c12";

export interface ServerConfig {
  port?: number;
  host?: string;
  ssl?: boolean;
}

export const defineConfig = createDefineConfig<ServerConfig>();
```

```typescript
// In consumer's server.config.ts:
import { defineConfig } from "my-server-lib";

export default defineConfig({
  port: 8080,
  host: "0.0.0.0"
});
```

---

## 9. Options Reference (`LoadConfigOptions`)

| Option | Type | Default | Description |
|---|---|---|---|
| `name` | `string` | `"config"` | Base name of the configuration (used for `${name}.config.*`, `.${name}rc`). |
| `cwd` | `string` | `process.cwd()` | Project root directory. |
| `configFile` | `string` | `${name}.config` | Override default config file base name. |
| `rcFile` | `string \| false` | `.${name}rc` | Override or disable RC file lookup. |
| `globalRc` | `boolean` | `false` | Also load global RC file from user home directory (`~/.${name}rc`). |
| `packageJson` | `boolean \| string \| string[]` | `false` | Load config from `package.json` key (matches `name` if `true`). |
| `dotenv` | `boolean \| DotenvOptions` | `false` | Automatically load `.env` file before loading config. |
| `envName` | `string \| string[] \| false` | `process.env.NODE_ENV` | Environment identifier for `$<envName>` overrides. |
| `defaults` | `T` | `undefined` | Lowest-priority fallback default values. |
| `defaultConfig` | `ResolvableConfig<T>` | `undefined` | Resolvable default configuration. |
| `overrides` | `ResolvableConfig<T>` | `undefined` | Highest-priority config overrides. |
| `schema` | `StandardSchemaV1` | `undefined` | Standard Schema (Zod/Valibot/ArkType) to validate merged config. |
| `omit$Keys` | `boolean` | `true` | Strip `$` environment keys from the final resolved config. |
| `context` | `Record<string, any>` | `{}` | Context object passed to config functions. |
| `merger` | `Function` | `defu` | Custom function used to merge config layers. |
| `extend` | `false \| { extendKey?: string \| string[] }` | `{ extendKey: "extends" }` | Options for layer extension or `false` to disable. |
| `configFileRequired` | `boolean` | `false` | Throw error if no config file is found. |
