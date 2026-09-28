# Consola Reporters Reference

Reporters control how log messages are formatted, transformed, and delivered to their final destination (terminal, file, remote monitoring, etc.).

---

## 1. Built-in Reporters

Consola includes three native reporters:

1. **`FancyReporter`**:
   - Used by default in interactive terminal environments.
   - Outputs colored badges, status icons, timestamps, and formatted errors.
2. **`BasicReporter`**:
   - Used in minimal environments, CI/CD pipelines, or test runners.
   - Clean, lightweight text without complex ANSI codes or icons.
3. **`BrowserReporter`**:
   - Used in browser environments (via `consola/browser`).
   - Uses browser `%c` CSS styling in `console.log`.

---

## 2. Managing Reporters

```typescript
import { consola } from "consola";

// Add a new reporter alongside existing ones
consola.addReporter(myReporter);

// Remove a specific reporter
consola.removeReporter(myReporter);

// Clear all reporters
consola.removeReporter();

// Replace all reporters with a new list
consola.setReporters([new CustomReporter()]);
```

---

## 3. Creating Custom Reporters

A reporter is any object implementing the `ConsolaReporter` interface:

```typescript
export interface ConsolaReporter {
  log: (logObj: LogObject, context: { options: ConsolaOptions }) => void;
}
```

### The `LogObject` Structure

| Field | Type | Description |
|---|---|---|
| `logObj.type` | `LogType` | Type of log (`"info"`, `"warn"`, `"error"`, `"fatal"`, etc.). |
| `logObj.level` | `number` | Numeric log level (0 to 5). |
| `logObj.args` | `any[]` | Array of arguments passed to the log function. |
| `logObj.date` | `Date` | Timestamp when the log was emitted. |
| `logObj.tag` | `string` | Tag or scope attached to the logger (empty string if untagged). |

---

## 4. Custom Reporter Examples

### JSON Reporter (for Cloud Logging & Observability)

```typescript
import { createConsola } from "consola";

const logger = createConsola({
  reporters: [
    {
      log(logObj) {
        const payload = {
          timestamp: logObj.date.toISOString(),
          severity: logObj.type.toUpperCase(),
          level: logObj.level,
          tag: logObj.tag || undefined,
          message: logObj.args.map(arg => typeof arg === "object" ? JSON.stringify(arg) : String(arg)).join(" ")
        };
        process.stdout.write(JSON.stringify(payload) + "\n");
      }
    }
  ]
});

logger.info("Application started", { port: 3000 });
// Outputs: {"timestamp":"2026-09-28T18:00:00.000Z","severity":"INFO","level":3,"message":"Application started {\"port\":3000}"}
```

### Process Exit on Fatal Error Reporter

```typescript
import { consola } from "consola";

consola.addReporter({
  log(logObj) {
    if (logObj.type === "fatal") {
      process.stderr.write("Fatal error occurred. Exiting process.\n");
      process.exit(1);
    }
  }
});

// Logs error and triggers immediate process.exit(1)
consola.fatal("Unable to start server");
```

### File Stream Reporter

```typescript
import { createWriteStream } from "node:fs";
import { consola } from "consola";

const logFile = createWriteStream("app.log", { flags: "a" });

consola.addReporter({
  log(logObj) {
    const formatted = `[${logObj.date.toISOString()}] [${logObj.type}] ${logObj.args.join(" ")}\n`;
    logFile.write(formatted);
  }
});
```
