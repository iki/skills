# SourceCode and Scope Reference

This document covers inspecting text, tokens, comments, and lexical scopes using `SourceCode.*`, `Scope.*`, `Comment.*`, and `Token.*`.

---

## 1. `SourceCode.*` Module

The `SourceCode` module provides helpers for querying source text, AST positions, tokens, and comments. Functions that query global file state return an `Effect` requiring `RuleContext`.

```typescript
import * as SourceCode from 'effect-oxlint/SourceCode';
```

### Text and Location Queries

| Function | Signature | Description |
|---|---|---|
| `getText()` | `Effect<string, never, RuleContext>` | Returns the complete source text of the file being linted. |
| `getLines()` | `Effect<ReadonlyArray<string>, never, RuleContext>` | Returns all source lines as an array of strings. |
| `getNodeText(node)` | `(node: Ranged) => string` | Returns the source text corresponding to an AST node or token. |
| `getRange(node)` | `(node: Ranged) => readonly [number, number]` | Returns the character range `[start, end]` of a node. |
| `getLocFromIndex(index)` | `(index: number) => Location` | Translates a character index into line/column coordinates. |
| `getIndexFromLoc(loc)` | `(loc: LineColumn) => number` | Translates line/column coordinates into a character index. |
| `getNodeByRangeIndex(index)` | `(index: number) => Option<ESTree.Node>` | Finds the AST node at a given character offset. |
| `getAncestors(node)` | `(node: ESTree.Node) => ReadonlyArray<ESTree.Node>` | Returns the array of ancestor nodes from root down to `node`. |

### Token Queries

| Function | Description |
|---|---|
| `getFirstToken(node)` | First token inside the node. |
| `getLastToken(node)` | Last token inside the node. |
| `getTokens(node)` | Array of all tokens inside the node. |
| `getTokenBefore(node)` | Token immediately preceding the node. |
| `getTokenAfter(node)` | Token immediately following the node. |
| `getTokensBetween(nodeA, nodeB)` | All tokens located between two nodes. |
| `getFirstTokenBetween(nodeA, nodeB)` | The first token located between two nodes. |
| `isSpaceBetween(first, second)` | True if whitespace or a newline separates the two tokens/nodes. |

### Comment Queries

| Function | Description |
|---|---|
| `getAllComments()` | `Effect<ReadonlyArray<Comment>, never, RuleContext>`: Returns all comments in the file. |
| `getCommentsBefore(nodeOrToken)` | Comments immediately preceding the node/token. |
| `getCommentsAfter(nodeOrToken)` | Comments immediately following the node/token. |
| `getCommentsInside(node)` | Comments placed inside the bounds of the node. |
| `commentsExistBetween(a, b)` | Returns `true` if any comment exists between `a` and `b`. |
| `getJSDocComment(node)` | Returns `Option.some(Comment)` if a JSDoc block comment precedes the node. |

---

## 2. `Scope.*` Module

The `Scope` module allows inspection of JavaScript/TypeScript variable declarations, references, and scope hierarchies.

```typescript
import * as Scope from 'effect-oxlint/Scope';
```

### Variable Lookup

- **`Scope.findVariable(scope, name)`**: Searches the immediate scope for a variable with `name`. Returns `Option.Option<Variable>`.
- **`Scope.findVariableUp(scope, name)`**: Recursively searches the current scope and all ancestor parent scopes. Returns `Option.Option<Variable>`.

```typescript
import * as Option from 'effect/Option';
import * as Scope from 'effect-oxlint/Scope';
import * as SourceCode from 'effect-oxlint/SourceCode';

const currentScope = SourceCode.getScope(node);
const maybeVar = Scope.findVariableUp(currentScope, 'myService');

Option.match(maybeVar, {
	onNone: () => Effect.void,
	onSome: (variable) => {
		if (!Scope.isUsed(variable)) {
			// Variable declared but never referenced
		}
	}
});
```

### Variable Inspection

| Function | Return Type | Description |
|---|---|---|
| `Scope.isUsed(variable)` | `boolean` | True if the variable has at least one reference. |
| `Scope.isWritten(variable)` | `boolean` | True if the variable is written/assigned to. |
| `Scope.isReadOnly(variable)` | `boolean` | True if the variable is never mutated or reassigned. |
| `Scope.getReferences(variable)` | `ReadonlyArray<Reference>` | All AST references to the variable. |
| `Scope.getReadReferences(variable)` | `ReadonlyArray<Reference>` | All read-only references to the variable. |
| `Scope.getWriteReferences(variable)` | `ReadonlyArray<Reference>` | All write/assignment references to the variable. |

### Scope Hierarchy

| Function | Description |
|---|---|
| `Scope.upper(scope)` | Returns `Option.some(parentScope)` or `Option.none()` if root. |
| `Scope.childScopes(scope)` | Returns `ReadonlyArray<Scope>` for all direct nested child scopes. |
| `Scope.variables(scope)` | Returns `ReadonlyArray<Variable>` declared in this scope. |
| `Scope.throughReferences(scope)` | References in this scope resolved in an outer scope. |
| `Scope.isStrict(scope)` | Returns `true` if this scope operates in ECMAScript strict mode. |

---

## 3. `Comment.*` & `Token.*` Predicates

### `Comment.*` Predicates

```typescript
import * as Comment from 'effect-oxlint/Comment';

Comment.isLine(comment);            // Single-line comment (//)
Comment.isBlock(comment);           // Multi-line block comment (/* ... */)
Comment.isJSDoc(comment);           // JSDoc block comment (/** ... */)
Comment.isShebang(comment);         // Shebang (#!)
Comment.isDisableDirective(comment);// oxlint-disable, eslint-disable directive
Comment.isEnableDirective(comment); // oxlint-enable, eslint-enable directive
Comment.text(comment);              // Comment content string (without delimiters)
```

### `Token.*` Predicates

```typescript
import * as Token from 'effect-oxlint/Token';

Token.isKeyword(token, 'function'); // Checks if keyword (optional name check)
Token.isPunctuator(token, ';');     // Checks punctuation / operators
Token.isIdentifier(token);          // Identifier token
Token.isString(token);              // String literal token
Token.isNumeric(token);             // Numeric literal token
Token.isBoolean(token);             // Boolean literal token
Token.isNull(token);                // null token
Token.isTemplate(token);            // Template literal token
Token.value(token);                 // Raw token string value
Token.type(token);                  // Token type name
```
