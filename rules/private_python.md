# Global Python Best Practices

## Python Standards

### Style & Formatting
- Follow PEP 8 strictly — 4-space indentation, 88-char line limit (Black-compatible).
- Use `ruff` for linting; `black` for formatting. Never submit code that fails either.
- Use `snake_case` for functions and variables, `PascalCase` for classes, `UPPER_SNAKE` for constants.
- Prefer explicit over implicit; readable over clever.

---

### Type Annotations
- Annotate **all** function parameters and return types. No bare `def` signatures.
- Use `from __future__ import annotations` at the top of every module.
- Use `typing` or built-in generics (`list[str]`, `dict[str, int]`) — not `List`, `Dict`.
- Use `Optional[X]` only when the value can truly be `None`; prefer `X | None` (Python 3.10+).

```python
# Good
def get_user(user_id: int) -> User | None:
    ...

# Bad
def get_user(user_id):
    ...
```

---

### Docstrings
- Every public module, class, and function **must** have a docstring.
- Use Google-style docstrings:

```python
def calculate_discount(price: float, rate: float) -> float:
    """Calculate the discounted price.

    Args:
        price: The original price in USD.
        rate: Discount rate as a decimal between 0 and 1.

    Returns:
        The final price after applying the discount.

    Raises:
        ValueError: If rate is not between 0 and 1.
    """
    if not 0 <= rate <= 1:
        raise ValueError(f"Rate must be between 0 and 1, got {rate!r}")
    return price * (1 - rate)
```

- Private methods (`_method`) need a one-line docstring minimum.
- Never use docstrings as a substitute for clear variable/function names.

---

### Exception Handling
- **Never** use bare `except:` or `except Exception:` without re-raising or logging.
- Catch the most specific exception type possible.
- Always include a meaningful message in raised exceptions.
- Use custom exception classes for domain errors:

```python
class InsufficientFundsError(ValueError):
    """Raised when a withdrawal exceeds the available balance."""

    def __init__(self, requested: float, available: float) -> None:
        self.requested = requested
        self.available = available
        super().__init__(
            f"Cannot withdraw {requested:.2f}; only {available:.2f} available."
        )
```

- Use `contextlib.suppress` only when silencing an exception is intentional and documented.
- Log exceptions at the boundary where they are caught, not where they are raised.

---

### Project Structure
```
project/
├── src/
│   └── package_name/
│       ├── __init__.py
│       ├── models.py
│       ├── services.py
│       └── utils.py
├── tests/
│   ├── conftest.py
│   └── test_*.py
├── pyproject.toml
└── README.md
```

- Use `src/` layout to prevent accidental imports from the project root.
- Mirror `src/` structure inside `tests/`.

---

### Imports
- Group imports: stdlib → third-party → local, each separated by a blank line.
- Use absolute imports; avoid relative imports except within the same package.
- Never use wildcard imports (`from module import *`).

---

### Testing
- Use `pytest`. Every public function needs at least one test.
- Name tests `test_<function>_<scenario>`.
- Use `pytest.raises` for expected exceptions; always assert on the message.
- Aim for ≥ 80% coverage; enforce with `pytest-cov --fail-under=80`.

```python
def test_calculate_discount_raises_for_invalid_rate() -> None:
    with pytest.raises(ValueError, match="Rate must be between 0 and 1"):
        calculate_discount(100.0, 1.5)
```

---

### Logging
- Use `logging` module, never `print` in library/service code.
- Configure loggers per module: `logger = logging.getLogger(__name__)`.
- Use structured log messages; avoid f-strings in `logger.debug/info` calls (use `%s` lazy formatting).

---

### Immutability & Safety
- Prefer immutable data: use `tuple` over `list`, `frozenset` over `set` where possible.
- Never use mutable default arguments (`def f(items=[])` → use `None` + body guard).
- Use `dataclasses` or `pydantic` models for structured data; avoid raw dicts for domain objects.

---

## Non-Negotiables
- No `print()` in committed code — use logging.
- No mutable default arguments.
- No missing type annotations on public functions.
- No bare `except`.
- No wildcard imports.
