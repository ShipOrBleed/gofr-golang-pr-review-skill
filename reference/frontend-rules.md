# Frontend Rules Reference (React / TypeScript / JavaScript)

## Components
- Functional only (class only for Error Boundaries)
- One per file, filename = PascalCase component name
- Under 200 lines — decompose if larger
- No side effects during rendering
- Event handlers: handle prefix (handleClick, handleSubmit)
- Boolean props: is/has/should prefix

## State & Hooks
- State as local as possible
- No redundant/derived state — compute during render
- No useEffect for derived state or event handling
- Complete dependency arrays — no missing deps
- No eslint-disable react-hooks/exhaustive-deps without justification
- Every subscription/listener/timer has cleanup return
- memo/useMemo/useCallback only after profiling

## TypeScript
- `any` is NEVER acceptable — use unknown
- No @ts-ignore/@ts-expect-error without justification
- strict: true in tsconfig
- Type assertions (as Type) sparingly with comments
- Props: ComponentNameProps naming
- Prefer as const over enum
- ?? instead of || for defaults
- Exported functions have explicit return types

## Error Handling
- All async has try/catch or .catch()
- Never catch(e){} (empty)
- 4xx vs 5xx handled differently
- Loading, error, empty states for every async op
- Top-level Error Boundary exists

## Security
- No dangerouslySetInnerHTML without DOMPurify
- No eval(), new Function(), document.write()
- No API keys/secrets in frontend
- No sensitive data in localStorage
- URL params validated before use

## Testing (MANDATORY)
- New components/hooks MUST have tests — zero tests = BLOCKER
- React Testing Library — test behavior not implementation
- getByRole, getByLabelText, getByText over getByTestId
- Test interactions and conditional rendering
- Custom hooks need unit tests

## Imports & Bundle
- Sorted: React → third-party → internal → relative → types → styles
- No unused imports
- Tree-shakeable: import { x } from 'lib' not import lib from 'lib'
- No duplicate libraries (moment + date-fns, lodash + lodash-es)
- Route-level code splitting for large features
- New deps must be justified

## Keys & Lists
- Every .map() has key prop
- Stable IDs — NEVER array index if reorderable
- Not generated during render (Math.random())

## Accessibility
- Semantic HTML: button not div onClick
- All inputs have label
- Images have meaningful alt
- Interactive elements keyboard-accessible

## Styling
- No inline styles except dynamic runtime values
- No !important
- Relative units (rem, %)
- Follow existing methodology (CSS Modules, Tailwind, etc.)

## API Contracts
- TypeScript types must match backend response shapes
- API error shapes handled correctly
- New deps > 50KB gzipped need lazy loading

## Breaking Changes
- Shared component prop changes → update all consumers
- Route path changes → migration plan
- CSS class name changes if part of public API

## Comments
- Same rules as backend — no excessive comments
- Self-documenting via good naming
- JSDoc only for public APIs and complex logic
