# Cancellation and Penalty Rules

## Overview
Defines the business logic for booking cancellations and the application of user-blocking penalties.

## Logic Flow
1. **Request:** User triggers a cancellation from `SCR-006`.
2. **Pre-check:** System calls `GET /bookings/{id}/cancellation-check` to verify the time remaining until the class start.
3. **Threshold Calculation:**
   - **Early Cancellation (>= 12h):** No penalty applied.
   - **Late Cancellation (< 12h):** Penalty triggered.
4. **Action:** If confirmed, the system executes `DELETE /bookings/{id}`.

## Technical Details
- **Penalty Logic:** A Late Cancellation leads to a 7-day account ban.
- **Idempotency:** Cancellation requests must use the `Idempotency-Key` header to ensure safe retries.

## Related Screens
- SCR-006 (My Bookings), SCR-007 (Cancellation Confirmation).

## Links
- FR-19, FR-20, FR-29.