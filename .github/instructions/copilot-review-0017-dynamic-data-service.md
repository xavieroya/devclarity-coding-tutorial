# Copilot Instructions — RFC 0017 Dynamic Data Service
**[RFC 0017 (Dynamic Data Service)](https://github.com/tkww/orion-rfcs/blob/main/mobile-ios/0017-dynamic-data-service.md)**

## Scope (When This Applies)
- Any change that introduces or modifies a Dynamic Data Service.
- Any feature consuming session-scoped shared data from services.
- Any references to a `.shared` entity.

---

## Dynamic Data Service Model (Must)
- Dynamic Data Services are the single source of truth for session-scoped data.
- Data state is dynamic and subscribable.
- No persistence is assumed beyond the app session.

---

## Data Model Rules (Must)
- Dynamic models are immutable `struct` types.
- Dynamic models conform to `Codable`.
- Avoid mutable model patterns and SwiftyJSON-style mutable data usage.

---

## Service Contracts (Must)
- Service exposes current state and a subscribable publisher via `DynamicDataProviding`.
- Service exposes mutation/fetch API via `DynamicDataServiceable`.
- Dependency access is protocol-based ([RFC 0016](https://github.com/tkww/orion-rfcs/blob/main/mobile-ios/0016-protocol-composition-dependency.md) style), not concrete type-coupled.

---

## Fetch and Mutation Behavior (Must)
- `fetch()` refreshes service state from network source of truth.
- Mutations (`update`, `add`, `remove`) route through the service, then refresh/publish updated state.
- Consumers react to publisher output rather than mutating shared data directly.

---

## Dependency and Ownership Rules (Must)
- Dynamic Data Services are injected via dependency protocols at composition root.
- Coordinators/assembly layers create and pass dependencies ([RFC 0013](https://github.com/tkww/orion-rfcs/blob/main/mobile-ios/0013-coordinators.md) + [RFC 0016](https://github.com/tkww/orion-rfcs/blob/main/mobile-ios/0016-protocol-composition-dependency.md)).
- ViewModels/Managers do not instantiate concrete dynamic/network services.
- No `.shared` access from feature logic.

---

## Concurrency and API Rules (Must)
- Networking layer uses `async/await`.
- Service mutation path returns one-shot result contracts as defined by the service protocol.
- Service encapsulates mutation and state updates; consumers only read/subscribe to state and invoke intents, and never mutate service state directly.

---

## Testing Expectations (Must)
- Unit test Dynamic Data Services as isolated service units.
- Unit test consumers (ViewModels/Managers) through input/output behavior.
- Mock `DynamicDataServiceable` via protocol conformance.
- Validate state/publisher updates, mutation invocation, and error handling paths.

---

## Anti-Patterns (Flag These)
- Memory-Backed Store, NotificationCenter, or other mutable shared cache as runtime source of truth.
- Shared mutable model state accessed from multiple features directly.
- Consumers directly mutating data that should be owned by Dynamic Data Service.
- Consumers instantiating networking/service concretes.
- Missing publisher-driven update handling in consumers.

---

## Minimal Canonical Example

```swift
import Combine

public protocol DynamicDataProviding {
    associatedtype Item
    typealias DataResult = Result<[Item], Error>

    var dynamicData: DataResult { get }
    var dataPublisher: Published<DataResult>.Publisher { get }
    func flush()
}

public protocol TodoDynamicServiceable: DynamicDataProviding where Item == Todo {
    func fetch()
    @discardableResult func update(with item: Todo) -> Future<Todo, Error>
    @discardableResult func add(_ item: Todo) -> Future<Todo, Error>
    @discardableResult func remove(_ item: Todo) -> Future<Todo, Error>
}

public protocol TodoDynamicServiceDependency {
    var todoDynamicService: any TodoDynamicServiceable { get }
}
```

```swift
import Combine

struct Todo: Codable, Equatable {
    let id: String
    let title: String
}

protocol TodoNetworkingServiceable {
    func getTodos() async throws -> [Todo]
    func create(_ todo: Todo) async throws -> Todo
    func update(_ todo: Todo) async throws -> Todo
    func delete(_ todo: Todo) async throws -> Todo
}

final class TodoDynamicService: TodoDynamicServiceable {
    typealias Item = Todo
    typealias DataResult = Result<[Todo], Error>

    @Published private(set) var dynamicData: DataResult = .success([])
    var dataPublisher: Published<DataResult>.Publisher { $dynamicData }

    private let networking: TodoNetworkingServiceable

    init(networking: TodoNetworkingServiceable) {
        self.networking = networking
    }

    func fetch() {
        Task {
            do { dynamicData = .success(try await networking.getTodos()) }
            catch { dynamicData = .failure(error) }
        }
    }

    @discardableResult
    func add(_ item: Todo) -> Future<Todo, Error> {
        Future { [weak self] promise in
            Task {
                guard let self else { return }
                do {
                    let added = try await self.networking.create(item)
                    self.fetch()
                    promise(.success(added))
                } catch {
                    self.dynamicData = .failure(error)
                    promise(.failure(error))
                }
            }
        }
    }

    @discardableResult
    func update(with item: Todo) -> Future<Todo, Error> {
        Future { [weak self] promise in
            Task {
                guard let self else { return }
                do {
                    let updated = try await self.networking.update(item)
                    self.fetch()
                    promise(.success(updated))
                } catch {
                    self.dynamicData = .failure(error)
                    promise(.failure(error))
                }
            }
        }
    }

    @discardableResult
    func remove(_ item: Todo) -> Future<Todo, Error> {
        Future { [weak self] promise in
            Task {
                guard let self else { return }
                do {
                    let removed = try await self.networking.delete(item)
                    self.fetch()
                    promise(.success(removed))
                } catch {
                    self.dynamicData = .failure(error)
                    promise(.failure(error))
                }
            }
        }
    }

    func flush() {
        dynamicData = .success([])
    }
}
```

```swift
import Combine

final class TodoListViewModel {
    typealias Dependencies = TodoDynamicServiceDependency

    private let dependencies: Dependencies
    private var cancelBag = Set<AnyCancellable>()
    @Published private(set) var todos: [Todo] = []

    init(dependencies: Dependencies) {
        self.dependencies = dependencies

        dependencies.todoDynamicService.dataPublisher
            .sink { [weak self] result in
                guard case let .success(items) = result else { return }
                self?.todos = items
            }
            .store(in: &cancelBag)
    }

    func onViewDidLoad() {
        dependencies.todoDynamicService.fetch()
    }

    func didTapAdd(_ item: Todo) {
        _ = dependencies.todoDynamicService.add(item)
    }
}
```

```swift
import Combine
import XCTest

final class MockTodoDynamicService: TodoDynamicServiceable {
    typealias Item = Todo
    @Published var dynamicData: Result<[Todo], Error> = .success([])
    var dataPublisher: Published<Result<[Todo], Error>>.Publisher { $dynamicData }

    var fetchCalled = false
    func fetch() { fetchCalled = true }
    func flush() { dynamicData = .success([]) }

    @discardableResult
    func add(_ item: Todo) -> Future<Todo, Error> { Future { $0(.success(item)) } }
    @discardableResult
    func update(with item: Todo) -> Future<Todo, Error> { Future { $0(.success(item)) } }
    @discardableResult
    func remove(_ item: Todo) -> Future<Todo, Error> { Future { $0(.success(item)) } }
}
```
