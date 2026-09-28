# Watching Configuration with `watchConfig`

`watchConfig` watches configuration files, extended layers, `.env` files, and RC files for changes, automatically reloading and calculating diffs for Hot Module Replacement (HMR).

---

## 1. Quick Example

```typescript
import { watchConfig } from "c12";

const watcher = await watchConfig({
  name: "myApp",
  chokidar: {}, // Optional watcher options
  onUpdate({ newConfig, oldConfig, getDiff }) {
    console.log("Configuration updated!");
    const diffs = getDiff();
    console.log("Changes:", diffs);
  }
});

// Access current config
console.log("Current port:", watcher.config.port);

// Access watched file list
console.log("Watching files:", watcher.watchingFiles);

// Stop watching when shutting down
// await watcher.unwatch();
```

---

## 2. Watching Lifecycle Hooks

`watchConfig` supports all options from `loadConfig`, plus specialized event callbacks:

### `onWatch(event)`
Called on low-level filesystem events before configuration is reloaded:

```typescript
await watchConfig({
  name: "myApp",
  onWatch(event) {
    console.log(`File ${event.type}: ${event.path}`); // 'created' | 'updated' | 'removed'
  }
});
```

### `acceptHMR(context)`
Determines whether an update can be applied in-place (hot reload) without restarting the entire server or application:

```typescript
await watchConfig({
  name: "myApp",
  acceptHMR({ newConfig, oldConfig, getDiff }) {
    const diffs = getDiff();

    // If only 'theme' changed, accept hot reload
    const isThemeOnly = diffs.every(d => d.key === "theme");
    return isThemeOnly; // Return true to accept HMR, false to trigger full restart
  }
});
```

### `onUpdate(context)`
Called whenever configuration has been reloaded and successfully merged:

```typescript
await watchConfig({
  name: "myApp",
  onUpdate({ newConfig, oldConfig, getDiff }) {
    const diffs = getDiff();
    for (const change of diffs) {
      console.log(`Key ${change.key} changed:`, change.oldValue, "->", change.value);
    }
  }
});
```

---

## 3. Working with Diffs (`getDiff()`)

The `getDiff()` utility function computes deep differences between `oldConfig` and `newConfig`:

```typescript
onUpdate({ getDiff }) {
  const diffs = getDiff();
  /*
  Example diff output:
  [
    {
      type: "changed",
      key: "port",
      oldValue: 3000,
      value: 8080
    },
    {
      type: "added",
      key: "features.auth",
      value: true
    }
  ]
  */
}
```

---

## 4. Debouncing

File changes are automatically debounced to prevent repeated reloads when multiple files are saved simultaneously:

```typescript
await watchConfig({
  name: "myApp",
  debounce: 100 // Delay in milliseconds (or `false` to disable)
});
```

---

## 5. Cleaning Up (`unwatch`)

Always stop watchers during test teardown or graceful process shutdown:

```typescript
process.on("SIGINT", async () => {
  await watcher.unwatch();
  process.exit(0);
});
```
