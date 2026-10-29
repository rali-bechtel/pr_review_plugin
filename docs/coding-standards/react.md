# React Coding Standards

| ID | Rule |
|----|------|
| RX-001 | Function components only, one component per file, filename matches component name. |
| RX-002 | Props typed via a named `XProps` type, destructured in the signature. |
| RX-003 | No business logic in JSX — extract to a hook or helper function. |
| RX-004 | Keep components under ~150 lines; split when they take on multiple responsibilities. |
| RX-005 | Hook files/functions start with `use`. |
| RX-006 | Exhaustive `useEffect`/`useMemo`/`useCallback` dependency arrays — don't suppress the lint rule. |
| RX-007 | No side effects (fetch, subscriptions) directly in render; wrap in `useEffect`. |
| RX-008 | Local UI state via `useState`; shared/cross-component state via context or the project's existing store — don't reinvent one. |
| RX-009 | Forms via `react-hook-form` + `zod` resolver (match existing patterns in `src/schemas`). |
| RX-010 | Don't call `fetch`/API clients directly from components — go through `src/services`. |
| RX-011 | Every `navigate()` target and `<Link to>`/`href` must correspond to a route actually registered in the router. |
| RX-012 | Never render untrusted/config-sourced URLs via `<a href>` without validating with `z.string().url()` (or equivalent) first. |
| RX-013 | No secrets, tokens, or credentials in client code or logs. |
| RX-014 | No new dependency unless it's actually imported and used in the same PR. |
| RX-015 | No dead code: don't add schemas/types/utilities that nothing imports. |
| RX-016 | ESLint + Prettier config in the repo is authoritative; don't hand-roll different formatting. |
| RX-017 | Named exports for everything except page-level components (default export OK for pages/routes). |
