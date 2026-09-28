# AST Pattern Matching and Visitors Reference

This document covers AST traversal combinators (`Visitor.*`) and `Option`-safe AST pattern matchers (`AST.*`) used to inspect ESTree nodes in custom oxlint rules.

---

## 1. Visitor Combinators (`Visitor.*`)

Oxlint visitors are maps from ESTree node names to handler functions. The `Visitor` module provides combinators to build, merge, filter, and track AST node visitors cleanly without mutable global variables.

### Handler Type: `EffectHandler`

Every handler receives an ESTree node (narrowed automatically if using typed visitor keys) and returns an Effect:

```typescript
type EffectHandler<N = ESTree.Node> = (node: N) => Effect.Effect<void, never, RuleContext>;
```

### `Visitor.on(nodeType, handler)`
Creates a single-entry visitor for the enter phase of an AST node:

```typescript
import { Diagnostic, RuleContext, Visitor } from 'effect-oxlint';

const visitor = Visitor.on('ThrowStatement', function* (node) {
	const ctx = yield* RuleContext;
	yield* ctx.report(Diagnostic.make({ node, message: 'Do not throw directly' }));
});
```

### `Visitor.onExit(nodeType, handler)`
Creates a single-entry visitor for the exit phase of an AST node (`NodeType:exit`):

```typescript
const exitVisitor = Visitor.onExit('FunctionDeclaration', (node) =>
	Effect.sync(() => {
		// Executed when leaving FunctionDeclaration
	})
);
```

### `Visitor.merge(...visitors)`
Merges multiple visitors into a single visitor map. If multiple visitors handle the same node type, their handlers execute sequentially in order from left to right:

```typescript
import { Visitor } from 'effect-oxlint';

const combinedVisitor = Visitor.merge(
	importVisitor,
	memberVisitor,
	statementVisitor
);
```

### `Visitor.tracked(nodeType, predicate, ref)`
Replaces mutable depth counters (`let depth = 0;`) with a `Ref<number>` that increments on enter and decrements on exit when the predicate matches:

```typescript
import * as Ref from 'effect/Ref';
import { AST, Rule, RuleContext, Visitor } from 'effect-oxlint';

export const noUnsafeInsideGen = Rule.define({
	name: 'no-unsafe-inside-gen',
	meta: Rule.meta({ type: 'problem', description: 'No unsafe calls inside Effect.gen' }),
	create: function* () {
		const ctx = yield* RuleContext;
		const genDepth = yield* Ref.make(0);

		// Increments when entering Effect.gen, decrements when exiting
		const tracker = Visitor.tracked(
			'CallExpression',
			(node) => AST.isCallOf(node, 'Effect', 'gen'),
			genDepth
		);

		const checker = Visitor.on('CallExpression', function* (node) {
			const depth = yield* Ref.get(genDepth);
			if (depth > 0 && AST.isCallOf(node, 'Effect', 'runSync')) {
				yield* ctx.report(
					Diagnostic.make({ node, message: 'Do not call runSync inside Effect.gen' })
				);
			}
		});

		return Visitor.merge(tracker, checker);
	}
});
```

### `Visitor.accumulate(nodeType, extract, analyze)`
Accumulates values into a `Ref<ReadonlyArray<A>>` during traversal and invokes an analysis generator once at `Program:exit`:

```typescript
import * as Option from 'effect/Option';
import { AST, Diagnostic, RuleContext, Visitor } from 'effect-oxlint';

const exportCollector = yield* Visitor.accumulate(
	'ExportNamedDeclaration',
	(node) => AST.narrow(node, 'ExportNamedDeclaration'),
	function* (exports) {
		const ctx = yield* RuleContext;
		if (exports.length === 0) {
			const ast = yield* RuleContext.ast;
			yield* ctx.report(
				Diagnostic.make({ node: ast, message: 'Module should export at least one symbol' })
			);
		}
	}
);
```

### `Visitor.filter(predicate, visitor)`
Conditionally activates a visitor based on a predicate evaluating the filename (`(filename: string) => boolean`):

```typescript
import { Visitor } from 'effect-oxlint';

// Ignore test files
const visitor = yield* Visitor.filter(
	(filename) => !filename.endsWith('.test.ts') && !filename.includes('__tests__'),
	productionVisitor
);
```

---

## 2. AST Pattern Matching (`AST.*`)

All `AST.*` matchers return `Option.Option<T>` for safe composition with `Option.map`, `Option.flatMap`, and `pipe`. All public combinators support a **dual API** (can be called as `AST.fn(node, arg)` or `AST.fn(arg)(node)`).

### `AST.narrow(node, type)`
Safely casts a general `ESTree.Node` to a specific AST node type, returning `Option.some(narrowedNode)` if the type matches:

```typescript
import { pipe } from 'effect';
import * as Option from 'effect/Option';
import { AST, type ESTree } from 'effect-oxlint';

declare const node: ESTree.Node;

const callExpr: Option.Option<ESTree.CallExpression> = AST.narrow(node, 'CallExpression');

// Often used in pipelines:
pipe(
	AST.narrow(node, 'CallExpression'),
	Option.flatMap(AST.matchCallOf('Effect', 'gen'))
);
```

### `AST.matchMember(node, obj, prop)` & `AST.isMember`
Matches a static member expression `obj.prop` where `obj` is an identifier and `prop` matches a string or list of strings.

- `matchMember` returns `Option.Option<ESTree.StaticMemberExpression>`.
- `isMember` returns a boolean predicate.

```typescript
import { AST, type ESTree } from 'effect-oxlint';

declare const member: ESTree.MemberExpression;

// Data-first
const match = AST.matchMember(member, 'JSON', ['parse', 'stringify']);

// Data-last (piping)
const matchPipe = pipe(member, AST.matchMember('Math', 'random'));

// Boolean check
if (AST.isMember(member, 'console', ['log', 'error'])) {
	// ...
}
```

### `AST.matchCallOf(node, obj, prop)` & `AST.isCallOf`
Matches a method call expression `obj.prop(...)` (a `CallExpression` whose callee is a static member expression `obj.prop`).

- `matchCallOf` returns `Option.Option<ESTree.CallExpression>`.
- `isCallOf` returns a boolean predicate.

```typescript
import { AST, type ESTree } from 'effect-oxlint';

declare const call: ESTree.CallExpression;

// Matches Effect.runSync(...) or Effect.runPromise(...)
const match = AST.matchCallOf(call, 'Effect', ['runSync', 'runPromise']);

if (AST.isCallOf(call, 'Effect', 'gen')) {
	// matches Effect.gen(...)
}
```

### `AST.matchImport(node, source)` & `AST.isImport`
Matches an `ImportDeclaration` whose module specifier matches a string or predicate.

- `matchImport` returns `Option.Option<ESTree.ImportDeclaration>`.
- `isImport` returns a boolean predicate.

```typescript
import { AST, type ESTree } from 'effect-oxlint';

declare const imp: ESTree.ImportDeclaration;

// Match exact module
const fsImport = AST.matchImport(imp, 'node:fs');

// Match predicate
const internalImport = AST.matchImport(imp, (src) => src.startsWith('@internal/'));
```

### Callee and Segment Extractors

#### `AST.calleeName(callNode)`
Returns `Option.some(string)` if the callee of a `CallExpression` is an Identifier (e.g. `'fetch'` for `fetch()`).

```typescript
pipe(
	AST.narrow(node, 'CallExpression'),
	Option.flatMap(AST.calleeName),
	Option.filter((name) => name === 'fetch')
);
```

#### `AST.calleeIdentifier(newNode)`
Returns `Option.some(string)` if the callee of a `NewExpression` is an Identifier (e.g. `'Date'` for `new Date()`).

#### `AST.memberNames(memberNode)`
Returns `Option.some([objectName, propertyName])` for a static member expression `a.b`.

#### `AST.importSource(importNode)`
Returns the source module specifier string (e.g. `'react'` for `import ... from 'react'`).

### Member Paths & Hierarchical Analysis

#### `AST.memberPath(node)`
Deconstructs chained member expressions into an array of identifier names.
For example, `a.b.c` returns `Option.some(['a', 'b', 'c'])`.

```typescript
import { AST, type ESTree } from 'effect-oxlint';

declare const node: ESTree.MemberExpression;
const path = AST.memberPath(node); // Option.Option<ReadonlyArray<string>>
```

#### `AST.findAncestor(node, predicate)` & `AST.hasAncestor`
Searches up the AST node's parent chain (via `node.parent`) for the nearest ancestor satisfying the predicate:

```typescript
// Check if the node is nested inside a TryStatement
const isInsideTry = AST.hasAncestor(node, (ancestor) => ancestor.type === 'TryStatement');

// Find the enclosing FunctionDeclaration
const enclosingFn = AST.findAncestor(node, (ancestor) => ancestor.type === 'FunctionDeclaration');
```

### Object Literal Helpers

#### `AST.objectKeys(node)`
Extracts static property names from an `ESTree.ObjectExpression` as `ReadonlyArray<string>`.

#### `AST.objectHasKey(node, key)`
Checks if an object expression contains a specific property key.

#### `AST.objectGetValue(node, key)`
Returns `Option.some(ESTree.Expression)` containing the property value for the given key in an object expression.
