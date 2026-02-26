---
applyTo: "**"
---


# Stack Best Practices

- Swift 5.x, Xcode project managed via project.yml (XcodeGen)
- Prefer SwiftUI for new UI, UIKit for legacy modules
- Use MVVM for view logic, Coordinator for navigation
- Dependency injection via protocols, avoid singletons
- Use Combine for reactive flows, NotificationCenter for cross-module events
- Avoid force-unwrapping, use guard/if-let for optionals
- Error handling: Result types, do/catch, never swallow errors silently
- Use Codable for model serialization, avoid manual parsing
- Prefer value types (structs) for models, reference types for controllers
- Unit/UI tests required for new features (see UnitTests, UITests)