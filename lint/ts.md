# TypeScript — Linter Config Reference

Created from: `erp-mcp-trainee`

## Tool: ESLint 10 + Prettier

```javascript
// eslint.config.mjs
export default [
  {
    rules: {
      // TypeScript
      "@typescript-eslint/no-explicit-any": "error",
      "@typescript-eslint/no-unused-vars": "error",
      "@typescript-eslint/explicit-function-return-type": "error",
      "@typescript-eslint/explicit-member-accessibility": "error",
      "@typescript-eslint/consistent-type-imports": "error",

      // Naming convention (Go-style)
      "@typescript-eslint/naming-convention": [
        "error",
        { selector: "method", format: ["PascalCase"] },         // public methods
        { selector: "property", format: ["camelCase"] },
        { selector: "interface", format: ["PascalCase"], prefix: ["I"] },
        { selector: "class", format: ["PascalCase"] },
        { selector: "variable", format: ["camelCase", "UPPER_CASE"] },
      ],

      // Function length
      "max-lines-per-function": ["error", 40],     // max 40 lines per function
      "max-params": ["error", 5],                   // max 5 parameters
      "complexity": ["error", 15],                  // max cyclomatic complexity
      "max-depth": ["error", 5],                    // max 5 nesting levels
      "max-nested-callbacks": ["error", 3],
      "max-statements": ["error", 30],              // max 30 statements

      // General
      "no-console": "warn",
      "prefer-const": "error",
      "eqeqeq": ["error", "always"],
      "no-var": "error",
      "prefer-template": "error",
      "object-shorthand": "error",
    },
  },
];
```

## Tool: Prettier

```json
{
  "semi": true,
  "singleQuote": true,
  "trailingComma": "all",
  "printWidth": 100,
  "tabWidth": 2
}
```

## Commands

```bash
npm run lint           # eslint src/
npm run format         # prettier --write src/
npm run typecheck      # tsc --noEmit
npm run test           # jest / vitest
```

## Key Rules

- No `any` — use `unknown`
- No `as` — use Zod `.parse()`
- Return types required on all functions
- Public methods: PascalCase
- Private methods: camelCase
- Interfaces: prefix `I`
- Max 40 lines per function
- Max 30 statements per function
- Max 5 parameters per function
- Max 15 cyclomatic complexity
- Max 5 nesting depth
- Max 3 nested callbacks
