# Copilot Instructions — RFC 0020 Unit Testing (Robust)

## Scope (When This Applies)
- Any new or modified unit tests in iOS feature code.
- Any change that affects ViewModel logic, Manager logic, or data transformations.

---

## Core Structure (Must)
- Every test follows **Given / When / Then** with `// Given`, `// When`, `// Then`.
- Tests are deterministic, fast, and isolated.

---

## What To Test (Must)
- ViewModel outputs driven by defined inputs (IO‑MVVM).
- Pure functions and transformations.
- Edge cases, empty states, and error handling.
- Manager state changes and side effects (SwiftUI architecture).

---

## What To Avoid (Must)
- UIKit rendering/layout or view lifecycle behavior.
- Real network, persistence, timers, or system frameworks.
- UI snapshot image testing (out of scope).

---

## Naming Conventions
- `test<Condition><Outcome>`
- Optional: `And<SecondaryOutcome>`

Examples:
- `testShowsErrorStateWhenLoadFails`
- `testShowsEmptyStateAndHidesSpinner`

---

## IO‑MVVM Output Testing (Must)
- Construct `Input` publishers for lifecycle and UI events.
- Call `transform(input:)` once, keep `Output`.
- Assert **only** on output publishers; subjects remain private.
- For list screens: output **full** `NSDiffableDataSourceSnapshot`.

---

## Snapshot History Pattern (Must)
- Collect snapshots in a local array within the sink.
- Assert in the Then block after expectations are fulfilled.
- Do **not** assert inside the sink.

---

## Determinism & Speed (Must)
- Use mocks/fakes for dependencies.
- Avoid real timeouts; use a shared timeout constant.
- Prefer synchronous control over async when feasible.

---

## SwiftUI Manager Testing (Must)
- Construct Manager with mocked dependencies and test coordinator.
- Trigger manager actions directly.
- Assert on manager state and side effects.
- Avoid rendering SwiftUI views in unit tests.

---

## Swift Testing (Optional, Allowed)
- Use `import Testing` for async and parameterized tests.
- Use `#expect` assertions and keep camelCase names.

---

## Anti‑Patterns (Flag These)
- Assertions embedded in sinks or Given block.
- Tests depending on real network, disk, timers, or UIKit.
- Direct access to ViewModel private subjects or state.
- Multiple, tangled expectations that obscure linear flow.

---

## Minimal Example (Given/When/Then)

```swift
func testWhenLoadFailsThenShowsErrorState() {
    // Given
    let input = MyViewModel.Input(
        viewDidLoad: viewDidLoadSubject.eraseToAnyPublisher(),
        didTapRetry: retrySubject.eraseToAnyPublisher()
    )
    let output = viewModel.transform(input: input)
    var isErrorStates: [Bool] = []

    output.isError
        .sink { isErrorStates.append($0) }
        .store(in: cancelBag)

    // When
    viewDidLoadSubject.send(())
    // wait for output…

    // Then
    XCTAssertTrue(isErrorStates.contains(true))
}
