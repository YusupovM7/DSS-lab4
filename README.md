# Tashkent Bike Sharing Network

This repository implements a console-based bike-sharing network simulation following strict encapsulation, domain rules, and object-oriented practices.

## Relationships Table (UML Specification)

| Pair | Relationship | Multiplicity | Verification Line Hint |
| :--- | :--- | :--- | :--- |
| `Network` &rarr; `Station` | Composition | 1 &rarr; * | `self._stations = [...]` in `Network.__init__` |
| `Station` &rarr; `Dock` | Composition | 1 &rarr; * | `self._docks = tuple(...)` in `Station.__init__` |
| `Dock` &rarr; `Bike` | Association | 0..1 &rarr; 0..1 | `self._bike = bike` in `Dock.dock_bike` |
| `Network` &rarr; `User` | Aggregation | 1 &rarr; * | Users pass into Network methods and survive outside |
| `User` &rarr; `Ride` | Association | 1 &rarr; 0..1 | `self._active_ride = ride` in `User` |
| `Ride` &rarr; `Station` | Dependency | * &rarr; 2 | Passed as parameters during instantiation |

### Lifetime Tests
* **If Network is deleted, do Stations disappear?** Yes. Stations are created inside and are bound to the Network instance lifecycle.
* **If Station is deleted, do Docks disappear?** Yes. Docks are encapsulated inside the station tuple.

---

## Handling the "Exactly One Place" Rule

To guarantee that **a bike is in exactly one place at any given time**, our system establishes a strict single-source-of-truth flow:
1. **Dock Exclusivity:** A `Dock` object can only reference a single `Bike` instance. When a bike is checked out, the `Dock` clears its reference completely (`None`).
2. **State Mutability Barriers:** External code cannot directly assign a bike to a dock or append items to station lists because all underlying collections are protected using private variables (`_bike`, `_docks`) and exposed only via custom read-only properties or immutable tuple copies.
3. **Atomic Transfers:** The `Network` coordinator updates the `Bike` status to `IN_USE` simultaneously when removing it from a dock, ensuring it cannot be flagged as docked anywhere else while active.
