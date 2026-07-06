# Auth Flow

## Overview
Describes the user authentication process and session management.

## Logic Flow
1. **Login:** User submits credentials via `SCR-001`.
2. **Session:** The backend issues `HttpOnly`, `Secure`, and `SameSite=Strict` cookies to prevent XSS attacks.
3. **Storage:** No tokens are stored in `localStorage` or `sessionStorage`[cite: 22].
4. **Fallback:** If OAuth (Google/Yandex) is unavailable, the system defaults to Email+Password authentication[cite: 25].

## Links
- FR-01, FR-02, NFR-14.