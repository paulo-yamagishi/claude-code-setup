# Python Projects with Claude Code

## Tooling

| Tool | Purpose | Install |
|------|---------|---------|
| **uv** | Fast package manager (recommended) | `curl -LsSf https://astral.sh/uv/install.sh \| sh` |
| **ruff** | Linter + formatter (replaces flake8, black, isort) | `uv pip install ruff` |
| **pytest** | Testing framework | `uv pip install pytest` |
| **mypy** | Static type checker | `uv pip install mypy` |

## Virtual Environments

Always use a virtual environment. Specify activation in your CLAUDE.md:

```bash
# uv (recommended)
uv venv && source .venv/bin/activate

# standard
python -m venv .venv && source .venv/bin/activate

# conda
conda create -n myproject python=3.12 && conda activate myproject
```

Tell Claude which environment to use so it runs commands correctly.

## Project Structure

### src layout (recommended for libraries)

```
myproject/
├── src/
│   └── mypackage/
│       ├── __init__.py
│       └── core.py
├── tests/
│   ├── conftest.py
│   └── test_core.py
├── pyproject.toml
└── CLAUDE.md
```

### Flat layout (fine for applications)

```
myproject/
├── mypackage/
│   ├── __init__.py
│   └── core.py
├── tests/
├── pyproject.toml
└── CLAUDE.md
```

## Style Conventions

- Follow **PEP 8** (ruff enforces this automatically)
- Use **type hints** on all function signatures
- Use **Google-style docstrings**
- Prefer `pathlib.Path` over `os.path`
- Use `dataclasses` or `pydantic` for structured data

```python
from pathlib import Path

def load_config(path: Path) -> dict[str, str]:
    """Load configuration from a TOML file.

    Args:
        path: Path to the configuration file.

    Returns:
        Dictionary of configuration values.
    """
    ...
```

## Testing

```python
# tests/conftest.py — shared fixtures
import pytest

@pytest.fixture
def sample_data() -> list[int]:
    return [1, 2, 3]

# tests/test_core.py
import pytest

@pytest.mark.parametrize("input,expected", [
    (1, 2),
    (2, 4),
])
def test_double(input: int, expected: int) -> None:
    assert double(input) == expected
```

## Common Commands

```bash
pytest                      # run all tests
pytest -x                   # stop on first failure
pytest -k "test_name"       # run specific tests
ruff check .                # lint
ruff format .               # format
mypy src/                   # type check
```

## Dependencies

Use `pyproject.toml` as the single source of truth:

```toml
[project]
dependencies = [
    "requests>=2.31,<3",
    "pydantic>=2.0",
]

[project.optional-dependencies]
dev = ["pytest", "ruff", "mypy"]
```

## Common Patterns to Suggest to Claude

- **dataclasses** for plain data containers
- **pathlib** for all file path operations
- **contextlib.contextmanager** for resource management
- **typing.Protocol** for structural subtyping
- **functools.lru_cache** for memoization
- **logging** over print statements
