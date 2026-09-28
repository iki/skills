# Logging with Consola

Consola provides a unified, elegant logging experience across Node.js, browsers, and testing environments with built-in log levels, tags, and spam prevention.

---

## 1. Quick Example

```typescript
import { consola, createConsola } from "consola";

// Standard logging methods
consola.info("Server initialization started");
consola.start("Connecting to database...");
consola.ready("Server listening on port 3000");
consola.success("Build finished successfully!");
consola.warn("Deprecation warning: method will be removed in next release");
consola.error(new Error("Database connection failed"));
consola.fatal("Critical failure: unable to continue execution");

// Debug and trace logs (visible only when log level >= 4 or >= 5)
consola.debug("Detailed debug payload:", { user: "alice", attempts: 3 });
consola.trace("Stack trace information");
```

---

## 2. Package Entrypoints

To optimize bundle size and execution environment:

```typescript
// Full bundle (Fancy reporter in terminal, Basic in CI/Test)
import { consola, createConsola } from "consola";

// Lightweight bundle (Basic text reporter only, saves ~80% bundle size)
import { consola, createConsola } from "consola/basic";

// Browser bundle (Styled with CSS in browser console)
import { consola, createConsola } from "consola/browser";

// Core class (No default reporters, minimal footprint)
import { Consola, createConsola } from "consola/core";
```

---

## 3. Log Levels & Environment Control

Consola suppresses messages whose level exceeds the current `level` threshold. Default level is `3` (`info`, `success`, `warn`, `error`, `fatal`).

| Level | Value | Types Displayed |
|---|---|---|
| `silent` | `-Infinity` | Suppresses all log output |
| `fatal`, `error` | `0` | Only catastrophic failures and errors |
| `warn` | `1` | Errors and warnings |
| `log` | `2` | Normal log messages |
| `info`, `success`, `ready`, `start`, `fail`, `box` | `3` (Default) | Informational and status messages |
| `debug` | `4` | Verbose debug messages |
| `trace` | `5` | Deep stack trace messages |
| `verbose` | `+Infinity` | All possible messages |

### Setting the Log Level

1. **Via Environment Variable**:
   ```bash
   CONSOLA_LEVEL=4 node app.js
   ```
2. **At Runtime**:
   ```typescript
   consola.level = 4; // Enable debug logs
   ```
3. **During Instance Creation**:
   ```typescript
   const logger = createConsola({ level: 4 });
   ```

---

## 4. Scoped Loggers and Tags

Organize logs by module or subsystem using `withTag` (alias `withScope`):

```typescript
import { consola } from "consola";

// Create child loggers with specific tags
const dbLogger = consola.withTag("database");
const authLogger = consola.withTag("auth");

dbLogger.info("Connected to PostgreSQL");
// Output: [database] Connected to PostgreSQL

authLogger.warn("Invalid password attempt for user: admin");
// Output: [auth] Invalid password attempt for user: admin
```

Child loggers inherit reporters, log levels, and configuration from the parent instance.

---

## 5. Raw Output (`raw`)

Each log type includes a `.raw(...)` modifier that outputs unformatted text without icons, prefixes, or timestamps:

```typescript
// Standard formatted log with icon and prefix
consola.success("File uploaded");

// Raw message output without formatting
consola.success.raw("File uploaded directly");
```

---

## 6. Spam Prevention (Throttling)

Consola automatically throttles repetitive log messages to avoid flooding the console output.
- `throttle`: Time window in milliseconds (default: `1000ms`).
- `throttleMin`: Number of identical logs before throttling activates (default: `5`).

Custom instance configuration:
```typescript
const logger = createConsola({
  throttle: 2000,
  throttleMin: 3
});
```
