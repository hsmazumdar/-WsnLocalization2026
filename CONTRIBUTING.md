# Contributing to WSN Localization Project

Thank you for your interest in contributing to the WSN Localization project! This document provides guidelines and instructions for contributing.

## Code of Conduct

By participating in this project, you agree to maintain a respectful and professional environment. We are committed to providing a welcoming and inclusive experience for all contributors.

## How to Contribute

### Reporting Issues

If you find a bug or have a suggestion for improvement:

1. Check if the issue already exists in the [Issues](https://github.com/hsmazumdar/WsnLocalization/issues) section
2. If not, create a new issue with:
   - Clear description of the problem or feature request
   - Steps to reproduce (for bugs)
   - Expected vs. actual behavior
   - Environment details (OS, .NET version, etc.)

### Contributing Code

#### 1. Fork the Repository

1. Click the "Fork" button on the GitHub repository page
2. Clone your fork locally:
   ```bash
   git clone https://github.com/YOUR_USERNAME/WsnLocalization.git
   cd WsnLocalization
   ```

#### 2. Create a Branch

Create a feature branch for your changes:

```bash
git checkout -b feature/your-feature-name
# or
git checkout -b fix/your-bug-fix
```

**Branch naming conventions:**
- `feature/` - New features
- `fix/` - Bug fixes
- `docs/` - Documentation updates
- `refactor/` - Code refactoring
- `test/` - Test additions/updates

#### 3. Make Your Changes

- Follow the existing code style and conventions
- Write clear, self-documenting code
- Add comments for complex logic
- Update documentation if needed
- Test your changes thoroughly

#### 4. Commit Your Changes

Write clear, descriptive commit messages:

```bash
git add .
git commit -m "Brief description of changes

More detailed explanation if needed:
- What was changed
- Why it was changed
- Any relevant context"
```

**Commit message guidelines:**
- Use imperative mood ("Add feature" not "Added feature")
- Keep first line under 50 characters
- Reference issue numbers if applicable: "Fix #123"

#### 5. Push and Create Pull Request

```bash
git push origin feature/your-feature-name
```

Then:
1. Go to the original repository on GitHub
2. Click "New Pull Request"
3. Select your branch
4. Fill out the PR template with:
   - Description of changes
   - Related issue numbers
   - Testing performed
   - Screenshots (if UI changes)

## Coding Standards

### C# Style Guidelines

- Follow Microsoft C# coding conventions
- Use meaningful variable and method names
- Keep methods focused and under 50 lines when possible
- Add XML documentation comments for public methods
- Use `PascalCase` for public members, `camelCase` for private members

### Code Organization

- Keep related functionality together
- Separate concerns (UI, logic, data)
- Use appropriate design patterns
- Avoid code duplication

### Documentation

- Update README.md if adding new features
- Add XML comments for public APIs
- Document complex algorithms
- Keep inline comments concise and meaningful

## Testing

Before submitting a pull request:

- [ ] Test your changes locally
- [ ] Ensure existing functionality still works
- [ ] Test edge cases and error conditions
- [ ] Verify the application builds without errors
- [ ] Check for any compiler warnings

## Pull Request Process

1. **Ensure your code is up to date:**
   ```bash
   git checkout main
   git pull upstream main
   git checkout feature/your-feature-name
   git rebase main
   ```

2. **Review your changes:**
   - Review the diff yourself
   - Ensure all files are necessary
   - Remove any debug code or comments

3. **Submit the PR:**
   - Fill out the PR template completely
   - Link related issues
   - Request review from maintainers

4. **Respond to feedback:**
   - Address review comments promptly
   - Make requested changes
   - Update the PR as needed

## Review Criteria

Your contribution will be reviewed for:

- ✅ Code quality and style
- ✅ Functionality and correctness
- ✅ Documentation completeness
- ✅ Test coverage
- ✅ Performance considerations
- ✅ Backward compatibility

## License

By contributing, you agree that your contributions will be licensed under the same license as the project.

## Questions?

If you have questions about contributing:

- Open an issue with the `question` label
- Contact the maintainer: H.S.Mazumdar
- Check existing documentation in the `documents/` folder

## Recognition

Contributors will be acknowledged in:
- Project README.md
- Release notes (for significant contributions)
- Academic publications (if applicable)

Thank you for contributing to the WSN Localization project!
