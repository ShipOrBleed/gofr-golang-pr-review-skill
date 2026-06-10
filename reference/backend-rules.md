# Backend Rules Reference (Go / GoFr)

## Architecture
- Handler → Service → Store strict separation
- Handlers ONLY: bind input via c.Bind/c.Param/c.PathParam, call service, return response
- Services: business logic, depend on store interfaces, NO DB imports
- Stores: data access via container, NO business logic
- Handler signature: `func(c *gofr.Context) (any, error)` only

## Error Handling
- Zero tolerance for panic() outside main()
- Every pointer nil-checked before dereference
- Every `if err != nil` must return/log+return/wrap+return — never empty block
- Handlers use GoFr types: http.ErrorEntityNotFound, http.ErrorInvalidParam, http.ErrorMissingParam, http.ErrorEntityAlreadyExist
- Error wrapping: fmt.Errorf("context: %w", err) — NOT fmt.Errorf for new errors
- Every function error return MUST be checked — no `_ = fn()`

## Comment Rules (CRITICAL)
- 5+ comment lines for a 1-3 line function → flag immediately
- First 10 lines of new file: if 5+ lines are comments → flag
- 3+ comment lines above const block → flag
- Self-explanatory functions (GetByID, Create, Update, Delete) need zero or one-line comments
- If file is >30% comments → flag the whole file

## File Organization
- Interfaces in interface.go ONLY
- Constants in constant.go (or constants.go) ONLY
- Constants don't need multi-line comments

## Export Hygiene
- Package-internal functions/types/constants MUST be unexported (lowercase)
- Only export what's registered in routes or used cross-package

## Database
- NO JOINs (JOIN, LEFT JOIN, RIGHT JOIN, INNER JOIN, CROSS JOIN) — separate queries, combine in service
- NO foreign keys (REFERENCES, FOREIGN KEY, ON DELETE/UPDATE CASCADE)
- NO queries inside loops (N+1) — batch with WHERE IN
- ALL list endpoints MUST have pagination (page/limit params, LIMIT in SQL)
- Parameterized queries ONLY — never string concatenation in SQL
- Migration naming: YYYYMMDDHHMMSS_description.go

## Resource Management
- defer rows.Close(), defer resp.Body.Close(), defer f.Close()
- Every goroutine has exit path (context cancel, channel close, timeout)
- Always pass ctx to DB/HTTP/service calls — never context.Background() in handlers

## HTTP & Service Calls
- NO net/http, http.Get, http.Post, http.NewRequest, &http.Client{} — use GoFr service client
- Inter-service calls use gRPC, never HTTP
- HTTP client timeouts MUST be configured

## GoFr Patterns
- c.Bind(&struct{}) for request body — not json.NewDecoder
- c.Param("key") for query, c.PathParam("id") for path
- c.Logger.Infof() for logging — not fmt.Println/log.Println
- c.Config.Get("KEY") for config — not os.Getenv
- Redis usage must be justified over in-memory cache

## Testing
- Table-driven ONLY: []struct{name; ...} + t.Run
- Cases: happy path, validation error, not found, already exists, DB error, edge case
- container.NewMockContainer(t) + gomock
- 90% minimum coverage
- Test ALL error paths
- No integration tests — unit only

## Security (OWASP)
- No hardcoded secrets/tokens (check for AKIA, sk-, ghp_, xoxb-, password=, token=)
- No exec.Command with user input (command injection)
- No filepath.Join with unsanitized user input (path traversal)
- No HTTP calls to user-provided URLs without allowlist (SSRF)
- No md5/sha1 for security — use sha256+
- Input validation at handler level

## Imports
- Sorted: stdlib → external → internal (blank line between groups)
- No unused, dot, or unexplained blank identifier imports
- Line length ≤ 140, function ≤ 100 lines, complexity ≤ 10

## Concurrency
- Shared mutable state needs sync.Mutex/RWMutex/channels
- Multiple mutexes acquired in consistent order (deadlock prevention)
- Simple counters use sync/atomic
- No select{default:} in hot loops
- Tests should use -race flag

## Breaking Changes
- Changed response fields, removed fields, type changes → BLOCKER
- Changed status codes, URL paths, query param names → BLOCKER
- Must be documented with migration plan in PR description
