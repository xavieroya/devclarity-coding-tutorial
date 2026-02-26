# Copilot Instructions — RFC 0023 SwiftUI Feature Architecture

## Scope (When This Applies)
- New or modified SwiftUI features.
- Mixed UIKit/SwiftUI screens.
- Any new or modified Manager, Coordinator, or Feature root view.

---

## Feature Topology (Must)
A Feature consists of:
- **Manager** (`@Observable`): state owner + business logic.
- **Coordinator**: navigation and feature lifecycle.
- **Views**: passive UI observing Manager state.
- **Dependencies/Services**: injected via protocols.

---

## Manager Rules (Must)
- Manager is `@Observable` and is the **single source of truth** for a feature.
- Manager receives **dependencies + coordinator** via `init`.
- Manager owns business logic; Views contain minimal logic.
- Manager exposes state via stored properties; actions via methods.
- Use `CancelBag` for subscriptions.

---

## Coordinator Rules (Must)
- Coordinator creates the Manager and root View.
- Coordinator handles all navigation; Views never navigate.
- Mixed UIKit/SwiftUI must host root view with `UIHostingController`.
- Conform to `*Coordinating` protocol; View/Manager use protocol only.

---

## View Rules (Must)
- Root View **owns** Manager (e.g., `@State private var manager`).
- Root View injects Manager via `.environment(manager)`.
- Subviews access Manager via `@Environment(MyFeatureManager.self)`.
- Views are passive: render state, call Manager actions.

---

## UIKit Interop (Must)
- SwiftUI root view is embedded using `UIHostingController`.
- Use `UIViewRepresentable` only when required.
- Do not use `NavigationStack`/`NavigationPath` for app navigation in mixed flows.

---

## Dependencies (Must)
- Protocol-composed dependencies via `typealias Dependencies = A & B & C`.
- Services are created at the composition root (Coordinator/App).
- No `.shared` or global singletons in feature code.

---

## Testing (Must)
- Unit test Managers in isolation with mocked dependencies/coordinator.
- Validate state changes and side effects, not UI rendering.
- View tests are optional; logic belongs in the Manager.

---

## Anti‑Patterns (Flag These)
- Navigation in Views or Managers (should be Coordinator).
- Views directly instantiating services or Managers.
- Subviews accessing unrelated Manager responsibilities.
- `NavigationStack` used for app navigation in mixed flows.
- Managers exposing UIKit types or leaking UI logic.

---

## Minimal Canonical Example

```swift
protocol MyFeatureCoordinating: AnyObject {
    func dismiss()
    func routeToNext()
}

@Observable
final class MyFeatureManager {
    typealias Dependencies = MyFeatureDependency

    private let dependencies: Dependencies
    private let coordinator: MyFeatureCoordinating
    private let cancelBag = CancelBag()

    private(set) var isLoading = false
    private(set) var message = "Loading..."

    init(dependencies: Dependencies, coordinator: MyFeatureCoordinating) {
        self.dependencies = dependencies
        self.coordinator = coordinator
    }

    func load() {
        isLoading = true
        // async work -> update state
        isLoading = false
    }

    func didTapNext() {
        coordinator.routeToNext()
    }
}

struct MyFeatureView: View {
    @State private var manager: MyFeatureManager

    init(manager: MyFeatureManager) {
        self.manager = manager
    }

    var body: some View {
        VStack {
            Text(manager.message)
            Button("Next") { manager.didTapNext() }
        }
        .environment(manager)
        .task { manager.load() }
    }
}
