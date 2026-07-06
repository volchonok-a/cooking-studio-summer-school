# Push Notifications Flow

## Overview
Describes the Web Push API integration for client alerts[cite: 22].

## Logic Flow
1. **Request:** Upon first login, the system prompts the user to grant push notification permissions[cite: 22].
2. **Types:**
   - **Important:** Class cancellations or account blocking[cite: 22].
   - **General:** Reminders (~24h before class) or unblocking notices[cite: 22].
3. **Fallback:** If permissions are denied, notifications are accessible only via the "Bell" icon on `SCR-010`[cite: 22].

## Links
- FR-25, FR-26, FR-27, NFR-16.