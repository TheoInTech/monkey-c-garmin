# Contributing to monkey-c-garmin

Thanks for your interest in improving the Monkey C / Garmin Connect IQ skill! This guide will help you get started.

## How to contribute

### Reporting issues

- Use the [bug report template](.github/ISSUE_TEMPLATE/bug_report.md) for errors or incorrect information
- Use the [feature request template](.github/ISSUE_TEMPLATE/feature_request.md) for new ideas
- Search existing issues before opening a new one

### Submitting changes

1. **Fork** the repository
2. **Create a branch** from `main` (`git checkout -b your-branch-name`)
3. **Make your changes** — see [what to contribute](#what-to-contribute) below
4. **Test your changes** — verify Monkey C code samples compile and are syntactically correct
5. **Commit** with a clear message describing what and why
6. **Open a Pull Request** against `main`

### What to contribute

This is a reference/documentation skill. Valuable contributions include:

- **Fixing inaccuracies** — incorrect API signatures, outdated syntax, wrong module references
- **Adding missing coverage** — Toybox modules, device-specific behaviors, new Connect IQ SDK features
- **Improving code examples** — fixing bugs in templates, adding edge case handling
- **Clarifying explanations** — rewording confusing sections, adding context

### Style guidelines

- Keep language concise and scannable — this is a reference, not a tutorial
- Use fenced code blocks with `mc` language tag for Monkey C examples
- Follow the existing structure in `SKILL.md` and `references/`
- Prefer real-world, compilable code over pseudocode

### What we won't merge

- Changes that break the progressive disclosure structure (SKILL.md -> references/)
- Large reformatting PRs with no content changes
- Content not related to Monkey C or Garmin Connect IQ development

## Code of Conduct

This project follows the [Contributor Covenant Code of Conduct](CODE_OF_CONDUCT.md). By participating, you agree to uphold it.

## Questions?

Open an issue — we're happy to help.
