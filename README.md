# GoFr Golang PR Review Skill

Opinionated, comprehensive PR review for **Go (GoFr framework)** backends and **React/TypeScript/JavaScript** frontends.

## What It Does

Reviews pull requests against 100+ specific, enforceable rules covering:

- **Architecture**: Handler → Service → Store layered architecture enforcement
- **Comment Bloat**: Detects excessive comments (7-8 lines for a 1-line method, verbose file headers, bloated constant docs)
- **Error Handling**: Zero tolerance for panics, nil pointer derefs, swallowed errors, missing error checks
- **Resource Safety**: Goroutine leak detection, connection/response body closure, context propagation
- **Export Hygiene**: Flags functions/types that should be unexported
- **Database Rules**: No JOINs, no foreign keys, parameterized queries only
- **Redis Justification**: Questions Redis usage when in-memory cache might suffice
- **File Organization**: Interfaces in `interface.go`, constants in `constant.go`
- **Testing**: Table-driven tests only, 90% coverage, real-world scenarios, no integration tests
- **GoFr Patterns**: Proper handler signatures, GoFr service client (not raw net/http), structured logging, config access
- **Frontend**: Component standards, TypeScript strictness (`any` = BLOCKER), hook discipline, mandatory tests for new UI code
- **PR Hygiene**: Title/description validation, size checks, draft detection

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

- **Context Caching**: Reads repo architecture docs once, caches locally in `.pr-review-context/`. Subsequent reviews of the same repo skip the context-building step.
- **Auto-Detection**: Detects whether PR touches backend (Go), frontend (React/TS/JS), or both — applies relevant rules only.
- **Inline Comments**: Posts review comments on exact lines via GitHub API.
- **Severity Classification**: BLOCKER → MAJOR → MINOR → SUGGESTION.
- **Never Approves**: Always requests changes or leaves comments. Humans decide when to merge.
- **Conversational Tone**: Comments read like a human reviewer, not a linter.

## Rule Categories

| Category | Rules | Applies To |
|----------|-------|------------|
| Architecture | Handler/Service/Store separation | Go/GoFr |
| Comment Bloat | File headers, constant docs, function docs | Go + Frontend |
| Error Handling | Panics, nil derefs, swallowed errors | Go/GoFr |
| Resource Safety | Goroutine leaks, defer close, context | Go |
| Export Hygiene | Unexported by default | Go |
| Database | No JOINs, no foreign keys | Go |
| Redis | Justify over in-memory cache | Go |
| File Organization | interface.go, constant.go | Go |
| Imports | Sorted, no unused, no dot imports | Go + Frontend |
| Testing | Table-driven, 90% coverage, no integration | Go |
| GoFr Patterns | Handler sig, service client, logging | Go/GoFr |
| Components | Functional, single file, size limits | React |
| TypeScript | No `any`, strict mode, proper types | TypeScript |
| Hooks | Effect discipline, cleanup, deps | React |
| Security | No secrets, no eval, no dangerouslySetInnerHTML | Go + Frontend |
| Frontend Testing | Mandatory for new UI code | React/TS/JS |
| PR Hygiene | Title, description, size, state | All |

## Requirements

- GitHub CLI (`gh`) installed and authenticated
- Access to the target repository

## License

MIT
