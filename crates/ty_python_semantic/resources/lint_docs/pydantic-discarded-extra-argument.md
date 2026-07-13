## What it does

Checks for extra keyword arguments that Pydantic silently discards when a model uses
`extra="ignore"`, either implicitly or explicitly.

## Why is this bad?

A discarded argument has no effect on the constructed model. It often indicates a misspelled field
name or an incorrect assumption about the model's schema.

## Example

```python
from pydantic import BaseModel


class User(BaseModel):
    name: str
    admin: bool = False


user = User(name="Alice", admni=True)  # error: [pydantic-discarded-extra-argument]
```

<details>
<summary>Test fixture</summary>

```toml
[environment]
python = "/src/.venv"
python-version = "3.12"
```

`.venv/<path-to-site-packages>/pydantic/__init__.py`:

```py
from .main import BaseModel as BaseModel
```

`.venv/<path-to-site-packages>/pydantic/main.py`:

```py
from typing import dataclass_transform


@dataclass_transform(kw_only_default=True)
class ModelMetaclass(type): ...


class BaseModel(metaclass=ModelMetaclass):
    def __init__(self, **data: object) -> None: ...
```

</details>

Fix the typo if the field name has been misspelled. Otherwise, use a declared field, remove the
argument, or explicitly configure the model with `extra="allow"` if it should retain arbitrary
extra data.
