# Console Utilities (`consola/utils`) Reference

The `consola/utils` submodule exports terminal formatting tools including text boxes, color functions, tree rendering, and string alignment helpers.

```typescript
import {
  box,
  colors,
  colorize,
  getColor,
  formatTree,
  stripAnsi,
  centerAlign,
  rightAlign,
  leftAlign
} from "consola/utils";
```

---

## 1. Terminal Boxes (`box`)

Renders a bordered box around text:

```typescript
import { box } from "consola/utils";
// Or directly via consola:
import { consola } from "consola";

consola.box("Deployment ready!\nVisit http://localhost:3000 to view.");

// With custom styling:
const renderedBox = box("Notice:\nDatabase migration completed.", {
  title: "Migration",
  padding: 1,
  borderColor: "green",
  style: {
    tl: "╭",
    tr: "╮",
    bl: "╰",
    br: "╯",
    h: "─",
    v: "│"
  }
});

console.log(renderedBox);
```

### `BoxOpts` Options
- `title?: string`: Header title printed along the top border.
- `padding?: number`: Padding inside the box (default: `1`).
- `borderColor?: ColorName`: Color applied to the borders.
- `style?: BoxBorderStyle`: Characters used for corners and lines.

---

## 2. ANSI Colors and Styling (`colors`)

Consola re-exports ANSI color functions with automatic color support detection:

```typescript
import { colors, colorize } from "consola/utils";

console.log(colors.cyan("Info:"), colors.bold("Operation successful"));
console.log(colors.yellow("Warning:"), colors.underline("Deprecated API"));
console.log(colors.red("Error:"), colors.bgRed(colors.white(" FATAL ")));

// Dynamic color application
const colored = colorize("green", "Completed in 42ms");
```

---

## 3. Tree Rendering (`formatTree`)

Formats hierarchical objects or directory structures into readable ASCII tree diagrams:

```typescript
import { formatTree } from "consola/utils";

const treeOutput = formatTree([
  "src",
  {
    text: "components",
    children: ["Header.tsx", "Footer.tsx"]
  },
  {
    text: "utils",
    children: ["index.ts", "math.ts"]
  },
  "package.json"
]);

console.log(treeOutput);
/*
Output:
├── src
├── components
│   ├── Header.tsx
│   └── Footer.tsx
├── utils
│   ├── index.ts
│   └── math.ts
└── package.json
*/
```

---

## 4. String Manipulation Helpers

### `stripAnsi`
Removes ANSI escape codes from strings (essential for calculating actual printed column length):

```typescript
import { stripAnsi, colors } from "consola/utils";

const coloredText = colors.red("Hello World");
const plainText = stripAnsi(coloredText); // "Hello World"
```

### Alignment Helpers (`centerAlign`, `leftAlign`, `rightAlign`, `align`)

```typescript
import { centerAlign, rightAlign, leftAlign } from "consola/utils";

centerAlign("Title", 20); // "       Title        "
leftAlign("Left", 10);    // "Left      "
rightAlign("Right", 10);  // "     Right"
```
