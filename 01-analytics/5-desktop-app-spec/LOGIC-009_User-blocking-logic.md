# User Blocking Logic

## Overview
Describes the process of restricting access for users who violate late cancellation policies.

## Logic Flow
1. **Trigger:** User performs a Late Cancellation (< 12h) or fails to show up (No-show).
2. **State:** Backend sets `is_blocked = true` and `unblock_date = current_date + 7 days`.
3. **UI Enforcement:**
   - All "Book" buttons across the application become `disabled`[cite: 26].
   - Banner displaying "Recordings unavailable until [date]" appears on `SCR-009` and `SCR-004`[cite: 26].
4. **Recovery:** Automatic unblocking once `current_date >= unblock_date`.

## Links
- FR-29, FR-30, US-17[cite: 26].