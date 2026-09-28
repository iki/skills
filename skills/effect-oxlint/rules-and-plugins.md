# Rules and Plugins Reference

This document covers defining rules, using shorthand rule factories, accessing `RuleContext`, handling the error channel contract, and assembling rules into oxlint plugins.

---

## 1. Rule Definition (`Rule.define`)

`Rule.define` is the primary entry point for creating custom oxlint rules with Effect. It produces a standard oxlint `CreateRule` function while allowing authors to write effectful generator functions for rule setup.

```typescript
import * as Effect from 'effect/Effect';
import * as Option from 'effect/Option';
import * as Schema from 'effect/Schema';
import { AST, Diagnostic, Rule, RuleContext } from 'effect-oxlint';

// Optional options schema
const OptionsSchema = Schema.Struct({
	allowInTests: Schema.optional(Schema.Boolean).pipe(Schema.withDefault(false))
});

export const myCustomRule = Rule.define({
	name: 'my-custom-rule',
	meta: Rule.meta({
		type: 'problem', // 'problem' | 'suggestion' | 'layout'
		description: 'Disallow direct calls to unsafeMethod',
		fixable: 'code', // 'code' | 'whitespace' | undefined
		hasSuggestions: true,
		messages: {
			avoidUnsafe: 'Avoid using unsafeMethod; use safeMethod instead.'
		}
	}),
	options: OptionsSchema, // Decoded automatically via Schema.decodeSync
	create: function* (options) {
		const ctx = yield* RuleContext;

		return {
			// Visitor keys provide narrowed ESTree nodes automatically
			CallExpression: (node) =>
				Option.match(AST.matchCallOf(node, 'Service', 'unsafeMethod'), {
					onNone: () => Effect.void,
					onSome: (callNode) =>
						ctx.report(
							Diagnostic.fromId({
								node: callNode,
								messageId: 'avoidUnsafe'
							})
						)
				})
		};
	}
});
```

### `RuleConfig<Options>` Interface

| Field | Type | Description |
|---|---|---|
| `name` | `string` | Rule name used for tracing and diagnostics. |
| `meta` | `RuleMeta` | Metadata including description, severity type, messages, and fixability. Built with `Rule.meta(...)`. |
| `options` | `Schema.Decoder<Options>` (optional) | Schema to decode `oxlintContext.options[0]`. Defaults to `undefined`. |
| `create` | `(options: Options) => Effect.gen.Return<TypedEffectVisitor, never, RuleContext>` | Generator executed once per linted file. May allocate `Ref` instances and query `RuleContext`. |

---

## 2. Runtime Boundary and Error Channel Contract

Oxlint is a high-performance synchronous Rust-based linter that executes JS plugin rules synchronously via an FFI bridge.

### The `never` Error Channel

Both the `create` generator and all visitor handlers have a fixed error channel of **`never`**:

```typescript
type EffectHandler<N = ESTree.Node> = (node: N) => Effect.Effect<void, never, RuleContext>;
```

- A rule **cannot fail** via `Effect.fail`. If an uncaught defect or failure occurs, oxlint's process will abort or crash for the file.
- If your rule needs to perform an operation that can fail (e.g. schema decoding, fallible string parsing, or custom Effect computations), catch the error inside the handler and decide how to report it:

```typescript
import * as Effect from 'effect/Effect';
import { Diagnostic, RuleContext, Visitor } from 'effect-oxlint';

Visitor.on('CallExpression', (node) =>
	fallibleAnalysis(node).pipe(
		Effect.catch((error) =>
			RuleContext.report(
				Diagnostic.make({
					node,
					message: `Analysis error: ${error.message}`
				})
			)
		)
	)
);
```

### State Management with `Ref`

Because `create` is executed once per file and visitor handlers are called per node, you can maintain state across node traversals using `Ref` without mutable global variables:

```typescript
import * as Ref from 'effect/Ref';
import { Rule, RuleContext } from 'effect-oxlint';

Rule.define({
	name: 'stateful-rule',
	meta: Rule.meta({ type: 'suggestion', description: 'Tracks traversal state' }),
	create: function* () {
		const ctx = yield* RuleContext;
		const callCount = yield* Ref.make(0);

		return {
			CallExpression: (node) =>
				Ref.update(callCount, (n) => n + 1),
			'Program:exit': (node) =>
				Effect.gen(function* () {
					const count = yield* Ref.get(callCount);
					if (count > 50) {
						yield* ctx.report(
							Diagnostic.make({ node, message: `Too many calls: ${count}` })
						);
					}
				})
		};
	}
});
```

---

## 3. `RuleContext` Service

`RuleContext` is an Effect service (`Context.Service`) containing the active oxlint context for the file being linted.

### Resolving `RuleContext`

Inside `create` or an effectful visitor handler:
```typescript
const ctx = yield* RuleContext;
```

### `RuleContextService` Members

| Member | Type | Description |
|---|---|---|
| `ctx.report(diagnostic)` | `(diagnostic: Diagnostic) => Effect.Effect<void>` | Report a lint diagnostic for the current file. |
| `ctx.filename` | `string` | Absolute path of the file being linted. |
| `ctx.cwd` | `string` | Current working directory. |
| `ctx.id` | `string` | Rule ID in `plugin/rule` form. |
| `ctx.options` | `Readonly<Options>` | Raw options array from oxlint configuration. |
| `ctx.sourceCode` | `SourceCode` | The oxlint `SourceCode` object (tokens, comments, AST, scope). |
| `ctx.languageOptions` | `Readonly<LanguageOptions>` | Parser & language options for the current file. |
| `ctx.settings` | `Readonly<Settings>` | Shared settings from oxlint config. |

### Top-Level Shorthand Accessors

You can directly `yield*` these convenience effects without resolving `RuleContext` first:

```typescript
import { RuleContext } from 'effect-oxlint';

function* handler(node) {
	const currentFile = yield* RuleContext.filename;
	const workingDir = yield* RuleContext.cwd;
	const ruleId = yield* RuleContext.id;
	const fullText = yield* RuleContext.text;
	const rootAst = yield* RuleContext.ast;
	const source = yield* RuleContext.sourceCode;

	yield* RuleContext.report(Diagnostic.make({ node, message: 'Violation' }));
}
```

---

## 4. Shorthand Ban Factories

`effect-oxlint` includes optimized built-in rule factories for the most common linting patterns.

### `Rule.banMember(obj, prop, opts)`
Bans property access `obj.prop` on member expressions.

```typescript
// Ban Math.random
const noMathRandom = Rule.banMember('Math', 'random', {
	message: 'Use Effect Random service instead'
});

// Ban multiple properties on an object
const noDateNow = Rule.banMember('Date', ['now', 'parse'], {
	message: 'Use Clock service instead'
});
```

### `Rule.banImport(source, opts)`
Bans module imports matching a exact string or predicate function.

```typescript
// Exact string match
const noNodeFs = Rule.banImport('node:fs', {
	message: 'Use the Effect FileSystem service instead'
});

// Predicate match
const noInternalDeps = Rule.banImport((src) => src.includes('/internal/'), {
	message: 'Do not import from internal paths'
});
```

### `Rule.banCallOf(name, opts)`
Bans direct function calls of identifiers (e.g. `fetch()`, `useState()`).

```typescript
// Single callee
const noFetch = Rule.banCallOf('fetch', {
	message: 'Use HttpClient instead'
});

// Multiple callees
const noReactHooks = Rule.banCallOf(['useState', 'useEffect'], {
	message: 'Use Effect state management'
});
```

### `Rule.banCallOfMember(obj, prop, opts)`
Bans method calls `obj.prop(...)` (e.g. `Effect.runSync(...)`, `console.log(...)`). Unlike `banMember`, this only matches when the member expression is executed as a call.

```typescript
const noRunSync = Rule.banCallOfMember('Effect', ['runSync', 'runPromise'], {
	message: 'Keep effects composable — run only at application boundaries'
});
```

### `Rule.banNewExpr(name, opts)`
Bans `new` constructor expressions (e.g. `new Date()`, `new Error()`).

```typescript
const noNewDate = Rule.banNewExpr('Date', {
	message: 'Use Clock service'
});

const noNativeErrors = Rule.banNewExpr(['Error', 'TypeError', 'RangeError'], {
	message: 'Use tagged Schema / Data errors'
});
```

### `Rule.banStatement(nodeType, opts)`
Bans specific AST statement types.

```typescript
const noThrow = Rule.banStatement('ThrowStatement', {
	message: 'Use Effect.fail instead of throw'
});
```

### `Rule.banMultiple(spec, opts)`
Combines multiple ban rules under a single rule definition with merged visitors.

```typescript
const noImperativeLoops = Rule.banMultiple(
	{
		statements: [
			'ForStatement',
			'ForInStatement',
			'ForOfStatement',
			'WhileStatement',
			'DoWhileStatement'
		]
	},
	{ message: 'Use Arr.map / Effect.forEach instead' }
);

const useFileSystemService = Rule.banMultiple(
	{
		imports: ['fs', 'node:fs', 'node:fs/promises'],
		members: [['fs', ['readFileSync', 'writeFileSync']]],
		memberCalls: [['fs', ['readFile', 'writeFile']]]
	},
	{ message: 'Use Effect FileSystem service' }
);
```

#### `BanMultipleSpec` Options
- `calls`: `string | ReadonlyArray<string>`
- `newExprs`: `string | ReadonlyArray<string>`
- `members`: `ReadonlyArray<readonly [obj: string, prop: string | ReadonlyArray<string>]>`
- `memberCalls`: `ReadonlyArray<readonly [obj: string, prop: string | ReadonlyArray<string>]>`
- `imports`: `ReadonlyArray<string | ((source: string) => boolean)>`
- `statements`: `ReadonlyArray<string>`

---

## 5. Plugin Assembly (`Plugin.define`)

Plugins bundle multiple rules together and generate shareable configs for `oxlint.config.ts`.

```typescript
import { Plugin } from 'effect-oxlint';
import { noJsonParse } from './rules/no-json-parse.ts';
import { noMathRandom } from './rules/no-math-random.ts';
import { noThrow } from './rules/no-throw.ts';

export default Plugin.define({
	name: 'my-effect-rules',
	specifier: 'oxlint-plugin-my-effect-rules',
	rules: {
		'no-json-parse': noJsonParse,
		'no-math-random': noMathRandom,
		'no-throw': noThrow
	},
	recommended: {
		severity: 'error',
		rules: ['no-json-parse', 'no-throw'] // Type-checked against keys of `rules`
	}
});
```

### Generated Configurations

`Plugin.define` outputs standard oxlint config objects:
- `plugin.configs.recommended`: Contains specified recommended rules (or all rules if `recommended.rules` is omitted) at the given severity (`'warn'` or `'error'`).
- `plugin.configs.all`: Contains every rule in the plugin at `'error'` severity.

### Consuming in `oxlint.config.ts`

```typescript
import { defineConfig } from 'oxlint';
import myPlugin from 'oxlint-plugin-my-effect-rules';

export default defineConfig({
	extends: [myPlugin.configs.recommended]
});
```

### Merging Plugins (`Plugin.merge`)

```typescript
import { Plugin } from 'effect-oxlint';

const combinedPlugin = Plugin.merge(pluginA, pluginB);
```
