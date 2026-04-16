# Contributing to monkey-c-garmin

Thanks for your interest in improving this Monkey C / Garmin Connect IQ skill! This guide will help you get started.

## How to contribute

### Reporting issues

- Use the [bug report template](.github/ISSUE_TEMPLATE/bug_report.md) for errors or inaccuracies
- Use the [feature request template](.github/ISSUE_TEMPLATE/feature_request.md) for new ideas
- Search existing issues before creating a new one

### Submitting changes

1. **Fork** the repository
2. **Create a branch** from `main` (`git checkout -b fix/your-change`)
3. **Make your changes** — see guidelines below
4. **Commit** with a clear message describing what and why
5. **Push** to your fork and open a **Pull Request** against `main`

### Branch naming

Use a prefix that describes the type of change:

- `fix/` — corrections to existing content
- `feat/` — new reference material or sections
- `docs/` — README, CONTRIBUTING, or meta-documentation updates
- `refactor/` — restructuring without content changes

### Content guidelines

This repo is a **reference skill** — accuracy and clarity are critical.

- **Verify against official Garmin docs** — all API references, method signatures, and behavior descriptions should match the [Connect IQ API Docs](https://developer.garmin.com/connect-iq/api-docs/)
- **Keep examples working** — code snippets should compile and run on real devices
- **Follow the existing structure** — SKILL.md covers essentials, `references/` files go deeper
- **Be concise** — AI agents have limited context windows; avoid unnecessary verbosity
- **Use Monkey C conventions** — PascalCase methods, apostrophe comments in examples, etc.

### What makes a good contribution

- Fixing inaccurate API references or outdated information
- Adding coverage for Toybox modules or features not yet documented
- Improving code templates with better patterns or device compatibility
- Clarifying confusing explanations

### What to avoid

- Adding content that duplicates existing sections
- Making stylistic-only changes (formatting, rewording without clarity improvement)
- Adding content not related to Monkey C or Garmin Connect IQ development

## Code of Conduct

This project follows the [Contributor Covenant Code of Conduct](CODE_OF_CONDUCT.md). By participating, you agree to uphold this standard.

## Questions?

Open an issue with the `question` label and we'll be happy to help.
