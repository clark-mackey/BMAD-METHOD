# Functional Programming — Review Gotchas

Patterns to flag during code review. Claude already knows FP theory — this file covers the mistakes people actually make.

## Purity Violations to Flag

- **Hidden side effects in "pure" functions**: Database calls, HTTP requests, logging, or file I/O buried inside functions that look pure. The fix is always: push effects to the boundary, keep core logic pure.
- **Mutation of function arguments**: Especially Python dicts/lists passed to functions that modify them in place. Callers don't expect this. Use `.copy()` or return new structures.
- **Global state access**: Functions reading from or writing to module-level variables, singletons, or class attributes. Makes testing impossible and creates coupling.

## Async/Await FP Traps (Python/FastAPI specific)

- **Mixing sync and async impurity**: Calling sync I/O (e.g., `requests.get`) inside an `async` function blocks the event loop. Use `httpx` async or `asyncio.to_thread()`.
- **Shared mutable state across coroutines**: Multiple `await` points mean other coroutines can mutate shared state between them. Each coroutine needs isolated state or atomic operations.
- **Side effects in list comprehensions/generators**: `[await fetch(url) for url in urls]` runs sequentially. Use `asyncio.gather()` for parallel, but watch for mutation in the gathered callbacks.

## Composition Anti-Patterns

- **Deep nesting instead of pipeline**: 5+ levels of `if/else` or nested function calls. Refactor to a pipeline: validate → transform → persist, each step a pure function.
- **Callback hell disguised as FP**: Passing lambdas 3+ levels deep. Extract named functions — readability > cleverness.
- **Premature abstraction**: Creating monadic wrappers, custom Result types, or effect systems in a 200-line script. Use plain `try/except` or `Optional` returns until complexity demands more.

## Immutability Mistakes

- **Defaulting to mutable containers**: Using `list` when `tuple` suffices, `dict` when `frozenset` or `namedtuple` works. Mutable defaults in function signatures (`def f(items=[])`) is a classic Python bug.
- **"Immutable" objects that aren't**: Dataclasses without `frozen=True`. Pydantic models that allow mutation. TypedDicts that look immutable but aren't enforced.
- **Unnecessary defensive copies**: Copying everything "just in case" when the data path is already immutable. Profile before adding copies.

## Error Handling as Values

- **Bare `except:` (no type)**: Catches `KeyboardInterrupt`, `SystemExit`, and everything else — makes the program uncancellable. `except Exception:` is usually fine (it skips `BaseException` subclasses). The real fix: catch specific exceptions like `except (ValueError, TypeError)`.
- **Silent swallowing in gather**: `asyncio.gather(return_exceptions=True)` followed by ignoring the exception objects in results. Always check and log.
- **Exceptions for control flow**: Using `try/except StopIteration` or `except KeyError` where `if key in dict` or `.get()` is cleaner and faster.

## What NOT to Flag

- Using mutation in hot loops for performance (scoped, local mutation is fine)
- Using `dataclass` without `frozen=True` when mutability is intentional and documented
- Not using monads/Either types in Python — Python idiom is exceptions at boundaries, not pervasive Result types
