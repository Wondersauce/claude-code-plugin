# Python Stack Reference

## Detection Files
- `pyproject.toml` (modern)
- `setup.py` (legacy)
- `requirements.txt`
- `Pipfile`

## Public vs Private API

### Naming Conventions
```python
# Public: No underscore prefix
def public_function():
    pass

class PublicClass:
    pass

# Private: Single underscore prefix (convention)
def _private_helper():
    pass

# Name-mangled: Double underscore prefix
class MyClass:
    def __internal_method(self):  # Becomes _MyClass__internal_method
        pass
```

### __all__ Declaration
```python
# Explicit public API
__all__ = ['PublicClass', 'public_function', 'PUBLIC_CONSTANT']
```
If `__all__` exists, only those items are public.

### Package __init__.py
```python
# Public re-exports
from .module import PublicClass
from .utils import helper_function
```

## Documentation Comments

### Docstring Formats

#### Google Style (Preferred)
```python
def function(param1: str, param2: int = 10) -> bool:
    """Brief description.

    Longer description if needed.

    Args:
        param1: Description of param1.
        param2: Description of param2. Defaults to 10.

    Returns:
        Description of return value.

    Raises:
        ValueError: When param1 is empty.
        TypeError: When param2 is not an integer.

    Example:
        >>> result = function("test", 20)
        >>> print(result)
        True
    """
```

#### NumPy Style
```python
def function(param1, param2):
    """
    Brief description.

    Parameters
    ----------
    param1 : str
        Description of param1.
    param2 : int, optional
        Description of param2. Default is 10.

    Returns
    -------
    bool
        Description of return value.

    Raises
    ------
    ValueError
        When param1 is empty.
    """
```

#### Sphinx Style
```python
def function(param1, param2):
    """Brief description.

    :param param1: Description of param1.
    :type param1: str
    :param param2: Description of param2.
    :type param2: int
    :returns: Description of return value.
    :rtype: bool
    :raises ValueError: When param1 is empty.
    """
```

## Type Hints

### Function Signatures
```python
from typing import Optional, List, Dict, Union, Callable, TypeVar

def process(
    items: List[str],
    callback: Optional[Callable[[str], None]] = None,
    config: Dict[str, Any] | None = None
) -> List[Result]:
    ...
```

### Type Aliases
```python
UserId = int
UserMap = Dict[UserId, User]
Handler = Callable[[Request], Response]
```

### Generics
```python
T = TypeVar('T')
E = TypeVar('E', bound=Exception)

class Result(Generic[T, E]):
    ...
```

### Protocol (Structural Typing)
```python
class Closeable(Protocol):
    def close(self) -> None: ...
```

## Error Handling Patterns

### Custom Exceptions
```python
class AppError(Exception):
    """Base exception for application."""
    pass

class ValidationError(AppError):
    """Raised when validation fails."""
    def __init__(self, field: str, message: str):
        self.field = field
        super().__init__(f"{field}: {message}")
```

### Exception Hierarchy
Document the full exception tree:
```
AppError
├── ValidationError
├── AuthenticationError
└── DatabaseError
    ├── ConnectionError
    └── QueryError
```

## Project Structure Conventions

### Package Layout
```
src/
  mypackage/
    __init__.py      # Public exports
    core.py          # Core implementation
    types.py         # Type definitions
    exceptions.py    # Custom exceptions
    _internal/       # Private modules
    utils/           # Utility functions
tests/               # Exclude from docs
```

### Flat Layout
```
mypackage/
  __init__.py
  module1.py
  module2.py
```

## Common Patterns to Recognize

### Context Managers
```python
class Connection:
    def __enter__(self) -> 'Connection':
        ...
    def __exit__(self, exc_type, exc_val, exc_tb) -> bool:
        ...
```

### Decorators
```python
def retry(max_attempts: int = 3):
    def decorator(func: Callable[P, T]) -> Callable[P, T]:
        ...
    return decorator
```

### Dataclasses
```python
@dataclass
class User:
    name: str
    email: str
    age: int = 0
```

### Pydantic Models
```python
class UserCreate(BaseModel):
    name: str = Field(..., min_length=1)
    email: EmailStr
    age: int = Field(default=0, ge=0)
```

### Abstract Base Classes
```python
from abc import ABC, abstractmethod

class Repository(ABC):
    @abstractmethod
    def get(self, id: str) -> Optional[Entity]:
        ...
```
