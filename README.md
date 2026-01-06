# IP Tools

Intellectual property data tools for AI agents.

## Installation

```bash
pip install ip-tools
```

## Development

```bash
# Clone the repository
git clone https://github.com/parkerhancock/ip_tools.git
cd ip_tools

# Install dependencies with uv
uv sync --group dev

# Run tests
uv run pytest

# Run linting
uv run ruff check .
uv run ruff format .

# Run type checking
uv run ty check
```

## License

Apache-2.0
