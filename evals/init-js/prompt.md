Set up the shipit contract for this repository.

It is a TypeScript project. `package.json` declares:

```json
{
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    "test": "vitest run",
    "test:watch": "vitest",
    "lint": "eslint .",
    "typecheck": "tsc --noEmit"
  },
  "devDependencies": {
    "vitest": "^2.1.0",
    "eslint": "^9.13.0",
    "typescript": "^5.6.0",
    "vite": "^5.4.0"
  }
}
```

There is a `pnpm-lock.yaml`. Tests live beside their sources as `*.test.ts`. There is
no `CLAUDE.md` and no CI configuration in the repository.

Produce the contract.
