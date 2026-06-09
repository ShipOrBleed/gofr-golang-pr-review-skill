# GoFr Golang PR Review Skill

Opinionated, comprehensive PR review for **Go (GoFr framework)** backends and **React/TypeScript/JavaScript** frontends.

## What It Does

Reviews pull requests against **140+ specific, enforceable rules** covering:

- **Architecture**: Handler → Service → Store layered architecture enforcement
- **Comment Bloat**: Detects excessive comments (7-8 lines for a 1-line method, verbose file headers, bloated constant docs)
- **Error Handling**: Zero tolerance for panics, nil pointer derefs, swallowed errors, missing error checks
- **Resource Safety**: Goroutine leak detection, connection/response body closure, context propagation
- **Concurrency**: Race conditions, mutex ordering, deadlock prevention, atomic operations, `-race` flag enforcement
- **N+1 Queries**: Detects queries inside loops, enforces batch fetching
- **Pagination**: All list endpoints must have pagination — no unbounded queries
- **Security (OWASP)**: SQL injection, command injection, path traversal, SSRF, secrets pattern detection (`AKIA`, `sk-`, `ghp_`), weak crypto
- **Export Hygiene**: Flags functions/types that should be unexported
- **Database Rules**: No JOINs, no foreign keys, parameterized queries only
- **Redis Justification**: Questions Redis usage when in-memory cache might suffice
- **Breaking Changes**: Detects changed response shapes, removed fields, changed status codes
- **File Organization**: Interfaces in `interface.go`, constants in `constant.go`
- **Testing**: Table-driven tests only, 90% coverage, real-world scenarios, no integration tests
- **GoFr Patterns**: Proper handler signatures, GoFr service client (not raw net/http), structured logging, config access
- **Dependency Health**: Flags vulnerable, deprecated, or unmaintained packages
- **Frontend**: Component standards, TypeScript strictness (`any` = BLOCKER), hook discipline, API contract verification, bundle size awareness
- **PR Hygiene**: Title/description validation, size checks (5K line threshold), linked issue enforcement, draft detection

## Installation

```bash
/install gofr-golang-pr-review-skill
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

## Features

- **Context Caching**: Reads repo architecture docs once, caches locally in `.pr-review-context/`. Subsequent reviews skip context-building.
- **Auto-Detection**: Detects whether PR touches backend (Go), frontend (React/TS/JS), or both — applies relevant rules only.
- **Prepare Before Post**: Shows all review comments locally first. Posts to GitHub only when you say "push"/"post".
- **Inline Comments**: Posts review comments on exact lines via GitHub API — like a real reviewer.
- **Re-Review Mode**: After author pushes fixes, verifies each previous issue is resolved, flags unfixed items, audits new commits for undocumented changes.
- **Severity Classification**: BLOCKER → MAJOR → MINOR → SUGGESTION.
- **Never Approves**: Always requests changes or leaves comments. Humans decide when to merge.
- **Senior SWE Tone**: Comments read like a helpful senior colleague — questions for non-critical issues, direct for critical ones.

## Rule Categories

| Category | Rules | Applies To |
|----------|-------|------------|
| Architecture | Handler/Service/Store separation | Go/GoFr |
| Comment Bloat | File headers, constant docs, function docs | Go + Frontend |
| Error Handling | Panics, nil derefs, swallowed errors | Go/GoFr |
| Resource Safety | Goroutine leaks, defer close, context | Go |
| Concurrency | Race conditions, mutex ordering, atomics | Go |
| N+1 Queries | Queries inside loops, batch enforcement | Go |
| Pagination | Unbounded list endpoints | Go |
| Security (OWASP) | Injection, SSRF, secrets, weak crypto | Go + Frontend |
| Export Hygiene | Unexported by default | Go |
| Database | No JOINs, no foreign keys | Go |
| Redis | Justify over in-memory cache | Go |
| Breaking Changes | Response shapes, status codes, endpoints | Go + Frontend |
| File Organization | interface.go, constant.go | Go |
| Imports | Sorted, no unused, no dot imports | Go + Frontend |
| Testing | Table-driven, 90% coverage, no integration | Go |
| GoFr Patterns | Handler sig, service client, logging | Go/GoFr |
| Dependencies | Vulnerable, deprecated, unmaintained | Go + Frontend |
| Connection Config | Pool sizing, timeouts | Go |
| Components | Functional, single file, size limits | React |
| TypeScript | No `any`, strict mode, proper types | TypeScript |
| Hooks | Effect discipline, cleanup, deps | React |
| API Contracts | Frontend types match backend shapes | React/TS |
| Bundle Size | Duplicate libs, large deps, lazy loading | React/TS/JS |
| Frontend Testing | Mandatory for new UI code | React/TS/JS |
| PR Hygiene | Title, description, size, linked issues | All |
| Changelog | User-facing changes need documentation | All |

## Review Workflow

```
1. /review owner/repo 142          → Initial review (prepares comments locally)
2. User reviews prepared comments   → Adjusts if needed
3. User says "push"                 → Comments posted to GitHub
4. Author fixes issues, pushes      → New commits appear
5. /review owner/repo 142          → Re-review (verifies fixes, audits new changes)
6. Repeat until production-ready
```

## Requirements

- GitHub CLI (`gh`) installed and authenticated
- Access to the target repository

## Contributing

Contributions welcome. Please open an issue or PR at [ShipOrBleed/gofr-golang-pr-review-skill](https://github.com/ShipOrBleed/gofr-golang-pr-review-skill).

## License

MIT
