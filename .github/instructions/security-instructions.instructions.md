---
applyTo: '**'
---
NEVER write environment variables, secrets, or sensitive information in the codebase.
Always use secure methods to manage and access sensitive data, such as environment variables or secret management tools.

If you need to reference sensitive information, use placeholders or comments to indicate where the sensitive data should be injected at runtime.
When documenting work done or code changes, ensure that no sensitive information is excluded from the documentation.

No environment variables, secrets, or sensitive information should be hardcoded in the codebase, this is only allowed for .env files which are not committed to the repository.