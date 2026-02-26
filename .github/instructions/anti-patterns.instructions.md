---
applyTo: "**"
---


# Anti-Patterns

- Logging sensitive data (PII, auth tokens, payment info)
- Hardcoded secrets or API keys in source (use environment/config)
- Non-parameterized SQL queries (risk of injection)
- Force-unwrapping optionals without checks
- Massive ViewControllers/ViewModels (split by feature)
- Business logic in views/controllers (should be in models/services)
- Direct use of singletons for dependencies (prefer DI)
- Blocking UI on network calls (use async/Combine)
- Copy-pasting code between modules (prefer shared utilities)