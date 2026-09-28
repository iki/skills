# Diagnostics, Autofixes, and Suggestions Reference

This document covers reporting rule violations, configuring message IDs, creating automated code fixes with `fixer`, and providing interactive suggestions.

---

## 1. Reporting Diagnostics (`context.report`)

Violations are emitted via `context.report`:

```typescript
context.report({
  node,
  message: 'Unexpected console.log statement.'
});
```

### Report Options

| Field | Type | Description |
|---|---|---|
| `node` | `ESTree.Node` | The AST node associated with the issue. |
| `message` | `string` | Human-readable explanation of the violation. |
| `messageId` | `string` | Key defined in `meta.messages` for standardized messages. |
| `data` | `Record<string, string>` | Data object used to interpolate variables in `messageId`. |
| `loc` | `Location \| LineColumn` | Explicit source location override. |
| `fix` | `(fixer: Fixer) => Fix \| Fix[]` | Automatic code repair function. |
| `suggest` | `Suggestion[]` | Interactive suggestions displayed in IDEs. |

### Message Interpolation

Define reusable messages with interpolation placeholders (`{{ variable }}`) in `meta.messages`:

```typescript
export default defineRule({
  meta: {
    messages: {
      avoidIdentifier: "Avoid using '{{ name }}' in this scope."
    }
  },
  create(context) {
    return {
      Identifier(node) {
        if (node.name === 'forbidden') {
          context.report({
            node,
            messageId: 'avoidIdentifier',
            data: {
              name: node.name
            }
          });
        }
      }
    };
  }
});
```

---

## 2. Autofixes (`fixer`)

An autofix is executed when `--fix` is passed to Oxlint. Fixes must be completely deterministic and preserve program correctness.

```typescript
context.report({
  node,
  message: 'Use const instead of var',
  fix(fixer) {
    return fixer.replaceText(node, 'const');
  }
});
```

### `Fixer` Methods

| Method | Description | Example |
|---|---|---|
| `replaceText(node, text)` | Replaces the source text of the node or token. | `fixer.replaceText(node, 'const')` |
| `replaceTextRange([start, end], text)` | Replaces characters between `start` and `end`. | `fixer.replaceTextRange([0, 3], 'let')` |
| `insertTextBefore(node, text)` | Inserts text immediately before the node. | `fixer.insertTextBefore(node, 'async ')` |
| `insertTextAfter(node, text)` | Inserts text immediately after the node. | `fixer.insertTextAfter(node, ';')` |
| `insertTextBeforeRange([start, end], text)` | Inserts text before the given offset range. | `fixer.insertTextBeforeRange(range, '(')` |
| `insertTextAfterRange([start, end], text)` | Inserts text after the given offset range. | `fixer.insertTextAfterRange(range, ')')` |
| `remove(node)` | Completely removes the node or token from source code. | `fixer.remove(node)` |
| `removeRange([start, end])` | Deletes the character range from source code. | `fixer.removeRange([start, end])` |

### Composing Multiple Edits

Return an array of fix objects to apply multiple changes atomically:

```typescript
context.report({
  node,
  message: 'Wrap expression in parenthesis',
  fix(fixer) {
    return [
      fixer.insertTextBefore(node, '('),
      fixer.insertTextAfter(node, ')')
    ];
  }
});
```

---

## 3. Interactive Suggestions (`suggest`)

Suggestions are used when a problem can be solved in multiple ways or when an automated fix might alter runtime semantics. Suggestions are shown in editor quick-fix menus and can be applied via `--fix-suggestions`.

```typescript
context.report({
  node,
  message: "Variable 'x' is reassigned but declared with const.",
  suggest: [
    {
      desc: "Change declaration to 'let'",
      fix(fixer) {
        return fixer.replaceText(declarationToken, 'let');
      }
    },
    {
      desc: 'Remove reassignment',
      fix(fixer) {
        return fixer.remove(assignmentNode);
      }
    }
  ]
});
```
