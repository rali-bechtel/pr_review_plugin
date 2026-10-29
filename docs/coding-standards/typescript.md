# TypeScript Coding Standards

| ID | Rule |
|----|------|
| TS-001 | `strict` mode on. No `any` — use `unknown` and narrow, or a proper type. |
| TS-002 | No non-null assertions (`!`) outside tests; handle the `undefined` case. |
| TS-003 | Prefer `type` for props/unions, `interface` for extendable object shapes. |
| TS-004 | Validate external data (API responses, config) with `zod` schemas, not manual casts. |
