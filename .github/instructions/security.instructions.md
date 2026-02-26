---
applyTo: "**"
---


# Security & Configuration

- Secrets managed via environment variables, not in source
- Sensitive config: Info.plist, .entitlements, GoogleService-Info.plist
- Authentication: OAuth, Firebase Auth, Apple Sign-In
- Authorization: role-based checks in business logic
- API keys: loaded at runtime, not hardcoded
- Compliance: GDPR, CCPA (user data export/delete flows)
- Secure storage: Keychain for tokens, UserDefaults for non-sensitive
- Network: HTTPS enforced, certificate pinning for APIs