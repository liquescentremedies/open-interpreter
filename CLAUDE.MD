# CLAUDE.MD - Open Interpreter Project Guide

## Project Overview

**Open Interpreter** is a Python-based tool that enables Large Language Models (LLMs) to run code locally on your computer. It provides a natural-language interface to execute Python, JavaScript, Shell, and other languages, giving LLMs access to the computer's general-purpose capabilities.

### Key Features
- Execute code locally with full system access
- Support for multiple programming languages
- Interactive terminal interface and Python API
- LLM integration via LiteLLM (supports OpenAI, Anthropic, local models, etc.)
- Computer control capabilities (keyboard, mouse, browser, files, etc.)
- Streaming output support
- Safety mode with user confirmation before code execution

### Repository Information
- **Main Branch**: `main`
- **Python Version**: >=3.9, <4
- **Current Version**: 0.4.3
- **License**: AGPL

## Architecture Overview

### Core Components

1. **`/interpreter/core/`** - Core functionality
   - `core.py` - Main interpreter class and logic
   - `async_core.py` - Asynchronous version of core functionality
   - `default_system_message.py` - System prompt for the LLM
   - `render_message.py` - Message rendering for terminal display

2. **`/interpreter/core/llm/`** - Language model integration
   - LLM communication layer using LiteLLM
   - Handles connections to various LLM providers (OpenAI, Anthropic, local models)
   - Token management and streaming

3. **`/interpreter/core/computer/`** - Computer control modules
   - **`terminal/`** - Code execution in various languages
   - **`browser/`** - Browser automation with Selenium
   - **`files/`** - File system operations
   - **`keyboard/`** - Keyboard control
   - **`mouse/`** - Mouse control
   - **`display/`** - Screen capture and display control
   - **`clipboard/`** - Clipboard operations
   - **`ai/`** - AI-powered capabilities
   - **`vision/`** - Computer vision features
   - **`calendar/`**, **`contacts/`**, **`mail/`**, **`sms/`** - System integrations

4. **`/interpreter/terminal_interface/`** - CLI interface
   - Terminal-based interactive chat interface
   - Profile management
   - Command-line argument parsing

5. **`/interpreter/core/utils/`** - Utilities
   - Telemetry
   - Code scanning
   - Output truncation
   - System debugging

## Directory Structure

```
open-interpreter/
├── interpreter/              # Main package
│   ├── core/                # Core interpreter logic
│   │   ├── computer/        # Computer control modules
│   │   ├── llm/            # LLM integration
│   │   └── utils/          # Utilities
│   └── terminal_interface/  # CLI interface
├── docs/                    # Documentation
├── tests/                   # Test suite
├── examples/               # Example scripts
├── scripts/                # Utility scripts
├── installers/             # Installation scripts
├── pyproject.toml          # Poetry configuration
└── README.md               # Project readme
```

## Development Guidelines

### Code Style
- **Formatter**: Black (target Python 3.11)
- **Import Sorting**: isort with Black profile
- **Pre-commit Hooks**: Configured in `.pre-commit-config.yaml`

### Running Code Formatter
```bash
black .
isort .
```

### Important Patterns

#### 1. Interpreter Instance Usage
```python
from interpreter import interpreter

# Single command execution
interpreter.chat("Create a plot of stock prices")

# Interactive chat
interpreter.chat()

# Streaming
for chunk in interpreter.chat(message, display=False, stream=True):
    print(chunk)
```

#### 2. Configuration
```python
# Model configuration
interpreter.llm.model = "gpt-4"
interpreter.llm.api_key = "your-key"
interpreter.llm.api_base = "http://localhost:1234/v1"

# System message customization
interpreter.system_message += "Additional instructions"

# Auto-run mode
interpreter.auto_run = True  # Skip confirmation prompts

# Offline mode
interpreter.offline = True
```

#### 3. Message History
```python
# Save messages
messages = interpreter.chat("Hello")

# Reset conversation
interpreter.messages = []

# Restore conversation
interpreter.messages = messages
```

### Key Files to Understand

1. **`interpreter/core/core.py`** - Main Interpreter class, message handling, execution flow
2. **`interpreter/core/default_system_message.py`** - System prompt that guides the LLM
3. **`interpreter/core/computer/terminal/terminal.py`** - Code execution engine
4. **`interpreter/terminal_interface/start_terminal_interface.py`** - CLI entry point
5. **`pyproject.toml`** - Dependencies, scripts, and package configuration

## Testing

### Running Tests
```bash
# Run all tests
pytest

# Run specific test file
pytest tests/test_name.py

# Run with verbose output
pytest -v
```

### Test Location
Tests are located in `/tests/` directory.

## Building and Installing

### Development Installation
```bash
# Install with Poetry
poetry install

# Install with extras
poetry install -E os      # OS control features
poetry install -E local   # Local model support
poetry install -E safe    # Safe mode with Semgrep
poetry install -E server  # FastAPI server support
```

### Standard Installation
```bash
pip install open-interpreter
```

### Local Development
```bash
# Install from source
pip install -e .
```

## Entry Points

The package defines several CLI commands in `pyproject.toml`:
- `interpreter` - Main CLI interface
- `i` - Short alias for interpreter
- `interpreter-classic` - Classic interface
- `wtf` - Utility script

## Dependencies

### Core Dependencies
- **LiteLLM** - Multi-provider LLM integration
- **Rich** - Terminal formatting
- **PyYAML** - Configuration files
- **Jupyter** - Kernel for code execution
- **Anthropic** - Claude integration
- **FastAPI/Uvicorn** - Server functionality
- **Selenium** - Browser automation

### Optional Dependencies
- **opencv-python, pytesseract** - Computer vision
- **torch, transformers** - Local model support
- **semgrep** - Code safety scanning

## Configuration Files

### Profile System
- Location: `~/.config/open-interpreter/profiles/`
- Format: YAML files
- Default: `default.yaml`
- Usage: `interpreter --profile my_profile.yaml`

### Example Profile
```yaml
model: gpt-4
auto_run: false
verbose: false
offline: false
```

## Safety and Security

### Important Security Considerations
1. **Code Execution**: Generated code runs in your local environment with full system access
2. **Confirmation Required**: By default, user must approve code before execution
3. **Auto-run Mode**: Use `interpreter -y` or `auto_run = True` with caution
4. **Safe Mode**: Experimental feature using Semgrep for code scanning
5. **Sandboxing**: Consider running in isolated environments (Docker, VM, Colab)

### Safe Mode
- Install with: `pip install open-interpreter[safe]`
- Scans code before execution for potential security issues
- Located in: `docs/SAFE_MODE.md`

## Common Tasks

### Adding a New Language
1. Create language handler in `/interpreter/core/computer/terminal/languages/`
2. Implement required methods (run, stop, etc.)
3. Register in terminal language loader

### Modifying System Message
- Edit: `/interpreter/core/default_system_message.py`
- Or customize at runtime: `interpreter.system_message += "..."`

### Adding Computer Capabilities
1. Create new module in `/interpreter/core/computer/`
2. Implement capability class
3. Integrate with main computer interface

### Debugging
```bash
# Enable verbose mode
interpreter --verbose

# In Python
interpreter.verbose = True

# Mid-chat command
%verbose true
```

## Interactive Mode Commands

Available during chat sessions:
- `%verbose [true/false]` - Toggle verbose mode
- `%reset` - Reset conversation
- `%undo` - Remove last exchange
- `%tokens [prompt]` - Calculate token usage
- `%help` - Show help

## Server Mode

### FastAPI Server
```python
from interpreter import interpreter

# Start built-in server
interpreter.server()

# Or create custom server
from fastapi import FastAPI
from fastapi.responses import StreamingResponse

app = FastAPI()

@app.get("/chat")
def chat_endpoint(message: str):
    def event_stream():
        for result in interpreter.chat(message, stream=True):
            yield f"data: {result}\n\n"
    return StreamingResponse(event_stream(), media_type="text/event-stream")
```

## Local Model Support

### LM Studio Integration
```python
interpreter.offline = True
interpreter.llm.model = "openai/x"
interpreter.llm.api_key = "fake_key"
interpreter.llm.api_base = "http://localhost:1234/v1"
```

### Llamafile
```bash
interpreter --local
```

## Contributing

### Contribution Guidelines
- See: `/docs/CONTRIBUTING.md`
- Format code with Black and isort
- Run pre-commit hooks
- Add tests for new features
- Update documentation

### Roadmap
- See: `/docs/ROADMAP.md`

## Common Issues and Solutions

### Type Errors
- **Issue**: Temperature initialization type error in Llm class
- **Fix**: Ensure temperature is float, not int (fixed in commit 81a9302)

### Import Errors
- Check Python version (>=3.9, <4)
- Verify all dependencies installed: `poetry install`
- For optional features: `poetry install -E os -E local`

### Model Connection Issues
- Verify API key configuration
- Check API base URL for local models
- Ensure model name matches LiteLLM format

## Related Resources

- **Documentation**: https://docs.openinterpreter.com/
- **Discord**: https://discord.gg/Hvz9Axh84z
- **Development Branch**: https://github.com/OpenInterpreter/open-interpreter/tree/development
- **Google Colab Demo**: Available in README

## Notes for AI Assistants

### When Working with This Codebase
1. **Always read files before modifying** - Understand context first
2. **Respect the architecture** - Keep computer capabilities modular
3. **Maintain backward compatibility** - This is a public API
4. **Security first** - Be cautious with code execution features
5. **Test thoroughly** - Code execution bugs can be dangerous
6. **Follow Black/isort formatting** - Use pre-commit hooks
7. **Document changes** - Keep docstrings and comments updated

### Key Design Principles
- **Modularity**: Computer capabilities are separate, composable modules
- **Streaming**: Support streaming output for better UX
- **Flexibility**: Support multiple LLM providers via LiteLLM
- **Safety**: Default to user confirmation for code execution
- **Simplicity**: Natural language interface, minimal configuration

### Common Modification Areas
- Adding language support: `/interpreter/core/computer/terminal/languages/`
- Computer capabilities: `/interpreter/core/computer/`
- LLM integration: `/interpreter/core/llm/`
- System prompts: `/interpreter/core/default_system_message.py`
- CLI interface: `/interpreter/terminal_interface/`

---

**Last Updated**: 2025-12-11
**Version**: 0.4.3
**Maintained by**: OpenInterpreter Community
