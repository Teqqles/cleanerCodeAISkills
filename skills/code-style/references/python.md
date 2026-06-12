# Code Style: Python

## Formatter

```bash
black .
ruff check --fix .
isort .
```

Typical config files: `pyproject.toml`, `.ruff.toml`

## Naming Conventions (PEP 8)

```python
user_name = 'Alice'                   # snake_case: variables
def calculate_total(): ...            # snake_case: functions
class OrderService: ...               # PascalCase: classes
MAX_RETRIES = 3                       # UPPER_SNAKE_CASE: module-level constants
_internal_helper = ...                # leading underscore: module-private
__dunder__ = ...                      # double underscore: Python protocol methods
```

## Comments

```python
# Bad: describes what, not why
# Loop through users and add to total
total = sum(user.balance for user in users)

# Good: only when why is non-obvious
# Balances can be negative (credit accounts); sum without filtering is intentional
total = sum(user.balance for user in users)
```

## Type Hints as Documentation

```python
# Prefer type hints over comments that describe types
# Bad
def get_user(user_id):  # user_id is a string, returns a User or None
    ...

# Good
def get_user(user_id: str) -> User | None:
    ...
```
