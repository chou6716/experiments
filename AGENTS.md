# Repository Guidelines

## Project Structure & Module Organization

This is a small Python project using a `src` layout. The importable package is in `src/experiments/`; its `__init__.py` currently provides the `main()` entry point, and `scratch.py` is for experimental code. `src/scratch.ipynb` is a notebook for exploratory work. `pyproject.toml` defines package metadata and dependencies, while `uv.lock` records the resolved dependency versions. There is no dedicated test or assets directory yet.

## Build, Test, and Development Commands

- `uv sync` installs the locked project dependencies into the local environment.
- `uv run experiments` runs the console script declared in `pyproject.toml`.
- `uv run python -m experiments` runs the package module when a module entry point is added.
- `uv run pytest` can be used to run tests if/when a pytest suite is added; no test framework or test suite is currently configured.

Use `uv` to manage dependencies and keep `uv.lock` in sync when changing `pyproject.toml`.

## Coding Style & Naming Conventions

Use Python with four spaces per indentation level. Name modules and functions in `snake_case`, classes in `PascalCase`, and constants in `UPPER_SNAKE_CASE`. Keep experiments small and focused; move reusable code into clearly named modules under `src/experiments/`. No formatter or linter is configured, so follow standard Python conventions and keep imports explicit.

## Testing Guidelines

There are currently no tests. For new behavior, add pytest tests under a `tests/` directory, name files `test_*.py`, and name test functions `test_*`. Run them with `uv run pytest` after adding pytest as a development dependency.

## Commit & Pull Request Guidelines

The available Git history contains only the initial commit, so no established commit format is evident. Use short, imperative commit subjects, such as `Add calculator example`. Pull requests should explain the change and how it was checked; include screenshots for notebook or other visual changes and link related issues when applicable.

## Security & Configuration

Do not commit API keys or other secrets. Read credentials from environment variables, and avoid placing private data in notebooks or committed outputs.

## LiteLLM Agent Example

This example uses LiteLLM tool calling to build a simple calculator agent. Set `OPENAI_API_KEY` in the environment before running it.

```python
import json
import os

from litellm import completion


def add_numbers(a: float, b: float) -> float:
    """Add two numbers."""
    return a + b


tools = [
    {
        "type": "function",
        "function": {
            "name": "add_numbers",
            "description": "Add two numbers.",
            "parameters": {
                "type": "object",
                "properties": {
                    "a": {"type": "number"},
                    "b": {"type": "number"},
                },
                "required": ["a", "b"],
            },
        },
    }
]


def run_agent(question: str) -> str:
    messages = [
        {
            "role": "system",
            "content": "You are a helpful assistant. Use the calculator when needed.",
        },
        {"role": "user", "content": question},
    ]

    while True:
        response = completion(
            model="openai/gpt-4o-mini",
            messages=messages,
            tools=tools,
            tool_choice="auto",
            api_key=os.environ["OPENAI_API_KEY"],
        )

        message = response.choices[0].message
        messages.append(message.model_dump(exclude_none=True))

        if not message.tool_calls:
            return message.content or ""

        for tool_call in message.tool_calls:
            arguments = json.loads(tool_call.function.arguments)

            if tool_call.function.name == "add_numbers":
                result = add_numbers(**arguments)
            else:
                result = {"error": "Unknown tool"}

            messages.append(
                {
                    "role": "tool",
                    "tool_call_id": tool_call.id,
                    "content": json.dumps(result),
                }
            )


print(run_agent("What is 123 plus 456?"))
```
