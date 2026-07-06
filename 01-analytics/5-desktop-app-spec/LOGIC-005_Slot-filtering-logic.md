# Slot Filtering Logic

## Overview
Describes how the application filters the list of available classes on `SCR-002`[cite: 23].

## Implementation Strategy
- **Calendar Filters:** Handled via API queries (`GET /slots?date_from=...&date_to=...`)[cite: 23, 27].
- **Local Filtering:** Searching by program name or chef is performed in-memory on the client side for instant UI response[cite: 27].
- **Debounce:** Input for search uses a 300ms debounce to optimize performance[cite: 27].

## Links
- FR-06, US-4[cite: 23].