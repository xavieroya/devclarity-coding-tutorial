# Copilot Instructions — RFC 0016 Protocol‑Composed Dependencies

## Scope (When This Applies)
- Any change involving dependency injection, services, containers, or feature construction.
- New/modified ViewModels, Managers, Coordinators, or SwiftUI Features.

---

## Core Rules (Must)
- Express each dependency as a **small protocol** (e.g., `AnalyticsServiceable`).
- Features declare **minimal composed dependencies**:
  - `typealias Dependencies = A & B & C` with only what they use.
- Dependencies are injected via **initializer injection**.
- **No** `.shared` or global singletons in feature code.

---

## Protocol Types (Must)
- Use **Serviceable** for static dependencies (methods/async only).
- Use **Providing** for dynamic dependencies that expose subscribable state.

---

## Dependency Protocols (Must)
- A dependency protocol exposes a **typed property** for the service:
  - `protocol AnalyticsDependency { var analytics: AnalyticsServiceable { get } }`

---

## Containers (Must)
- Dependency containers are **lightweight structs** with stored properties only.
- Containers **do not** include business logic or side effects.
- Composition is **explicit** when passing to sub‑features.

---

## Local Composition (Must)
- If a new dependency is introduced mid‑tree, build a **local container**
  that composes the new service with existing dependencies.

---

## Ownership & Construction (Must)
- Feature/Manager/ViewModel does **not** instantiate concrete services.
- Concrete services are created at the **composition root** (Coordinator/App).
- Dependencies are stored **privately** in the consumer.

---

## Dynamic Dependencies (Must)
- Managers/ViewModels treat `Providing` dependencies as **read‑only streams**.
- Mutations happen inside the service implementation, not in the consumer.

---

## Testing (Must)
- Dependencies are mocked via protocol conformance.
- Unit tests inject mocks via init and assert behavior without real services.

---

## Anti‑Patterns (Flag These)
- Using `.shared` or globals inside features.
- Overgrown containers acting like service locators.
- Consumers instantiating concrete services directly.
- Containers with logic, network calls, or caching behavior.

---

## Minimal Canonical Example

```swift
protocol AnalyticsServiceable {
    func track(event: String)
}
protocol AnalyticsDependency {
    var analytics: AnalyticsServiceable { get }
}

protocol SessionProviding {
    var sessionPublisher: AnyPublisher<SessionState, Never> { get }
}
protocol SessionDependency {
    var session: SessionProviding { get }
}

struct AppDependencies: AnalyticsDependency, SessionDependency {
    let analytics: AnalyticsServiceable
    let session: SessionProviding
}

final class FeatureViewModel {
    typealias Dependencies = AnalyticsDependency & SessionDependency
    private let dependencies: Dependencies

    init(dependencies: Dependencies) {
        self.dependencies = dependencies
    }

    func didLoad() {
        dependencies.analytics.track(event: "feature_loaded")
    }
}
