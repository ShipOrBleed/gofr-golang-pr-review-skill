---
description: Review a PR against Go/GoFr backend and React/TypeScript/JS frontend standards
allowed-tools: Bash(gh pr view:*), Bash(gh pr diff:*), Bash(gh pr list:*), Bash(gh pr comment:*), Bash(gh pr review:*), Bash(gh api:*), Bash(git log:*), Bash(git blame:*), Bash(git diff:*), Bash(git show:*), Bash(git rev-parse:*), Bash(cat:*), Bash(ls:*), Bash(find:*), Bash(head:*), Bash(wc:*), Read, Glob, Grep, Agent, Write
disable-model-invocation: false
argument-hint: <owner/repo> <pr-number>
---

# PR Review

Strict, in-depth PR review for Go (GoFr) backends and React/TypeScript/JS frontends. You are a Senior SWE with 5-8 years of experience. You own production. Act like it.

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

---

## EXECUTION FLOW — FOLLOW THIS EXACTLY

You MUST follow every step below in order. Do NOT skip steps. Do NOT shortcut. A 9K LOC PR should produce 30-80+ comments, not 4.

---

### STEP 1: Fetch PR Data

Run these in parallel:

```bash
# PR metadata
gh pr view <NUMBER> --repo <OWNER/REPO> --json title,body,state,isDraft,headRefName,baseRefName,files,author,additions,deletions,headRefOid

# Full diff
gh pr diff <NUMBER> --repo <OWNER/REPO>

# HEAD SHA for posting comments later
gh pr view <NUMBER> --repo <OWNER/REPO> --json headRefOid -q '.headRefOid'
```

If PR is closed/merged → stop. If draft → note it, still review.

---

### STEP 2: PR Metadata Review

Check these and collect issues:

**Title:**
- Empty or generic ("fix bug", "update", "changes") → MAJOR
- Doesn't match what the diff actually does → MAJOR

**Description:**
- Empty → BLOCKER
- Vague ("minor fixes", "some changes") → MAJOR
- Doesn't explain WHAT changed and WHY → MAJOR
- New dependency added but not justified in description → MAJOR
- No linked issue/ticket → MINOR

**Size:**
- Over 5000 lines (excluding generated/lockfiles) → flag as too large

---

### STEP 3: Build File Lists

From the PR files, create categorized lists:

```
GO_FILES: all .go files (excluding _test.go)
TEST_FILES: all _test.go files
MIGRATION_FILES: all files in migrations/ directory
FRONTEND_FILES: all .ts, .tsx, .js, .jsx files
STYLE_FILES: all .css, .scss, .module.css files
CONFIG_FILES: go.mod, package.json, tsconfig.json, .env*, Dockerfile
OTHER_FILES: everything else
```

Print the file counts:
```
Files changed: X total
  Go source: N
  Go tests: N
  Migrations: N
  Frontend: N
  Styles: N
  Config: N
  Other: N
```

---

### STEP 4: Repo Context (First Review Only)

Check if `.pr-review-context/<owner>-<repo>.md` exists and is <7 days old.

If NO cached context:
1. Read `README.md`, `CLAUDE.md`, `go.mod`, `package.json` from the repo
2. Read 2 existing handler files, 2 service files, 2 store files to learn patterns
3. Save context to `.pr-review-context/<owner>-<repo>.md`

---

### STEP 5: Dispatch Parallel Review Agents

Launch these agents IN PARALLEL. Each agent reviews a specific aspect across ALL relevant files.

**CRITICAL: Each agent MUST read the FULL content of every file it reviews — not just the diff. The diff tells you WHAT changed, the full file tells you if the change is CORRECT in context.**

#### Agent 1: Architecture & Structure Review (Go files)

```
Prompt for agent:
You are reviewing these Go files from a PR for architecture violations.
PR diff is provided below. For EACH Go file, read the FULL file content, then check:

1. HANDLER LAYER: Does ANY handler contain business logic, direct DB access (c.SQL, c.Redis), or complex conditionals beyond input binding? Flag each instance with file:line.
2. SERVICE LAYER: Does ANY service directly access DB/cache instead of going through store interface? Does it import HTTP/framework packages?
3. STORE LAYER: Does ANY store contain business logic instead of pure data access?
4. FILE ORGANIZATION: Are interfaces defined outside interface.go? Are constants scattered instead of in constant.go?
5. EXPORT HYGIENE: Are there exported functions/types that are only used within the same package? Should they be unexported?

For EACH violation, output: FILE:LINE — description of the issue and how to fix it.
Do NOT return generic observations. Return SPECIFIC file:line issues only.

Files to review: [GO_FILES list]
Diff: [paste diff for Go files]
```

#### Agent 2: Error Handling & Safety Review (Go files)

```
Prompt for agent:
You are reviewing these Go files for error handling and safety issues.
For EACH Go file, read the FULL file content, then check:

1. SWALLOWED ERRORS: Find every `if err != nil` — is the error returned, logged, or wrapped? Flag empty blocks.
2. IGNORED RETURNS: Find `_ = someFunc()` where error is discarded. Flag each one.
3. PANIC USAGE: Any `panic()` outside main()? Flag it.
4. NIL POINTER RISK: Any pointer dereference without nil check? Any method call on a potentially nil receiver?
5. ERROR TYPES: In handlers, are GoFr error types used (http.ErrorEntityNotFound, etc.) or raw errors.New()?
6. ERROR WRAPPING: Is fmt.Errorf used with %w for wrapping? Or is it used to create new errors (should use GoFr custom errors)?
7. RESOURCE CLEANUP: Every rows.Close(), resp.Body.Close(), file.Close() must have defer. Flag missing defers.
8. GOROUTINE LEAKS: Any `go func()` without context cancellation, channel close, or timeout?
9. CONTEXT: Any context.Background() or context.TODO() inside a handler? Should use request context.

For EACH issue, output: FILE:LINE — description and fix.
```

#### Agent 3: Code Quality & Comments Review (Go + Frontend files)

```
Prompt for agent:
You are reviewing code quality and comment bloat. This is the #1 issue — OVER-COMMENTING.
For EACH file, read the FULL content, then check:

1. COMMENT BLOAT — THIS IS CRITICAL:
   a. Count comment lines vs code lines for each function. If comments > code lines, flag it.
   b. Check first 10 lines of every NEW file — if 5+ lines are comments, flag: "Too many header comments, one-liner package comment is enough"
   c. Check above every constant block — if 3+ comment lines, flag: "Constants don't need multi-line comments"
   d. Check above every function — if a function named GetByID/Create/Update/Delete has 3+ comment lines, flag: "Function name is self-explanatory, remove or reduce to one line"
   e. Count TOTAL comment lines in each new file. If >30% of the file is comments, flag the file.

2. DEAD CODE: Commented-out code blocks, unreachable code after return, unused variables
3. DEBUG LOGGING: fmt.Println, log.Println, console.log left in
4. TODO/FIXME: Any without a tracking issue reference
5. MAGIC NUMBERS: Hardcoded values that should be named constants
6. DUPLICATE CODE: Same logic repeated 3+ times

For EACH issue, output: FILE:LINE — description and fix.
You MUST check EVERY file. If a file has zero issues, say "FILE: clean". But CHECK it.
```

#### Agent 4: Database & Query Review (Go files + Migrations)

```
Prompt for agent:
You are reviewing database access patterns and queries.
For EACH Go file and migration file, read FULL content, then check:

1. SQL JOINS: Search for JOIN, LEFT JOIN, RIGHT JOIN, INNER JOIN, CROSS JOIN in any SQL string. Flag ALL of them — joins are not allowed.
2. FOREIGN KEYS: Search for REFERENCES, FOREIGN KEY, ON DELETE CASCADE, ON UPDATE CASCADE in migrations. Flag ALL.
3. N+1 QUERIES: Any for/range loop that contains a SQL query or store method call inside it? Flag as N+1.
4. UNBOUNDED QUERIES: SELECT without LIMIT? List endpoints without pagination parameters? Flag each.
5. PARAMETERIZED QUERIES: Any string concatenation in SQL? Any fmt.Sprintf building SQL? Flag as SQL injection risk.
6. MIGRATION SYNTAX: Correct dialect (? for MySQL, $1 for Postgres)? Naming follows YYYYMMDDHHMMSS_description.go?
7. REDIS USAGE: Any new Redis usage? Flag and ask: "Why Redis instead of in-memory cache?"
8. CONNECTION CLEANUP: All DB rows closed with defer? All response bodies closed?

For EACH issue, output: FILE:LINE — description and fix.
```

#### Agent 5: Testing Review (Test files)

```
Prompt for agent:
You are reviewing test files for completeness and correctness.
For EACH test file, read FULL content, then check:

1. TABLE-DRIVEN: Every test function MUST use []struct{name string; ...} with t.Run. Flag non-table-driven tests.
2. SCENARIO COVERAGE: Does each test cover: happy path, validation error, not found, already exists, DB error, edge case? List missing scenarios.
3. MOCK USAGE: Using container.NewMockContainer(t) and gomock? Or something else?
4. ERROR PATH TESTING: Are error paths tested, not just happy path? Count error test cases vs happy path.
5. MISSING TESTS: For each NEW Go source file, check if a corresponding _test.go exists. If not, BLOCKER.
6. For frontend: Are there tests for new components/hooks? If zero tests for new UI code, BLOCKER.

For EACH issue, output: FILE:LINE — description and fix.
For EACH file missing tests, output: FILE (NO TESTS) — BLOCKER.
```

#### Agent 6: Security & Patterns Review (All files)

```
Prompt for agent:
You are reviewing for security vulnerabilities and pattern violations.
For EACH file, read FULL content, then check:

BACKEND:
1. SECRETS: Search for strings matching: AKIA*, sk-*, ghp_*, xoxb-*, password=, token=, secret=, any base64 that looks like a credential
2. COMMAND INJECTION: exec.Command with user input? os/exec with string concat?
3. PATH TRAVERSAL: filepath.Join with user input containing ..?
4. SSRF: HTTP calls to URLs from user input without allowlist?
5. HTTP CALLS: Any net/http, http.Get, http.Post, http.NewRequest, &http.Client{}? Must use GoFr service client.
6. INTER-SERVICE: HTTP calls between internal services? Must use gRPC.
7. GOFR PATTERNS: json.NewDecoder instead of c.Bind? os.Getenv instead of c.Config.Get? fmt.Println instead of c.Logger?
8. IMPORTS: Sorted in groups (stdlib → external → internal)? Unused imports? Dot imports?

FRONTEND:
9. dangerouslySetInnerHTML without DOMPurify?
10. eval(), new Function(), document.write()?
11. API keys in frontend code?
12. Sensitive data in localStorage?
13. TypeScript `any` usage? Flag EVERY instance.
14. Missing useEffect cleanup for subscriptions/timers?

For EACH issue, output: FILE:LINE — description and fix.
```

#### Agent 7: Frontend Deep Review (Frontend files only — skip if no frontend files)

```
Prompt for agent:
You are reviewing React/TypeScript/JavaScript files.
For EACH frontend file, read FULL content, then check:

1. COMPONENT SIZE: Over 200 lines? Flag for decomposition.
2. STATE: Redundant/derived state stored instead of computed? State lifted higher than needed?
3. EFFECTS: useEffect for derived state (should compute in render)? useEffect for events (use handlers)? Missing deps in dependency array?
4. PERFORMANCE: React.memo/useMemo/useCallback without profiling justification?
5. KEYS: .map() without key? Array index as key on reorderable list?
6. ERROR HANDLING: Async without try/catch? Silently swallowed errors (catch(e){})?
7. ACCESSIBILITY: div with onClick instead of button? Inputs without labels? Images without alt?
8. API CONTRACTS: Do TypeScript types match backend response shapes?
9. EXISTING PATTERNS: Does new code break existing component signatures, route paths, or UI patterns?

For EACH issue, output: FILE:LINE — description and fix.
```

---

### STEP 6: Collect & Deduplicate Results

After ALL agents return:

1. Merge all issues into one list
2. Remove duplicates (same file:line flagged by multiple agents)
3. Sort by file path, then by line number
4. Classify each issue: BLOCKER / MAJOR / MINOR / SUGGESTION

**BLOCKER**: Missing tests, security issues, nil pointer risk, panic, swallowed errors, goroutine leaks, race conditions, breaking API changes, hardcoded secrets, N+1 queries, no PR description
**MAJOR**: Architecture violations, comment bloat (5+ lines for 1-line function), JOINs/foreign keys, raw HTTP, wrong error types, missing cleanup, exported-when-should-be-unexported
**MINOR**: Import sorting, naming, minor style, missing changelog
**SUGGESTION**: Redis justification, design questions, dependency alternatives

---

### STEP 7: Present Review to User

Print the FULL review locally. DO NOT post to GitHub yet.

Format:
```
## PR Review: owner/repo #NUMBER

**Title**: [PR title]
**Author**: [author]
**Size**: +X / -Y lines across N files
**State**: [open/draft]

---

### PR Metadata Issues (if any):
1. [issue] — severity

---

### Code Review Issues (sorted by file):

#### path/to/file.go

**Line 42** (MAJOR):
> This handler is doing direct SQL access via `c.SQL.QueryRowContext`.
> Handlers should only bind input and call the service layer.
> Move this to the store layer and call through the service.

**Line 87** (MAJOR):
> 8 lines of comments above a 2-line function `GetUserByID`.
> The function name is self-explanatory — remove the comment block entirely.

**Lines 102-115** (BLOCKER):
> This `go func()` has no context cancellation or timeout.
> If the parent request finishes, this goroutine leaks.
> Fix: pass `ctx` and select on `ctx.Done()`.

#### path/to/another.go
...

---

### Files With No Issues:
- path/to/clean_file.go ✓
- path/to/another_clean.go ✓

### Files Missing Tests (BLOCKER):
- path/to/new_handler.go — no corresponding test file
- path/to/new_service.go — no corresponding test file

---

### Summary

| Severity | Count |
|----------|-------|
| BLOCKER  | X     |
| MAJOR    | Y     |
| MINOR    | Z     |
| SUGGESTION | W   |

**Total issues: N**

**What's Done Well:**
- [specific positive feedback]

**Production Readiness:** Not ready / Ready with fixes / Ready

---
Type "push" to post all comments to GitHub, or "skip" to discard.
```

**IMPORTANT**: If total issues < 10 on a PR with 50+ changed files, you missed things. Go back and re-check. A 9K LOC PR should have 30-80+ comments minimum.

---

### STEP 8: Post to GitHub (only when user says "push"/"post")

After user confirms:

1. Get HEAD SHA:
```bash
gh pr view <NUMBER> --repo <OWNER/REPO> --json headRefOid -q '.headRefOid'
```

2. For EACH issue, post inline comment:
```bash
gh api repos/{owner}/{repo}/pulls/{pr_number}/comments \
  -f body="comment text" \
  -f commit_id="HEAD_SHA" \
  -f path="path/to/file" \
  -F line=LINE_NUMBER \
  -f side="RIGHT"
```

**Line number derivation from diff:**
- Find hunk header: `@@ -old,count +new_start,count @@`
- `new_start` = first line in new file for that hunk
- Count forward: ` ` (context) and `+` (added) lines increment. `-` lines DON'T exist in new file — skip.
- Target line = `new_start` + count of ` ` and `+` lines to reach your target

**Comment style:**
- Conversational, like a senior colleague
- NO severity tags like [BLOCKER] in the comment
- Explain WHAT is wrong, WHY, HOW to fix
- For non-critical: prefer questions ("What happens if this is nil?")
- For critical: be direct ("This will panic in production — add nil check")
- Group related: "Same issue in lines 45, 67, 89 — fix all"

3. Post summary review:
```bash
# If blockers/majors:
gh pr review <NUMBER> --repo <OWNER/REPO> --request-changes --body "$(cat <<'EOF'
### Review Summary
**Verdict**: Changes Requested
**Stats**: X blockers, Y major, Z minor
**Key Issues:** [top 5 issues]
**What's Good:** [positives]
**Production Readiness:** Not ready
EOF
)"

# If only minor/suggestions:
gh pr review <NUMBER> --repo <OWNER/REPO> --comment --body "$(cat <<'EOF'
### Review Summary
**Verdict**: Looks Good with Minor Comments
**Production Readiness:** Ready with minor fixes
EOF
)"
```

**NEVER use --approve. Always --request-changes or --comment.**

---

### STEP 9: Re-Review (when user invokes /review on same PR again)

1. Fetch previous review comments:
```bash
gh api repos/{owner}/{repo}/pulls/{pr_number}/comments
```

2. Fetch new commits since last review:
```bash
gh pr view <NUMBER> --repo <OWNER/REPO> --json commits
```

3. For EACH previous comment, read the current file and check if fixed:
   - ✅ FIXED — resolved
   - ❌ NOT FIXED — poke again: "Still not addressed from previous review"
   - ⚠️ PARTIALLY FIXED — explain what's remaining

4. Check new commits for:
   - NEW rule violations (run agents again on new changes)
   - Undocumented changes (new behavior not in PR description → flag)
   - PR title still accurate?

5. Present re-review locally, wait for "push" to post.

---

## RULES — DO NOT VIOLATE

1. **Read EVERY file fully** — not just the diff. Context matters.
2. **Check EVERY file** — if you skip files, you're not doing your job. List files with zero issues as "clean" to prove you checked.
3. **Use agents for large PRs** — 50+ files means you MUST use parallel agents. Do NOT try to review everything in a single pass.
4. **Minimum comment expectation** — A 1K LOC PR should have 10-30 comments. A 5K LOC PR should have 30-60. A 9K LOC PR should have 50-100. If you're below these, you're skimming.
5. **DO NOT auto-post** — always show to user first.
6. **Never approve** — always request changes or comment.
7. **Never modify files** — read-only review.
8. **Production readiness is the bar** — if it's not production-ready, say so.
