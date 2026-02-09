# Security Rules

## Never Commit
- API keys, tokens, or secrets
- .env files with real credentials
- Private keys or certificates
- Database connection strings with passwords

## Input Validation
- Validate all user input at system boundaries
- Use parameterized queries (never string concatenation for SQL)
- Sanitize HTML output to prevent XSS
- Validate file paths to prevent directory traversal

## Dependencies
- Keep dependencies updated
- Review new dependencies before adding
- Prefer well-maintained packages with security policies
- Run `npm audit` / `pip audit` regularly

## Authentication & Authorization
- Never store passwords in plain text
- Use established auth libraries (don't roll your own)
- Validate permissions on every request
- Use HTTPS for all external communication
