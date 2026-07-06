# Slot Availability Calculation

## Overview
Defines how slot availability is calculated and displayed on `SCR-003`.

## Logic Flow
1. **Real-time Check:** Capacity data is fetched directly from the backend to ensure accuracy.
2. **Dynamic UI:** If `available_places === 0`, the "Book" button is disabled and the status is updated to "No seats available"[cite: 22].
3. **Concurrency:** In case of simultaneous booking requests (409 Conflict), the system triggers an error modal to prevent overbooking[cite: 25].

## Links
- FR-07, FR-09.