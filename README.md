# Bearer of Bad Auth — Full Login Bypass (Argus)

**Portal Used In PoC:** `https://86.48.17.4/`

**PoC Video:**

[![Argus Bearer Token Bypass - PoC Video](https://img.youtube.com/vi/5EVHt983fKE/maxresdefault.jpg)](https://youtu.be/5EVHt983fKE)

---

## Before

<img width="1280" alt="Login Portal Before Bypass" src="https://github.com/user-attachments/assets/16fb3a60-830f-456c-9ffb-ebbdb66df934" />
<img width="1280" alt="Shell Hidden Before Bypass" src="https://github.com/user-attachments/assets/25ab3397-b5f1-40f1-9d29-66d4a1969677" />

## After

<img width="1280" alt="Shell Visible After Bypass" src="https://github.com/user-attachments/assets/2befd23f-8b71-421d-9fb2-848a14bda9ec" />
<img width="1280" alt="Dashboard Access After Bypass" src="https://github.com/user-attachments/assets/165ed100-4c00-417a-948a-b4456ae8be12" />

---

## Overview

This script is a browser console-based attack designed to bypass a single-page application's authentication mechanism through multiple vectors. It targets a web interface (a cryptocurrency mining pool dashboard called "Argus") that uses bearer token authentication.

## Attack Vectors (In Order of Execution)

### 1. Token Extraction from Page Source
The script scans the entire HTML document for tokens using regex patterns that look for:
- Bearer tokens in the format `Bearer [token]`
- Token assignments in various formats (`token: "value"`, `token = "value"`)
- API keys, secrets, and CTF flags
- Long alphanumeric strings (32+ characters) that might be tokens

This exploits the common vulnerability of tokens being inadvertently embedded in client-side code.

### 2. Common Token Location Scanning
It searches predictable DOM locations where tokens might be stored:
- Meta tags (commonly used for CSRF tokens or API keys)
- Hidden input fields (often used to pass authentication data)

### 3. Default/Brute Force Tokens
The script maintains a list of weak/default passwords and predictable token values including:
- Common defaults: `admin`, `password`, `changeme`, `test`
- Application-specific guesses: `argus`, `parasite`, `ctf2024`
- Numeric patterns: `123456`, `admin123`

### 4. Response Interception (Man-in-the-Middle within Browser)
This is the most sophisticated vector. The script:

**Fetch API Interception:**
- Wraps the native `window.fetch` function
- When a request to `/login` is detected, it blocks the actual request
- Returns a fake successful response: `{ success: true }`

**XMLHttpRequest Interception:**
- Wraps `XMLHttpRequest.prototype.open` and `send`
- When a login request is detected, it forcibly sets:
  - HTTP status to 200
  - Response body to `{"success": true}`
- Triggers the `onload` callback to simulate a successful response

This effectively makes any login attempt succeed regardless of the actual token sent.

### 5. Login Function Override
If the application exposes a global `login()` function, the script replaces it with a version that:
- Logs the token attempt
- Immediately removes the authentication overlay
- Reveals the hidden `.shell` element
- Returns a resolved promise

### 6. Direct DOM Manipulation
The most straightforward bypass — it:
- Removes the `auth-overlay` element entirely
- Removes the `hidden` class from the `.shell` element
- Sets authentication flags in both `localStorage` and `sessionStorage`
- Manually triggers refresh functions and dispatches a fake `auth:success` event

### 7. API Endpoint Discovery
Probes common API endpoints (`/api/status`, `/api/version`, `/api/config`, etc.) looking for any that might leak tokens in their responses.

### 8. Prototype Pollution-esque Attack
If the application has a `fetchJson` utility function, the script overrides it to return mock data for specific API calls, preventing the UI from showing unauthenticated states.

## Why This Works

This attack succeeds because it exploits several common security weaknesses:

1. **Client-side authentication logic** — The application checks authentication status entirely in the browser, allowing it to be manipulated through the console
2. **No server-side validation** — The mock responses and DOM manipulation work because the server doesn't properly validate authentication on subsequent requests
3. **Exposed global functions** — Functions like `login()`, `refresh()`, and `initIndexPage()` being in the global scope makes them easy to override
4. **Predictable structure** — Consistent class names (`.shell`, `#auth-overlay`) and endpoint patterns make the application easy to target

## Security Implications

This type of attack is particularly dangerous because:
- It requires no special tools — just a browser console
- The script can be copied and pasted by anyone with basic technical knowledge
- It uses multiple fallback methods, so if one vector is patched, others may still work

> **Fundamental Fix:** Move authentication validation entirely to the server side, where client-side manipulation cannot affect it.
