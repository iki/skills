# Testing Rules Reference

This document covers testing custom `effect-oxlint` rules using `@effect/vitest` and the `effect-oxlint/testing` module.

---

## 1. Test Harness Overview

`effect-oxlint/testing` provides mock AST node builders, an in-memory rule execution harness (`runRule`, `runRuleMulti`), and assertion utilities that verify reported diagnostics.

### Basic Test Example

```typescript
import { describe, expect, test } from '@effect/vitest';
import * as Option from 'effect/Option';
import { Rule } from 'effect-oxlint';
import * as Testing from 'effect-oxlint/testing';

const noJsonParse = Rule.banMember('JSON', 'parse', {
	message: 'Use Schema for JSON'
});

describe('no-json-parse', () => {
	test('reports on JSON.parse', () => {
		const result = Testing.runRule(
			noJsonParse,
			'MemberExpression',
			Testing.memberExpr('JSON', 'parse')
		);

		Testing.expectDiagnostics(result, [
			{ message: 'Use Schema for JSON' }
		]);

		// Alternative assertion using Option-wrapped message strings:
		expect(Testing.messages(result)).toEqual([
			Option.some('Use Schema for JSON')
		]);
	});

	test('ignores other member expressions', () => {
		const result = Testing.runRule(
			noJsonParse,
			'MemberExpression',
			Testing.memberExpr('JSON', 'stringify')
		);

		Testing.expectNoDiagnostics(result);
	});
});
```

---

## 2. Running Rules in Tests

### `Testing.runRule(rule, nodeType, mockNode, options?)`
Executes a rule against a single mock AST node.

```typescript
const result = Testing.runRule(
	myRule,
	'CallExpression',
	Testing.callExpr(Testing.id('fetch'), [Testing.strLiteral('/api')]),
	{
		filename: '/app/src/index.ts', // Mock filename
		options: [{ allowFetch: false }] // Rule options array
	}
);
```

### `Testing.runRuleMulti(rule, mockNodes, options?)`
Executes a rule against an array of mock nodes sequentially. Useful for testing rules that maintain state across nodes using `Ref` or enter/exit pairs:

```typescript
test('tracks enter/exit depth', () => {
	const result = Testing.runRuleMulti(myScopedRule, [
		Testing.callOfMember('Effect', 'gen'), // Enter Effect.gen
		Testing.callOfMember('Effect', 'runSync'), // Inner call (should report)
		Testing.astNode('CallExpression:exit') // Exit Effect.gen
	]);

	Testing.expectDiagnostics(result, [
		{ message: 'Do not call runSync inside Effect.gen' }
	]);
});
```

---

## 3. Assertions

### `Testing.expectDiagnostics(result, expectedDiagnostics)`
Verifies that the rule reported the expected diagnostics. Matches partial diagnostic properties (e.g. `message`, `messageId`).

```typescript
Testing.expectDiagnostics(result, [
	{ message: 'Avoid unsafe calls' }
]);
```

### `Testing.expectNoDiagnostics(result)`
Asserts that zero diagnostics were reported.

```typescript
Testing.expectNoDiagnostics(result);
```

### `Testing.messages(result)`
Returns `ReadonlyArray<Option.Option<string>>` containing all reported diagnostic messages.

### `Testing.messageIds(result)`
Returns `ReadonlyArray<Option.Option<string>>` containing all reported diagnostic `messageId` identifiers.

---

## 4. Mock AST Node Builders

`effect-oxlint/testing` includes ergonomic shorthand builders so you do not need to construct raw ESTree AST objects by hand.

### Identifiers & Literals

| Builder | Description | Example |
|---|---|---|
| `Testing.id(name)` | `Identifier` node | `Testing.id('foo')` |
| `Testing.strLiteral(value)` | String literal | `Testing.strLiteral('hello')` |
| `Testing.numLiteral(value)` | Numeric literal | `Testing.numLiteral(42)` |
| `Testing.boolLiteral(value)` | Boolean literal | `Testing.boolLiteral(true)` |

### Calls & Member Expressions

| Builder | Description | Example |
|---|---|---|
| `Testing.memberExpr(obj, prop)` | Static member expression `obj.prop`. `obj` can be a string name or node. | `Testing.memberExpr('Math', 'random')` |
| `Testing.computedMemberExpr(obj, prop)` | Computed member `obj[prop]`. | `Testing.computedMemberExpr('obj', Testing.strLiteral('prop'))` |
| `Testing.chainedMemberExpr(parts)` | Deep member chain `a.b.c`. | `Testing.chainedMemberExpr(['a', 'b', 'c'])` |
| `Testing.callExpr(callee, args?)` | Call expression. | `Testing.callExpr(Testing.id('fetch'))` |
| `Testing.callOfMember(obj, prop, args?)` | Method call `obj.prop(...)`. | `Testing.callOfMember('console', 'log')` |
| `Testing.newExpr(callee, args?)` | `new` expression. String callee is auto-wrapped in `id()`. | `Testing.newExpr('Date')` |

### Statements & Flow Control

| Builder | Description |
|---|---|
| `Testing.exprStmt(expr)` | Wraps an expression into an `ExpressionStatement`. |
| `Testing.throwStmt(arg?)` | `ThrowStatement` node. |
| `Testing.tryStmt()` | Minimal `TryStatement` node. |
| `Testing.returnStmt(arg?)` | `ReturnStatement` node. |
| `Testing.ifStmt(test?, cons?, alt?)` | Minimal `IfStatement` node (all parameters optional). |
| `Testing.blockStmt([stmts])` | `BlockStatement` containing statements. |
| `Testing.switchStmt()` | Minimal `SwitchStatement`. |
| `Testing.forStmt()`, `Testing.forInStmt()`, `Testing.forOfStmt()` | Loop statements. |
| `Testing.whileStmt()`, `Testing.doWhileStmt()` | While statements. |
| `Testing.varDecl(kind, name, init?)` | Variable declaration (e.g. `'const'`, `'let'`). |

### Declarations & Program

| Builder | Description |
|---|---|
| `Testing.importDecl(source)` | `ImportDeclaration` for a given module specifier. |
| `Testing.importDeclWithSpecifiers(source, specifiers)` | Import with named or default specifiers. |
| `Testing.exportNamedDecl(declaration)` | `ExportNamedDeclaration` node. |
| `Testing.classDecl(name)` | `ClassDeclaration` node. |
| `Testing.interfaceDecl(name)` | TypeScript `TSInterfaceDeclaration` node. |
| `Testing.typeAliasDecl(name)` | TypeScript `TSTypeAliasDeclaration` node. |
| `Testing.program([body], [comments])` | Full `Program` root node with optional comments. |

### Comments & Tokens

```typescript
Testing.comment('Line', ' oxlint-disable no-console');
Testing.comment('Block', '* JSDoc info ');
Testing.token('Keyword', 'function');
Testing.token('Punctuator', ';');
```

---

## 5. Mocking Rule Context Directly

For testing custom functions that require `RuleContext`:

```typescript
import * as Effect from 'effect/Effect';
import { RuleContext } from 'effect-oxlint';
import * as Testing from 'effect-oxlint/testing';

const mockCtx = Testing.createMockContext({
	filename: '/project/app.ts',
	cwd: '/project'
});

const result = await Effect.runPromise(
	Effect.provide(
		Effect.gen(function* () {
			const filename = yield* RuleContext.filename;
			return filename;
		}),
		Testing.mockRuleContextLayer(mockCtx)
	)
);
```
