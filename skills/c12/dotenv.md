# Environment Variables and Dotenv Reference

`c12` includes built-in environment variable loading with support for variable interpolation, multi-file priority, and container secret resolution (`_FILE` suffix).

---

## 1. Quick Example

### Standalone `setupDotenv` (Mutates `process.env`)

```typescript
import { setupDotenv } from "c12";

// Loads .env and injects variables into process.env
const env = await setupDotenv({
  cwd: process.cwd(),
  fileName: [".env", ".env.local"], // Later files override earlier ones
  interpolate: true,
  resolveFileSecrets: true
});

console.log(process.env.DATABASE_URL);
```

### Standalone `loadDotenv` (Pure Parsing)

```typescript
import { loadDotenv } from "c12";

// Returns parsed key-value pairs without modifying process.env
const env = await loadDotenv({
  cwd: process.cwd(),
  fileName: ".env",
  interpolate: true
});

console.log("Parsed keys:", Object.keys(env));
```

### Integrated with `loadConfig`

Enable `.env` resolution directly within `loadConfig`:

```typescript
import { loadConfig } from "c12";

const { config } = await loadConfig({
  name: "myApp",
  dotenv: true // Or pass DotenvOptions: { fileName: [".env", ".env.production"] }
});
```

---

## 2. Variable Interpolation Syntax

When `interpolate: true` is enabled, variables within `.env` files can reference other variables:

```env
# Basic reference
HOST="localhost"
PORT=3000
API_URL="http://${HOST}:${PORT}/v1"

# Unbraced syntax
DOMAIN=example.com
FULL_URL=https://$DOMAIN/api

# Default fallback if unset or empty (:-):
PORT="${APP_PORT:-8080}"

# Default fallback only if unset (-):
TIMEOUT="${APP_TIMEOUT-5000}"

# Escaped variable (evaluates to literal "${UNRESOLVED}")
LITERAL="\${UNRESOLVED}"
```

### Interpolation Rules
- **`${VAR:-default}`**: Uses `default` if `VAR` is unset OR empty.
- **`${VAR-default}`**: Uses `default` only if `VAR` is unset (empty string is preserved).
- Nested interpolations are supported: `${FALLBACK:-${DEFAULT_HOST:-localhost}}`.
- Unresolved references without defaults are left as-is instead of replaced with empty strings.

---

## 3. Resolving Container File Secrets (`_FILE`)

In containerized environments (Docker Swarm, Kubernetes), secrets are typically mounted as files (e.g. `/run/secrets/db_password`).

When `resolveFileSecrets: true` is enabled, `c12` automatically resolves any environment variable ending in `_FILE`:
1. Reads the file content at the specified path.
2. Trims leading/trailing whitespace and newlines.
3. Assigns the content to the base variable name without the `_FILE` suffix.

```env
# .env
DB_PASSWORD_FILE="/run/secrets/db_password"
```

Result in `process.env`:
```typescript
process.env.DB_PASSWORD === "superSecret123" // Read directly from /run/secrets/db_password
```

---

## 4. `DotenvOptions` Reference

| Option | Type | Default | Description |
|---|---|---|---|
| `cwd` | `string` | `process.cwd()` | Project root directory to search for env files. |
| `fileName` | `string \| string[]` | `".env"` | Name(s) of files to load. Array order enforces priority (later overrides earlier). |
| `interpolate` | `boolean` | `true` in `setupDotenv` | Enables variable interpolation (`${VAR}`, `${VAR:-default}`). |
| `env` | `NodeJS.ProcessEnv` | `process.env` | Target object where environment variables are assigned. |
| `resolveFileSecrets` | `boolean` | `false` | Resolve `_FILE` suffixed paths to their file contents. |
| `override` | `boolean` | `false` | If `true`, `.env` values will overwrite already existing `process.env` values. |
