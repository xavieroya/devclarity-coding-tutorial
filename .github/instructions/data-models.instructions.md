---
applyTo: "**"
---


# Data Models

- Core entities: Guest, Event, RegistryItem, User, Vendor
- Value objects: Address, RSVPStatus, GiftPreference
- Codable for model serialization/deserialization
- Validation: email, phone, required fields in models
- Relationships: Guests linked to Events, RegistryItems linked to Users
- Data transfer: DTOs for API requests/responses
- Persistence: UserDefaults, CoreData, in-memory stores
- Migration: handled via CoreData or manual scripts