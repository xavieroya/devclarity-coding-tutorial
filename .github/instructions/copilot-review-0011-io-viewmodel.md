# Copilot Instructions — RFC 0011 IO ViewModel

## Scope (When This Applies)
- UIKit screens and mixed UIKit/SwiftUI screens that use ViewModels.
- Any new or modified ViewModel that represents UI state or handles user events.

If a screen is SwiftUI-only and uses `@Observable`/`ObservableObject`, this file is advisory; for UIKit or mixed screens, it is required.

---

## Core Contract (Must)
- ViewModel defines `Input` and `Output` as `struct`.
- ViewModel exposes **exactly one** entry point: `transform(input:) -> Output`.
- `Input` contains view-driven event publishers (lifecycle, taps, gestures).
- `Output` contains only publishers the view binds to.

**Rationale:** predictable IO boundary; testable with inputs/outputs only.

---

## Encapsulation & Exposure (Must)
- Subjects are **private** inside the ViewModel.
- Outputs are `AnyPublisher` via `eraseToAnyPublisher()`.
- Do **not** expose `@Published` or public subjects from ViewModels.
- ViewModel dependencies are injected in `init`, stored privately.

---

## Cancellation (Must)
- Store Combine subscriptions in `CancelBag` (not `Set<AnyCancellable>`).

---

## Threading & UI Binding (Must)
- UI updates are delivered on the main thread at the **binding boundary** (controller).
- ViewModel may operate on background queues internally unless UI is required.
- Avoid forcing main thread inside the ViewModel unless necessary.

---

## List Screens (Must)
- For diffable data sources, ViewModel outputs a full
  `NSDiffableDataSourceSnapshot<Section, Item>`.
- Do **not** output ad-hoc “reload” or “invalidate” signals.

---

## Binding Pattern (Must)
- Controller owns input subjects.
- Controller calls `transform(input:)` **once** and binds to outputs.
- Controller sends inputs from lifecycle and UI events.
- ViewModel does **not** know about UIKit types.

---

## Dependency Injection (Must)
- Dependencies injected via `init`.
- Prefer protocol-based dependencies.
- No service locators or global singletons for ViewModel logic.

---

## Testing (Must)
- Unit tests drive inputs and assert outputs.
- ViewModel logic is tested without view/controller instantiation.
- Use `CancelBag` to keep test subscriptions alive.

---

## Anti-Patterns (Flag These)
- Public subjects or `@Published` on ViewModels.
- Multiple entry points instead of `transform(input:)`.
- UI types in ViewModel outputs (e.g., `UIColor`, `UIView`).
- List screens without snapshot outputs.
- ViewController reading private ViewModel state directly.

---

## Minimal Template Expectations (Recommended)
Use Xcode templates when available:
- `IO-MVVM.xctemplate` for standard UIKit screens.
- `IO-MVVM-CollectionView.xctemplate` for diffable lists.

---

## Example (Minimal, Canonical)

```swift
final class ExampleViewModel {
    struct Input {
        let viewDidLoad: AnyPublisher<Void, Never>
        let didTapRefresh: AnyPublisher<Void, Never>
    }

    struct Output {
        let title: AnyPublisher<String, Never>
        let isLoading: AnyPublisher<Bool, Never>
    }

    private let titleSubject = PassthroughSubject<String, Never>()
    private let isLoadingSubject = CurrentValueSubject<Bool, Never>(false)
    private let cancelBag = CancelBag()

    func transform(input: Input) -> Output {
        input.viewDidLoad
            .sink { [weak self] in self?.load() }
            .store(in: cancelBag)

        input.didTapRefresh
            .sink { [weak self] in self?.load() }
            .store(in: cancelBag)

        return Output(
            title: titleSubject.eraseToAnyPublisher(),
            isLoading: isLoadingSubject.eraseToAnyPublisher()
        )
    }

    private func load() {
        isLoadingSubject.send(true)
        // async work
        titleSubject.send("Loaded")
        isLoadingSubject.send(false)
    }
}
