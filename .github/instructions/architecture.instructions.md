---
applyTo: "**"
---


# Architecture & Patterns

- Modular, feature-based folder structure (e.g., App/Plan, App/Registry, GuestServices/GuestListManager)
- Uses MVVM and Coordinator patterns (see App/Shared/Coordinators)
- SwiftUI and UIKit coexist (SwiftUIUtils, legacy UIKit modules)
- Dependency injection via protocol composition (see MarketplaceCore/Services)
- Shared business logic in MarketplaceCore, MarketplaceFeatures, GuestServices/Shared
- Communication: NotificationCenter, delegates, Combine, closures
- External integrations: Firebase, Google, in-app purchases, push notifications
- App entry: App/main.swift, App/FakeAppDelegate.swift
- Testing: UnitTests, UITests, TestUtilities