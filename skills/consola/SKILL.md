---
name: consola
description: >-
  Log messages, display terminal boxes, run interactive prompts, and mock output using consola.
  Use when building CLI tools or application logging, configuring log levels and custom reporters,
  prompting users for input, wrapping console/stdout, or unit testing logs.
---

# consola Skill

`consola` is an elegant, zero-dependency console wrapper and logging toolkit for Node.js and the browser from the UnJS ecosystem. It provides rich terminal formatting, pluggable reporters, interactive CLI prompts, console redirection, log throttling, and mock testing utilities.

---

## ⚡ Quick Reference

### 1. Basic Logging

```typescript
import { consola } from "consola";

consola.info("Starting task runner...");
consola.start("Compiling TypeScript sources...");
consola.success("Build completed in 120ms!");
consola.ready("Server listening at http://localhost:3000");
consola.warn("Port 3000 busy, falling back to 3001");
consola.error(new Error("Failed to load plugin"));
consola.fatal("Process aborted due to fatal error");

// Bordered terminal callout box
consola.box("Deployment ready!\nVisit http://localhost:3000 to preview.");
```

### 2. Scoped Loggers (`withTag`)

```typescript
// Create namespaced child loggers
const authLogger = consola.withTag("auth");
const dbLogger = consola.withTag("database");

authLogger.info("User session verified");
// Output: [auth] User session verified

dbLogger.error("Connection pool exhausted");
// Output: [database] Connection pool exhausted
```

### 3. Interactive Prompts (`prompt`)

Built-in interactive CLI prompts powered by `@clack/prompts`:

```typescript
import { consola } from "consola";

// Text input
const name = await consola.prompt("Project name:", { type: "text" });

// Confirmation (Yes / No)
const proceed = await consola.prompt("Deploy to production?", {
  type: "confirm",
  initial: false
});

// Single selection menu
const framework = await consola.prompt("Choose framework:", {
  type: "select",
  options: ["Nuxt", "Next.js", "Nitro"]
});

// Multiple selection menu
const plugins = await consola.prompt("Select plugins:", {
  type: "multiselect",
  options: ["TypeScript", "Oxlint", "Oxfmt"]
});
```

### 4. Log Levels

Set the minimum threshold for displayed logs:

```typescript
// 0: fatal/error, 1: warn, 2: log, 3: info/success (default), 4: debug, 5: trace
consola.level = 4; // Enable debug logs
```

Or via environment variable:
```bash
CONSOLA_LEVEL=4 node app.js
```

### 5. Console & Stream Redirection

Intercept global `console` and `process.stdout`/`stderr` calls through Consola's pipeline:

```typescript
import { consola } from "consola";

// Intercept both console and std
consola.wrapAll();

console.log("Automatically routed through Consola!");

// Restore original behavior
consola.restoreAll();
```

### 6. Unit Testing & Mocking (`mockTypes`)

```typescript
import { describe, it, expect, beforeEach, vi } from "vitest";
import { consola } from "consola";

describe("CLI logger", () => {
  const logMock = vi.fn();

  beforeEach(() => {
    logMock.mockClear();
    consola.mockTypes(() => logMock);
  });

  it("logs operation success", () => {
    myService.run();
    expect(logMock).toHaveBeenCalledWith("Operation finished");
  });
});
```

### 7. Terminal Utilities (`consola/utils`)

```typescript
import { colors, box, formatTree } from "consola/utils";

console.log(colors.cyan("Custom color:"), colors.bold("Active"));

const tree = formatTree([
  "src",
  { text: "components", children: ["Button.tsx", "Modal.tsx"] }
]);
console.log(tree);
```

---

## 📚 References

The following guides are located directly alongside this skill file:

- [Logging Reference](file:///c:/_/skills/skills/consola/logging.md): Log methods (`info`, `success`, `warn`, `error`, `fatal`, `debug`), log level thresholds, raw output, bundle subpaths, and spam throttling.
- [Prompts Reference](file:///c:/_/skills/skills/consola/prompts.md): Interactive CLI prompts (`text`, `confirm`, `select`, `multiselect`), cancel strategies (`default`, `null`, `reject`), and placeholders.
- [Reporters Reference](file:///c:/_/skills/skills/consola/reporters.md): Built-in reporters (`FancyReporter`, `BasicReporter`, `BrowserReporter`), custom reporter implementations, and JSON stream integration.
- [Redirection & Testing Reference](file:///c:/_/skills/skills/consola/redirection-and-testing.md): Stream interception (`wrapConsole`, `wrapStd`, `wrapAll`), log pausing/resuming (`pauseLogs`), and unit testing with Vitest/Jest via `mockTypes`.
- [Utils Reference](file:///c:/_/skills/skills/consola/utils.md): Terminal formatting utilities (`box`, `colors`, `colorize`, `formatTree`, `stripAnsi`, and string alignment).