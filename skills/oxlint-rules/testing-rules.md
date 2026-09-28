# Testing Rules Reference

Oxlint provides an ESLint-compatible `RuleTester` utility to unit test custom rules across Vitest, Jest, or Node test runner.

---

## 1. Setting Up `RuleTester`

```typescript
import { RuleTester } from 'oxlint'; // or import from test harness
import noDebugger from './no-debugger.ts';

const ruleTester = new RuleTester();

ruleTester.run('no-debugger', noDebugger, {
  valid: [
    'const a = 1;',
    'function test() { return 42; }'
  ],
  invalid: [
    {
      code: 'debugger;',
      errors: [
        {
          message: 'Unexpected debugger statement.'
        }
      ],
      output: '' // Expected code after autofix
    }
  ]
});
```

---

## 2. Test Case Structure

### Valid Test Cases

Valid cases ensure the rule does not report false positives. Cases can be specified as plain strings or configuration objects:

```typescript
valid: [
  // Simple code snippet
  'console.info("Allowed");',

  // Configured with custom rule options
  {
    code: 'console.log("Allowed by option");',
    options: [{ allow: ['log'] }]
  },

  // Configured with specific filename
  {
    code: 'import test from "test";',
    filename: 'src/index.test.ts'
  }
]
```

### Invalid Test Cases

Invalid cases verify that violations are reported accurately and autofixes produce the exact expected output.

```typescript
invalid: [
  {
    code: 'var x = 1;',
    output: 'const x = 1;', // Verifies output after autofix
    errors: [
      {
        message: 'Use const instead of var',
        line: 1,
        column: 1
      }
    ]
  },
  {
    code: 'forbidden();',
    options: [{ banned: ['forbidden'] }],
    errors: [
      {
        messageId: 'bannedCall',
        data: { name: 'forbidden' }
      }
    ]
  }
]
```

---

## 3. Testing Interactive Suggestions

Verify that suggestions are provided with matching descriptions and outputs:

```typescript
invalid: [
  {
    code: 'const a = 1; a = 2;',
    errors: [
      {
        message: 'Reassignment to const variable',
        suggestions: [
          {
            desc: "Change declaration to 'let'",
            output: 'let a = 1; a = 2;'
          }
        ]
      }
    ]
  }
]
```
