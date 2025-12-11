# Contributing to Coach Claude

Thank you for your interest in contributing to Coach Claude! This document provides guidelines and instructions for contributing.

## Code of Conduct

Be respectful, inclusive, and constructive in all interactions.

## How to Contribute

### Reporting Bugs

1. Check if the bug has already been reported in [Issues](https://github.com/ClutchEngineering/coach-claude/issues)
2. If not, create a new issue with:
   - Clear title and description
   - Steps to reproduce
   - Expected vs actual behavior
   - System information (OS, Python version, Node version)
   - Relevant logs

### Suggesting Features

1. Check existing [Issues](https://github.com/ClutchEngineering/coach-claude/issues) for similar suggestions
2. Create a new issue with:
   - Clear description of the feature
   - Use case and motivation
   - Proposed implementation (optional)

### Contributing Code

1. **Fork the repository**

2. **Clone your fork**
   ```bash
   git clone https://github.com/YOUR-USERNAME/coach-claude.git
   cd coach-claude
   ```

3. **Create a branch**
   ```bash
   git checkout -b feature/your-feature-name
   ```

4. **Set up development environment**

   Python MCP server:
   ```bash
   cd packages/mcp-server
   pip install -e ".[dev]"
   ```

   Node.js CLI:
   ```bash
   cd packages/cli
   npm install
   npm link
   ```

5. **Make your changes**
   - Follow existing code style
   - Add tests for new functionality
   - Update documentation as needed

6. **Run tests**

   Python:
   ```bash
   cd packages/mcp-server
   pytest
   black src/
   ruff check src/
   ```

   Node.js:
   ```bash
   cd packages/cli
   npm test
   npm run build
   ```

7. **Commit your changes**
   ```bash
   git add .
   git commit -m "Description of changes"
   ```

   Commit message format:
   - Use present tense ("Add feature" not "Added feature")
   - First line: brief summary (50 chars or less)
   - Blank line, then detailed description if needed

8. **Push to your fork**
   ```bash
   git push origin feature/your-feature-name
   ```

9. **Create a Pull Request**
   - Go to the original repository
   - Click "New Pull Request"
   - Select your branch
   - Provide clear description of changes
   - Link related issues

## Development Guidelines

### Code Style

**Python**:
- Use Black for formatting (line length: 100)
- Use Ruff for linting
- Type hints for function signatures
- Docstrings for modules, classes, and functions

**TypeScript**:
- Use Prettier for formatting
- Follow existing conventions
- Type everything (no `any` unless necessary)
- JSDoc comments for exported functions

### Testing

**Python**:
- Use pytest
- Aim for 80%+ coverage
- Test both success and error cases
- Mock external dependencies

**TypeScript**:
- Use Jest
- Test all commands
- Mock database operations
- Test CLI argument parsing

### Documentation

- Update README.md if user-facing changes
- Update relevant docs in `docs/`
- Add JSDoc/docstrings for new code
- Include examples in documentation

### Commit Messages

Format: `type: description`

Types:
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Code style changes (formatting, etc.)
- `refactor`: Code refactoring
- `test`: Adding/updating tests
- `chore`: Maintenance tasks

Examples:
- `feat: add weekly stats view`
- `fix: handle database locked error`
- `docs: update installation guide`

## Project Structure

```
coach-claude/
├── packages/
│   ├── mcp-server/      # Python MCP server
│   │   ├── src/
│   │   │   └── coach_claude/
│   │   │       ├── server.py
│   │   │       ├── database.py
│   │   │       ├── models.py
│   │   │       └── config.py
│   │   └── tests/
│   │
│   └── cli/             # Node.js CLI tool
│       ├── src/
│       │   ├── index.ts
│       │   ├── commands/
│       │   ├── db/
│       │   └── config/
│       └── tests/
│
├── skills/              # Claude skill
│   └── coach-claude/
│       └── SKILL.md
│
└── docs/                # Documentation
    ├── installation.md
    ├── configuration.md
    └── architecture.md
```

## Testing Locally

### Test MCP Server

```bash
cd packages/mcp-server
python -m coach_claude.server &  # Start in background
# Test with Claude Code or MCP inspector
```

### Test CLI

```bash
cd packages/cli
npm link  # Link for testing
coach-claude water 16
coach-claude stats
```

### Test Skill

1. Copy skill to `~/.claude/skills/coach-claude/`
2. Start Claude Code
3. Test in a conversation

## Release Process

(For maintainers)

1. Update version in `package.json` and `pyproject.toml`
2. Update CHANGELOG.md
3. Create git tag: `git tag v1.0.0`
4. Push tag: `git push --tags`
5. Publish to PyPI: `twine upload dist/*`
6. Publish to npm: `npm publish`
7. Create GitHub release

## Questions?

- Open an issue for questions
- Email: hello@clutch.engineering

## License

By contributing, you agree that your contributions will be licensed under the Apache 2.0 License.
