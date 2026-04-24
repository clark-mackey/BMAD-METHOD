# Code Review Checklist

ALWAYS check for these issues in code reviews:

## Naming
- Vague/unclear function, variable, or class names
- Names that don't reveal intent

## Documentation
- Missing docstrings, type hints, or comments for complex logic
- Undocumented public APIs

## Side Effects
- Functions that mutate inputs or have unexpected side effects
- Hidden state changes

## Single Responsibility
- Classes/functions doing too many things
- God objects, god functions

## Performance
- Inefficient loops, operations that could use comprehensions or built-ins
- N+1 queries, unnecessary allocations

## Security
- Command injection, unsafe operations
- Input validation gaps
- SQL injection via string formatting
- Missing parameterized queries

## Error Handling
- Missing try/catch, unhandled edge cases
- Silent exception swallowing
- Catching too broad (except Exception catches KeyboardInterrupt)

## Code Style
- Non-Pythonic patterns, unnecessary complexity
- Copy-paste duplication

## Immutability
- Prefer immutable operations where possible
- Unnecessary mutation of shared state

## Type Safety
- Missing type annotations, unclear return types
- Implicit type coercion

