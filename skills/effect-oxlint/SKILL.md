---
name: effect-oxlint
description: >-
  Author custom oxlint lint rules and plugins using effect-oxlint and Effect v4 idioms.
  Use when writing, refactoring, testing, or configuring custom oxlint rules, AST visitors,
  pattern matchers, diagnostics, autofixes, or plugin definitions.
---

# effect-oxlint Skill

`effect-oxlint` provides an Effect-first API for authoring custom [oxlint](https://oxc.rs/docs/guide/usage/linter) rules and plugins. It wraps `@oxlint/plugins` to provide typed errors, composable visitors, `Option`-safe AST matching, and `Ref`-based state without mutable variables.

---

## Core Constraints & Execution Model

1. **Synchronous Boundary (`Effect.runSync`)**:
   Oxlint runs synchronously in Rust. The `create` generator runs once per file via `Effect.runSync`, and visitor handlers execute per matching node via `Effect.runSync`.
2. **Fixed `never` Error Channel**:
   Visitor handlers and `create` cannot fail via `Effect.fail`. Any fallible sub-effects must be caught inside the handler (`Effect.catch`) and converted to `Effect.void` or reported as a diagnostic via `RuleContext.report`.
3. **State Management via `Ref`**:
   Never use mutable global `let` variables. Create `Ref` instances inside the `create` generator (`yield* Ref.make(...)`). Handlers access and mutate these refs via closure.
4. **`Option` for Missing Data**:
   All AST matchers return `Option.Option<T>`. Never use `null`, `undefined`, or non-null assertions (`!`) on AST queries.

---

## ⚡ Quick Reference

### 1. Defining a Custom Rule (`Rule.define`)

```typescript
import * as Effect from 'effect/Effect';
import * as Option from 'effect/Option';
import { AST, Diagnostic, Rule, RuleContext } from 'effect-oxlint';

export const noJsonParse = Rule.define({
	name: 'no-json-parse',
	meta: Rule.meta({
		type: 'suggestion',
		description: 'Use Schema for JSON decoding instead of JSON.parse'
	}),
	create: function* () {
		const ctx = yield* RuleContext;

		return {
			// Visitor keys provide narrowed ESTree nodes automatically
			MemberExpression: (node) =>
				Option.match(
					AST.matchMember(node, 'JSON', ['parse', 'stringify']),
					{
						onNone: () => Effect.void,
						onSome: (matched) =>
							ctx.report(
								Diagnostic.make({
									node: matched,
									message: 'Use Schema for JSON decoding'
								})
							)
					}
				)
		};
	}
});
```

### 2. Shorthand Ban Factories

For common banning patterns, use one-line factories instead of writing full visitors:

```typescript
import { Rule } from 'effect-oxlint';

// Ban property access (e.g. Math.random)
const noMathRandom = Rule.banMember('Math', 'random', {
	message: 'Use Effect Random service instead'
});

// Ban imports by specifier or predicate
const noNodeFs = Rule.banImport('node:fs', {
	message: 'Use Effect FileSystem service instead'
});

// Ban bare function calls (e.g. fetch(), useState())
const noFetch = Rule.banCallOf('fetch', {
	message: 'Use HttpClient instead'
});

// Ban method calls (e.g. Effect.runSync(...), console.log(...))
const noRunSync = Rule.banCallOfMember('Effect', ['runSync', 'runPromise'], {
	message: 'Run effects only at application entry points'
});

// Ban new expressions (e.g. new Date(), new Error())
const noNewDate = Rule.banNewExpr('Date', {
	message: 'Use Clock service instead'
});

// Ban statement types
const noThrow = Rule.banStatement('ThrowStatement', {
	message: 'Use Effect.fail instead of throw'
});

// Ban multiple patterns under a single rule
const noLoops = Rule.banMultiple(
	{
		statements: ['ForStatement', 'ForInStatement', 'ForOfStatement', 'WhileStatement'],
		newExprs: 'Date',
		memberCalls: [['console', ['log', 'warn', 'error']]]
	},
	{ message: 'Avoid imperative patterns' }
);
```

### 3. Composable Visitors & Depth Tracking

```typescript
import * as Ref from 'effect/Ref';
import { AST, Diagnostic, Rule, RuleContext, Visitor } from 'effect-oxlint';

export const noSyncInGen = Rule.define({
	name: 'no-sync-in-gen',
	meta: Rule.meta({ type: 'problem', description: 'Disallow runSync in Effect.gen' }),
	create: function* () {
		const ctx = yield* RuleContext;
		const genDepth = yield* Ref.make(0);

		// Increment depth when entering Effect.gen, decrement on exit
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

### 4. AST Matching with Dual API

```typescript
import { pipe } from 'effect';
import * as Option from 'effect/Option';
import { AST, type ESTree } from 'effect-oxlint';

declare const node: ESTree.Node;

// Narrow node type and match call pattern
pipe(
	AST.narrow(node, 'CallExpression'),
	Option.flatMap(AST.matchCallOf('Effect', 'gen'))
);

// Extract member path: a.b.c -> Some(['a', 'b', 'c'])
declare const member: ESTree.MemberExpression;
const path = AST.memberPath(member);
```

### 5. Diagnostics with Autofixes

```typescript
import { Diagnostic, RuleContext } from 'effect-oxlint';

const ctx = yield* RuleContext;

// Diagnostic with single autofix
const diag = Diagnostic.withFix(
	Diagnostic.make({ node, message: 'Use Effect.fail' }),
	Diagnostic.replaceText(node, 'Effect.fail(err)')
);

// Diagnostic with composed fixes
const multiFix = Diagnostic.composeFixes(
	Diagnostic.insertBefore(node, 'Effect.succeed('),
	Diagnostic.insertAfter(node, ')')
);
const diagMulti = Diagnostic.withFix(Diagnostic.make({ node, message: 'Wrap expression' }), multiFix);

yield* ctx.report(diagMulti);
```

### 6. Testing Rules (`effect-oxlint/testing`)

```typescript
import { describe, expect, test } from '@effect/vitest';
import * as Option from 'effect/Option';
import * as Testing from 'effect-oxlint/testing';
import { noJsonParse } from './noJsonParse.ts';

describe('no-json-parse', () => {
	test('reports on JSON.parse', () => {
		const result = Testing.runRule(
			noJsonParse,
			'MemberExpression',
			Testing.memberExpr('JSON', 'parse')
		);

		Testing.expectDiagnostics(result, [
			{ message: 'Use Schema for JSON decoding' }
		]);
		expect(Testing.messages(result)).toEqual([
			Option.some('Use Schema for JSON decoding')
		]);
	});

	test('ignores other members', () => {
		const result = Testing.runRule(
			noJsonParse,
			'MemberExpression',
			Testing.memberExpr('console', 'log')
		);
		Testing.expectNoDiagnostics(result);
	});
});
```

### 7. Assembling into a Plugin (`Plugin.define`)

```typescript
import { Plugin } from 'effect-oxlint';
import { noJsonParse } from './rules/noJsonParse.ts';
import { noMathRandom } from './rules/noMathRandom.ts';

export default Plugin.define({
	name: 'my-effect-rules',
	specifier: 'oxlint-plugin-my-effect-rules',
	rules: {
		'no-json-parse': noJsonParse,
		'no-math-random': noMathRandom
	},
	recommended: {
		severity: 'error',
		rules: ['no-json-parse'] // Type-checked against rules keys
	}
});
```

In `oxlint.config.ts`:
```typescript
import { defineConfig } from 'oxlint';
import plugin from 'oxlint-plugin-my-effect-rules';

export default defineConfig({
	extends: [plugin.configs.recommended]
});
```

---

## Detailed References

For comprehensive signatures, advanced combinators, and usage details, refer to:

- [Rules and Plugins Reference](./rules-and-plugins.md): `Rule.define`, options decoding with Schema, ban factories, `RuleContext` service and shorthands, `Plugin.define`, and oxlint config integration.
- [AST Pattern Matching and Visitors Reference](./ast-and-visitors.md): `Visitor.on`, `Visitor.tracked`, `Visitor.accumulate`, `Visitor.filter`, `AST.narrow`, `AST.matchMember`, `AST.matchCallOf`, `AST.memberPath`, and ancestor traversal.
- [Diagnostics and Autofixes Reference](./diagnostics-and-fixes.md): Diagnostic construction, message IDs, autofixes (`replaceText`, `insertBefore`, `insertAfter`, `removeFix`, `composeFixes`), and suggestions.
- [SourceCode and Scope Reference](./source-code-and-scope.md): Querying source text, tokens, comments, JSDoc directives, variable references, and scope trees.
- [Testing Rules Reference](./testing-rules.md): Rule testing harness (`runRule`, `runRuleMulti`), assertion helpers (`expectDiagnostics`, `expectNoDiagnostics`), and mock AST node builders.