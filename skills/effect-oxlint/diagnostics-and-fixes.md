# Diagnostics and Autofixes Reference

This document covers constructing oxlint diagnostics, configuring message IDs, and creating composable autofixes and suggestions using `Diagnostic.*`.

---

## 1. Creating Diagnostics

Diagnostics are reported via `ctx.report(...)` or `RuleContext.report(...)`.

### `Diagnostic.make(opts)`
Constructs a diagnostic with an explicit error message and node location:

```typescript
import { Diagnostic, RuleContext } from 'effect-oxlint';

const ctx = yield* RuleContext;

yield* ctx.report(
	Diagnostic.make({
		node, // Any AST node or token with range/loc
		message: 'Use Schema.decodeUnknown instead of JSON.parse'
	})
);
```

### `Diagnostic.fromId(opts)`
Constructs a diagnostic using a `messageId` defined in `meta.messages`:

```typescript
import { Diagnostic, Rule, RuleContext } from 'effect-oxlint';

export const rule = Rule.define({
	name: 'no-json-parse',
	meta: Rule.meta({
		type: 'problem',
		description: 'Bans JSON.parse',
		messages: {
			preferSchema: 'Avoid JSON.parse in favor of Schema.'
		}
	}),
	create: function* () {
		const ctx = yield* RuleContext;
		return {
			MemberExpression: (node) =>
				ctx.report(
					Diagnostic.fromId({
						node,
						messageId: 'preferSchema'
					})
				)
		};
	}
});
```

---

## 2. Composable Autofixes

Autofixes are created using fix helpers and attached to diagnostics with `Diagnostic.withFix`.

### Fix Helpers

Each fix helper takes a node or token (`Ranged`) and returns a `FixFn`:

| Function | Signature | Description |
|---|---|---|
| `Diagnostic.replaceText` | `(nodeOrToken: Ranged, text: string) => FixFn` | Replaces the text of the given node/token with new text. |
| `Diagnostic.insertBefore` | `(nodeOrToken: Ranged, text: string) => FixFn` | Inserts text immediately before the node/token. |
| `Diagnostic.insertAfter` | `(nodeOrToken: Ranged, text: string) => FixFn` | Inserts text immediately after the node/token. |
| `Diagnostic.removeFix` | `(nodeOrToken: Ranged) => FixFn` | Deletes the node/token from source code. |
| `Diagnostic.composeFixes` | `(...fixes: ReadonlyArray<FixFn>) => FixFn` | Combines multiple fixes into a single composite fix. |

### Attaching a Fix (`Diagnostic.withFix`)

`Diagnostic.withFix` has a dual API (data-first and data-last):

```typescript
import { Diagnostic, RuleContext } from 'effect-oxlint';

const ctx = yield* RuleContext;

// 1. Data-first
const diagWithFix = Diagnostic.withFix(
	Diagnostic.make({ node, message: 'Use Effect.fail' }),
	Diagnostic.replaceText(node, 'Effect.fail(new Error("failed"))')
);

// 2. Data-last with pipe
const diagPiped = Diagnostic.make({ node, message: 'Remove obsolete directive' }).pipe(
	Diagnostic.withFix(Diagnostic.removeFix(node))
);

yield* ctx.report(diagWithFix);
```

### Composing Multiple Fixes (`Diagnostic.composeFixes`)

When a single violation requires multiple edits (such as wrapping an expression and inserting an import, or updating both prefix and suffix):

```typescript
import { Diagnostic } from 'effect-oxlint';

const multiFix = Diagnostic.composeFixes(
	Diagnostic.insertBefore(node, 'Effect.succeed('),
	Diagnostic.insertAfter(node, ')')
);

const diagnostic = Diagnostic.withFix(
	Diagnostic.make({ node, message: 'Wrap expression in Effect.succeed' }),
	multiFix
);
```

---

## 3. Providing Suggestions (`Diagnostic.withSuggestions`)

Suggestions are optional, interactive fixes shown to the developer in IDEs when an autofix cannot be applied automatically with 100% confidence.

```typescript
import { Diagnostic, RuleContext } from 'effect-oxlint';

const ctx = yield* RuleContext;

const diagWithSuggestions = Diagnostic.withSuggestions(
	Diagnostic.make({ node, message: 'Avoid unsafe casting' }),
	[
		{
			desc: 'Cast using Schema.decodeUnknownSync',
			fix: Diagnostic.replaceText(node, 'Schema.decodeUnknownSync(MySchema)(val)')
		},
		{
			desc: 'Cast with Schema.assert',
			fix: Diagnostic.replaceText(node, 'Schema.assert(MySchema)(val)')
		}
	]
);

yield* ctx.report(diagWithSuggestions);
```
