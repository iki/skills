# Console Redirection, Log Pausing, and Testing Reference

Consola provides utilities to intercept global `console` and standard I/O calls, pause/resume logs, and mock logger methods in unit test suites.

---

## 1. Global Stream Redirection

Redirect native `console` and standard output/error calls through Consola to enforce consistent styling and reporter forwarding:

### Redirecting `console`

```typescript
import { consola } from "consola";

// Intercept console.log, console.warn, console.error, etc.
consola.wrapConsole();

console.log("This will now be formatted by Consola!");

// Restore original console behavior
consola.restoreConsole();
```

### Redirecting `stdout` and `stderr`

```typescript
// Intercept raw writes to process.stdout and process.stderr
consola.wrapStd();

// Restore native streams
consola.restoreStd();
```

### Redirecting Everything (`wrapAll`)

```typescript
// Intercept both console and standard streams
consola.wrapAll();

// Restore everything
consola.restoreAll();
```

---

## 2. Pausing and Resuming Logs

When performing critical tasks (such as interactive CLI prompts, progress bars, or sensitive operations), you can temporarily pause logging. Consola queues all log events in memory and flushes them when resumed:

```typescript
import { consola } from "consola";

// Pause all logging output; logs are held in memory
consola.pauseLogs();

// These will not print immediately
consola.info("Hidden task 1 in progress...");
consola.warn("Hidden warning...");

// Perform interactive prompt or animation here...

// Resume logging; all queued messages are flushed to reporters in order
consola.resumeLogs();
```

---

## 3. Unit Testing with Vitest and Jest (`mockTypes`)

Consola allows mocking all logger methods cleanly across test suites:

### Vitest Example

```typescript
import { describe, it, expect, beforeEach } from "vitest";
import { consola } from "consola";

describe("Application logging test", () => {
  let logMock = vi.fn();

  beforeEach(() => {
    logMock = vi.fn();
    // Mock all log methods to route to the mock function
    consola.mockTypes(() => logMock);
  });

  it("emits expected warning", () => {
    myFunction();

    // Verify calls
    expect(logMock).toHaveBeenCalledWith("Warning message");
  });
});
```

### Mocking Specific Log Types Only

Return a function for types you wish to mock, or return a falsy value to leave the type unmocked:

```typescript
import { consola } from "consola";

// Only mock 'error' and 'fatal'
consola.mockTypes((typeName) => {
  if (typeName === "error" || typeName === "fatal") {
    return vi.fn();
  }
  return false;
});
```

### Scoped Logger Inheritance

Mocks established on a parent Consola instance automatically propagate to child loggers created via `withTag`:

```typescript
consola.mockTypes(() => vi.fn());

const childLogger = consola.withTag("auth");
// childLogger.info is also automatically mocked!
```
