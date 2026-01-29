# Node.js/TypeScript Stack Reference

## Detection Files
- `package.json` (primary)
- `tsconfig.json` (TypeScript)
- `.nvmrc`, `.node-version`

## Public vs Private API

### Export Patterns
```typescript
// Public: Explicit exports
export function publicFunc() {}
export class PublicClass {}
export type PublicType = {}
export { namedExport } from './module'
export default MainClass

// Private: No export keyword
function privateFunc() {}
const internalHelper = () => {}
```

### Index File Patterns
Public API is typically re-exported from:
- `src/index.ts` or `index.ts`
- `src/public/index.ts`
- Files referenced in `package.json` exports field

### package.json Exports Field
```json
{
  "exports": {
    ".": "./dist/index.js",
    "./utils": "./dist/utils/index.js"
  }
}
```
Anything in `exports` is public API.

## Documentation Comments

### JSDoc Format
```typescript
/**
 * Brief description on first line.
 *
 * Detailed description follows after blank line.
 *
 * @param name - Parameter description
 * @param options - Options object
 * @param options.timeout - Timeout in ms
 * @returns Description of return value
 * @throws {TypeError} When name is invalid
 * @example
 * const result = myFunction('test', { timeout: 1000 })
 */
```

### TSDoc Additions
```typescript
/**
 * @public - Explicitly marks as public API
 * @internal - Marks as internal (not public)
 * @beta - Unstable API
 * @deprecated Use newFunction instead
 */
```

## Error Handling Patterns

### Custom Error Classes
```typescript
class CustomError extends Error {
  constructor(message: string, public code: string) {
    super(message)
    this.name = 'CustomError'
  }
}
```

### Error Throwing
- Sync functions: `throw new Error()`
- Async functions: rejected promise or thrown error
- Callbacks: `callback(error, null)`

### Error Types to Document
- Input validation errors
- Network/IO errors
- Business logic errors
- Type coercion errors

## Project Structure Conventions

### Common Layouts
```
src/
  index.ts          # Main entry, public exports
  types.ts          # Shared types
  errors.ts         # Error definitions
  utils/            # Internal utilities
  lib/              # Core implementation
  __tests__/        # Tests (exclude from docs)
```

### Monorepo Patterns
- `packages/*/src/index.ts` - Per-package entry
- Shared types in `@org/types` package
- Look for `workspaces` in package.json

## Type Extraction

### Interface/Type Definitions
```typescript
interface UserOptions {
  /** User's display name */
  name: string
  /** Optional email address */
  email?: string
  /** Age in years @default 0 */
  age: number
}
```

### Generic Types
```typescript
type Result<T, E = Error> = { ok: true; value: T } | { ok: false; error: E }
```
Document generic parameters and constraints.

## Common Patterns to Recognize

### Factory Functions
```typescript
export function createClient(config: Config): Client
```

### Builder Pattern
```typescript
class QueryBuilder {
  where(condition: string): this
  limit(n: number): this
  execute(): Promise<Result[]>
}
```

### Middleware Pattern
```typescript
type Middleware = (ctx: Context, next: () => Promise<void>) => Promise<void>
```

### Event Emitters
```typescript
class MyEmitter extends EventEmitter {
  on(event: 'data', listener: (data: Data) => void): this
  on(event: 'error', listener: (err: Error) => void): this
}
```
Document all event types and payloads.
