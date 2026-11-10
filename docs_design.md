# Object-Oriented Analysis & Design: Tashkent Bike Sharing Network

## Noun Sorting & Classification
Based on the requirement text, nouns have been extracted and classified into Classes, Attributes, Actors, Merged entities, or Vague terms.

* **Station**: Class (Has data like docks, behavior like managing available docks)
* **Dock**: Class (Has status, holds at most one bike, handles locking/unlocking)
* **Bike**: Class (Has unique identity, maintenance status, and usage state)
* **Rider / User**: Class/Actor (Rider represents the actor outside, but the system tracks an internal `User` object with data like active ride status)
* **Ride**: Class (An event with data: start/end stations, timestamps, and encapsulates the payment fee rules)
* **Manager**: Actor (Interacts with the system to see reporting views)
* **Network**: Class (System coordinator that runs the rides and manages stations)
* **Time / Timestamp**: Attribute (Value object stored inside Ride)
* **Fee / Money / Sum**: Attribute (Value rule handled inside Ride)
* **Repair / Status**: Attribute (State of a bike)

---

## CRC Cards (Class-Responsibility-Collaborator)

### 1. Class: Bike
* **Responsibilities:**
  * Track its current operational status (`AVAILABLE`, `IN_USE`, `REPAIR`).
  * Enforce that it can only be sent to repair if it is currently docked and free.
* **Collaborators:** None

### 2. Class: Dock
* **Responsibilities:**
  * Track whether it is free or occupied by a specific bike.
  * Park a bike safely and switch its status to `AVAILABLE`.
  * Prevent external modification of its bike slot (`read-only` enforcement).
  * Allow a bike to remain in the dock while marked under `REPAIR`.
* **Collaborators:** Bike

### 3. Class: Station
* **Responsibilities:**
  * Maintain a distinct, isolated collection of docks (no shared mutable state).
  * Report total docked bikes (`len(station)`) and format string display (`Chilonzor: 4/10 bikes`).
  * Enable sorting based on the number of free docks available.
* **Collaborators:** Dock, Bike

### 4. Class: User (Rider)
* **Responsibilities:**
  * Maintain a link to exactly one active `Ride` object.
  * Prevent starting a new ride if an active ride already exists (`RideActiveError`).
* **Collaborators:** Ride

### 5. Class: Ride (Event Class)
* **Responsibilities:**
  * Record start station, end station, start time, and end time.
  * Own and calculate the fee rules using constants (`15 min free`, `500 sum / 5 min`, `20,000 cap`).
  * Raise `ValueError` if end time precedes start time.
* **Collaborators:** Station

### 6. Class: Network
* **Responsibilities:**
  * Coordinate the process of taking and returning bikes across stations.
  * Maintain failure retry counts for full-station returns (`n = 2`).
  * Issue automated redirects to the station with the most free docks upon failure.
  * Generate manager reports sorted by free docks.
* **Collaborators:** User, Station, Bike, Ride

---

## Scenario Simulation: "Rider returns bike to a full station"
1. **Network** receives the return request from a **User** at a specific **Station**.
2. **Network** queries the target **Station** to check for a free **Dock**.
3. **Station** checks its internal **Dock** objects and determines all are occupied.
4. **Network** increments the full-station failure counter for this specific **Ride**.
5. If failure count < 2: **Network** raises `StationFullError`. The ride clock keeps running.
6. If failure count == 2: **Network** scans all **Station** objects, sorts them by free docks, and embeds a redirect recommendation into the raised error.
