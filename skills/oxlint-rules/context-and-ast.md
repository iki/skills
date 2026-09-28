# RuleContext and AST Traversal Reference

This document covers accessing the rule context (`Context`), querying source text and tokens via `SourceCode`, and traversing ESTree AST nodes in custom Oxlint rules.

---

## 1. The `Context` Object

Every rule handler receives the `context` object providing metadata, configuration, and reporting methods:

| Property / Method | Type | Description |
|---|---|---|
| `context.id` | `string` | Fully-qualified rule identifier (e.g. `'my-plugin/my-rule'`). |
| `context.filename` | `string` | Absolute path of the file being linted. |
| `context.cwd` | `string` | Current working directory. |
| `context.options` | `unknown[]` | Options array passed to the rule in configuration. |
| `context.settings` | `Settings` | Shared settings object defined in oxlint config. |
| `context.sourceCode` | `SourceCode` | High-level API for querying source text, tokens, and scope. |
| `context.report(...)` | `Function` | Reports a lint diagnostic or autofix. |

---

## 2. Querying `SourceCode`

`context.sourceCode` provides access to the tokenized representation and AST structure of the file:

```typescript
const sourceCode = context.sourceCode;
```

### Text and Line Queries

```typescript
// Complete source text
const fullCode = sourceCode.text;

// Array of source text lines
const lines = sourceCode.lines;

// Source text of a specific node
const nodeText = sourceCode.getText(node);

// AST Program root
const astRoot = sourceCode.ast;
```

### Token Queries

```typescript
// First and last tokens of a node
const first = sourceCode.getFirstToken(node);
const last = sourceCode.getLastToken(node);

// Adjacent tokens
const prev = sourceCode.getTokenBefore(node);
const next = sourceCode.getTokenAfter(node);

// Tokens between two AST nodes
const middleTokens = sourceCode.getTokensBetween(nodeA, nodeB);

// Check if whitespace exists between tokens
const hasSpace = sourceCode.isSpaceBetween(tokenA, tokenB);
```

### Comment Queries

```typescript
// All comments in the file
const allComments = sourceCode.getAllComments();

// Comments surrounding a node
const leading = sourceCode.getCommentsBefore(node);
const trailing = sourceCode.getCommentsAfter(node);
const inside = sourceCode.getCommentsInside(node);
```

### Scope and Variable Queries

```typescript
// Get the scope associated with an AST node
const scope = sourceCode.getScope(node);

// Variables declared by a declaration node
const declaredVars = sourceCode.getDeclaredVariables(node);

// Mark a variable as referenced
sourceCode.markVariableAsUsed('myVar');
```

---

## 3. Scope and Variable Analysis

Oxlint implements standard ESLint-compatible Scope Analysis:

### `Scope` Structure
- `scope.type`: Scope category (`'global'`, `'module'`, `'function'`, `'block'`, `'catch'`, etc.).
- `scope.block`: The AST node that generated the scope.
- `scope.upper`: Parent scope (`null` if global).
- `scope.childScopes`: Array of nested child scopes.
- `scope.variables`: Variables declared directly within this scope.
- `scope.references`: References to variables resolved in or through this scope.

### `Variable` Structure
- `variable.name`: Name string of the variable.
- `variable.identifiers`: AST Identifier nodes where the variable is defined.
- `variable.references`: All `Reference` instances pointing to this variable.
- `variable.defs`: Definition descriptors (`'Variable'`, `'Parameter'`, `'FunctionName'`, etc.).

### `Reference` Structure
- `reference.identifier`: The AST Identifier node of the reference.
- `reference.isRead()`: True if read from.
- `reference.isWrite()`: True if assigned/written to.
- `reference.resolved`: The `Variable` instance this reference resolves to.

---

## 4. AST Visitors and ESTree Nodes

Visitor maps match ESTree node types. Handlers execute during the enter phase by default, or during the exit phase when using the `:exit` suffix:

```typescript
export default defineRule({
  create(context) {
    let callDepth = 0;

    return {
      // Enter phase
      CallExpression(node) {
        callDepth++;
      },

      // Exit phase
      'CallExpression:exit'(node) {
        callDepth--;
      },

      // Specific node checks
      MemberExpression(node) {
        if (!node.computed && node.property.name === 'eval') {
          context.report({
            node,
            message: 'Avoid calling eval property'
          });
        }
      }
    };
  }
});
```
