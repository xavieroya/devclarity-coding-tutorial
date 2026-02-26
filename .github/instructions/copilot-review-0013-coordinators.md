# Copilot Instructions — RFC 0013 Coordinators

## Scope (When This Applies)
- UIKit or mixed UIKit/SwiftUI flows that use navigation.
- Any new or modified Coordinator, or changes to navigation logic.

---

## Core Contract (Must)
- Coordinators conform to `Coordinator` with:
  - `func start()`
  - `var childCoordinators: [Coordinator] { get set }`
- Optionally use extension helpers for lifecycle:
  - `start(from:)`, `add(childCoordinator:)`, `remove(childCoordinator:)`,
    `remove(childCoordinatorType:)`, `childDidFinish(_:)`, `removeAllChildren()`.

---

## Ownership & Lifecycle (Must)
- A coordinator **owns a flow** and decides which screens appear and when.
- Parent retains child coordinators while active.
- Child must notify parent when finished; parent removes it to avoid leaks.
- Use child coordinators for subflows or reusable flows.

---

## UIKit Flow (Must)
- Coordinator constructs the root view controller in `start()`.
- ViewModel communicates navigation through a `*Coordinating` protocol.
- ViewControllers do **not** call coordinators directly.
- Coordinator pushes/presents via its navigation controller.

---

## SwiftUI Integration (Must)
- SwiftUI screens are hosted via `UIHostingController` inside `start()`.
- Do **not** use `NavigationStack`/`NavigationPath` for app navigation.
- Coordinator remains the single source of navigation truth.

---

## RootViewCoordinator (When Needed)
- Use `RootViewCoordinator` when a coordinator owns a navigation stack:
  - `var navigationController: UINavigationController { get }`

---

## Dependency Injection (Must)
- Dependencies injected into coordinators via `init`.
- Prefer protocol-based dependencies.
- No service locators or global singletons.

---

## ViewModel/Manager Interaction (Must)
- Define `MyFeatureCoordinating` protocol as the navigation contract.
- ViewModel/Manager holds a reference to that protocol only.
- Coordinator is the concrete implementer.

---

## Testing (Must)
- Unit tests assert:
  - `start()` sets the root view and pushes expected VC.
  - Selecting an item starts a child coordinator and retains it.
  - `childDidFinish(_:)` removes the child coordinator.
- Prefer Given/When/Then test style for behavior clarity.

---

## Anti-Patterns (Flag These)
- Navigation logic in ViewControllers or SwiftUI Views.
- Coordinators that do not retain children.
- Child coordinators never removed.
- Using `NavigationStack`/`NavigationPath` for app navigation.
- God coordinators that manage unrelated flows.

---

## Minimal Canonical Example

```swift
protocol MyFeatureCoordinating: AnyObject {
    func didSelectDetails(id: String)
}

final class MyFeatureCoordinator: Coordinator {
    typealias Dependencies = MyFeatureViewModel.Dependencies

    private weak var navigationController: UINavigationController?
    private let dependencies: Dependencies
    var childCoordinators: [Coordinator] = []

    init(dependencies: Dependencies, navigationController: UINavigationController?) {
        self.dependencies = dependencies
        self.navigationController = navigationController
    }

    func start() {
        let viewModel = MyFeatureViewModel(dependencies: dependencies, coordinator: self)
        let rootVC = MyFeatureViewController(viewModel: viewModel)
        navigationController?.pushViewController(rootVC, animated: true)
    }
}

extension MyFeatureCoordinator: MyFeatureCoordinating {
    func didSelectDetails(id: String) {
        let child = DetailsCoordinator(id: id, navigationController: navigationController)
        child.start(from: self)
    }
}
