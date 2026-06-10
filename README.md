# GoFr Golang PR Review Skill

Opinionated, in-depth PR review for **Go (GoFr framework)** backends and **React/TypeScript/JavaScript** frontends. Uses **parallel review agents** to analyze every file in the PR — not just skim the diff.

## Why This Exists

Most AI review tools give 3-5 surface-level comments on a 9K LOC PR. This skill dispatches **7 parallel agents** — each specialized in one review aspect — to read every file fully and check 140+ rules. A 9K LOC PR gets 50-100 comments, not 4.

## How It Works

```
STEP 1: Fetch PR data (metadata + full diff)
STEP 2: Validate PR title, description, linked issues, size
STEP 3: Categorize files (Go, tests, migrations, frontend, config)
STEP 4: Load/cache repo context (architecture, patterns, conventions)
STEP 5: Dispatch 7 parallel agents:
         Agent 1: Architecture & structure
         Agent 2: Error handling & safety
         Agent 3: Code quality & comment bloat
         Agent 4: Database & queries
         Agent 5: Testing completeness
         Agent 6: Security & GoFr patterns
         Agent 7: Frontend deep review
STEP 6: Collect, deduplicate, classify issues
STEP 7: Present review locally (DO NOT auto-post)
STEP 8: Post to GitHub only when user says "push"
STEP 9: Re-review mode (verify fixes, audit new commits)
```

## Installation

```bash
/plugin install gofr-golang-pr-review-skill
```

## Usage

```
/review <owner/repo> <pr-number>
```

Examples:
```
/review acme-corp/order-service 142
/review acme-corp/payment-api 87
/review acme-corp/dashboard-ui 53
```

After review is presented locally:
- Say **"push"** or **"post"** to post all comments to GitHub
- Say **"skip"** to discard

For re-review after author fixes:
```
/review acme-corp/order-service 142
```
(Same command — detects previous review and runs re-review mode)

## What It Checks

### Backend (Go / GoFr) — 21 Rule Groups

| Rule | What It Catches |
|------|----------------|
| Architecture | Handler→Service→Store violations, business logic in wrong layer |
| Comment Bloat | 7-line comments on 1-line functions, verbose file headers |
| Export Hygiene | Exported functions that should be unexported |
| Error Handling | Panics, nil derefs, swallowed errors, wrong error types |
| Resource Safety | Goroutine leaks, missing defer close, context.Background() |
| No JOINs | SQL JOINs and foreign keys in queries/migrations |
| Redis Justify | New Redis usage without justification over in-memory cache |
| File Org | Interfaces not in interface.go, scattered constants |
| Imports | Unsorted, unused, dot imports |
| Testing | Non-table-driven, missing error paths, <90% coverage |
| HTTP Calls | Raw net/http instead of GoFr service client |
| Migrations | Wrong naming, syntax errors, JOINs in DDL |
| GoFr Patterns | json.NewDecoder vs c.Bind, os.Getenv vs c.Config.Get |
| Dead Code | Commented-out code, unreachable code, debug logging |
| Security | OWASP: injection, SSRF, path traversal, secrets, weak crypto |
| N+1 Queries | DB queries inside loops |
| Pagination | List endpoints without LIMIT/pagination |
| Concurrency | Race conditions, mutex ordering, goroutine safety |
| Breaking API | Changed response fields, removed endpoints, status codes |
| Dependencies | Vulnerable, deprecated, or unnecessary packages |
| Timeouts | Missing HTTP client timeouts, connection pool defaults |

### Frontend (React / TypeScript / JS) — 15 Rule Groups

| Rule | What It Catches |
|------|----------------|
| Existing Patterns | New code breaking established UI patterns |
| Components | Size, naming, side effects in render |
| State & Hooks | Derived state in useEffect, missing cleanup, bad deps |
| TypeScript | `any` usage (BLOCKER), missing strict mode |
| Error Handling | Empty catch blocks, missing loading/error states |
| Security | dangerouslySetInnerHTML, eval, secrets in code |
| Testing | Zero tests for new UI code (BLOCKER) |
| Imports | Unsorted, unused, non-tree-shakeable |
| Keys | Missing keys, array index as key |
| Accessibility | div onClick, missing labels, no alt text |
| Styling | Inline styles, !important, fixed units |
| Comments | Same bloat rules as backend |
| API Contracts | Frontend types don't match backend response |
| Bundle Size | Duplicate libraries, unjustified large deps |
| Breaking UI | Changed component props, route paths |

### PR Hygiene

- Empty/vague title or description
- Missing linked issue/ticket
- PR over 5K lines
- Undocumented changes in new commits (re-review)

## Expected Comment Counts

| PR Size | Expected Comments |
|---------|------------------|
| 1K LOC | 10-30 |
| 5K LOC | 30-60 |
| 9K LOC | 50-100 |

If the review produces fewer comments than expected, the agents re-check.

## Key Design Decisions

- **Never auto-posts** — shows review locally first, posts only on "push"
- **Never approves** — always requests changes or comments. Humans merge.
- **Reads full files** — not just diffs. Context matters for architecture checks.
- **Caches repo context** — first review builds understanding, subsequent reviews are faster.
- **Re-review verifies fixes** — doesn't just re-run; checks each previous comment is resolved.

## Requirements

- GitHub CLI (`gh`) installed and authenticated
- Access to the target repository

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).

## License

MIT
