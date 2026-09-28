# Interactive Prompts Reference

Consola includes an interactive prompt engine powered by `@clack/prompts` accessible via `consola.prompt()`.

---

## 1. Prompt Types

### Text Input (`type: "text"`)

Prompts the user for a text string:

```typescript
import { consola } from "consola";

const name = await consola.prompt("What is your project name?", {
  type: "text",
  placeholder: "my-awesome-app",
  default: "my-app"
});

console.log("Project name:", name);
```

### Confirmation (`type: "confirm"`)

Prompts the user with a Yes/No boolean confirmation:

```typescript
const confirmed = await consola.prompt("Deploy to production?", {
  type: "confirm",
  initial: false // Default selection (false = No)
});

if (confirmed) {
  consola.info("Deploying...");
} else {
  consola.warn("Deployment aborted.");
}
```

### Single Selection (`type: "select"`)

Presents a menu for selecting a single item:

```typescript
const framework = await consola.prompt("Select a framework:", {
  type: "select",
  options: [
    { label: "Nuxt", value: "nuxt", hint: "Vue full-stack framework" },
    { label: "Next.js", value: "next", hint: "React full-stack framework" },
    { label: "Nitro", value: "nitro", hint: "Fast, portable server engine" }
  ],
  initial: "nuxt"
});

console.log("Selected framework:", framework);
```

### Multiple Selection (`type: "multiselect"`)

Allows the user to select multiple options using spacebar:

```typescript
const tools = await consola.prompt("Select developer tools to install:", {
  type: "multiselect",
  options: [
    { label: "TypeScript", value: "typescript" },
    { label: "Oxlint", value: "oxlint" },
    { label: "Oxfmt", value: "oxfmt" },
    { label: "Vitest", value: "vitest" }
  ],
  initial: ["typescript", "oxlint"],
  required: true
});

console.log("Selected tools:", tools);
```

---

## 2. Cancellation Handling (`cancel`)

When the user cancels a prompt (e.g. pressing `Ctrl+C`), Consola resolves the promise according to the `cancel` option:

| Strategy | Behavior |
|---|---|
| `"default"` (Default) | Resolves the promise with the `default` or `initial` value. |
| `"undefined"` | Resolves the promise with `undefined`. |
| `"null"` | Resolves the promise with `null`. |
| `"symbol"` | Resolves with the cancel symbol (`Symbol.for("cancel")`). |
| `"reject"` | Rejects the promise with an error. |

### Example with Explicit Cancellation Check

```typescript
const answer = await consola.prompt("Enter API key:", {
  type: "text",
  cancel: "null"
});

if (answer === null) {
  consola.warn("Prompt was cancelled by user.");
  process.exit(1);
}
```

### Example with Promise Rejection

```typescript
try {
  const result = await consola.prompt("Proceed with migration?", {
    type: "confirm",
    cancel: "reject"
  });
} catch {
  consola.error("Operation cancelled by user.");
}
```
