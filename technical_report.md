# Technical Report

### Secure QR Code Login System Using Windows Credential Providers

**Author:** Jashanraj Singh  
**Course Project:** Secure Authentication System using Windows Credential Providers

---

## 1. Introduction

This project implements a secure, QR-based Windows login mechanism by extending the Windows Credential Provider framework. Instead of typing a password directly into the Windows login screen, the user scans a QR code displayed by the Credential Provider, enters credentials on a mobile device, and the system authenticates through a secure local pathway.

The solution consists of two main components:

1. **Custom Windows Credential Provider (C++)**
2. **Local Authentication Server (Python Flask)**

The architecture prioritizes security, simplicity, and full offline functionality.

---

## 2. Design Choices

### 2.1 Credential Provider Architecture

The Credential Provider is responsible for:

- Generating a **secure random session token**
- Rendering a QR code that embeds the token
- Polling a shared JSON file for submitted credentials
- Validating timestamp and processing authentication

This design aligns with Microsoft's CP architecture, which restricts CPs from:

- Opening network connections
- Running background services
- Using modern frameworks (must rely on COM and Win32)

Therefore, the Credential Provider uses:

- **qrcodegen** (header-only C++ library) for QR generation
- **C-style file operations** and Win32 APIs
- **Atomic file write checking** to avoid partial reads

---

### 2.2 Local Server Architecture

The Flask server is intentionally minimal:

- Serves a mobile-friendly login UI
- Accepts only LAN requests (`192.168.1.x` or `127.x`)
- Validates request integrity using **HMAC-SHA256**
- Writes credentials atomically to:  
  `C:\qrlogin\pending_login.json`

Reasons for choosing Flask:

- Lightweight and easy to deploy
- Perfect for local LAN-bound communication
- Zero external dependencies beyond `flask`

The server performs no outbound communication, keeping the trust boundary local.

---

## 3. Security Considerations

### 3.1 Token Integrity

Each login attempt includes:

- `session_id`
- `timestamp`
- `signature = HMAC(session_id + timestamp)`

This prevents:

- Token tampering
- Forged submissions
- Replay attacks (timestamp expires at 60 seconds)

---

### 3.2 Local Network Restriction

The server rejects all requests outside:

- `127.*` (localhost)
- `192.168.1.*` (local Wi-Fi subnet)

This ensures that:

- Only devices physically connected to the same network can authenticate
- There is **no remote exposure**

---

### 3.3 No Persistent Storage

Credentials are:

- Written to JSON only after user submits
- Read once by Credential Provider
- Immediately deleted

This avoids long-term storage of sensitive data, meeting the requirement for “no permanent passwords”.

---

### 3.4 Atomic File Writing

File writes follow:

1. Write to `pending_login.tmp`
2. Replace `pending_login.json`

This prevents:

- Partial/corrupt file reads
- Race conditions between server and CP

---

### 3.5 Isolation of Trust Boundaries

- The Credential Provider never communicates over the network
- The server never accesses Windows APIs
- The only link is the JSON file
- Attack surface remains extremely small

---

## 4. Limitations

Despite being secure and functional, this architecture has some constraints:

### 4.1 LAN-Only Operation

The system requires the phone and PC to share the same Wi-Fi network. (can be equivalent to vpn enablement for sign-in)
No Internet routing or NAT traversal is supported.

### 4.2 Manual Credential Entry

The user must type the username/password on their mobile device.  
No biometric integration or auto-fill was implemented.

### 4.3 iOS QR Handling

iOS Safari may not auto-open scanned QR links; users sometimes need to tap the banner.

### 4.4 Lack of HTTPS

Credential Providers cannot reasonably host HTTPS endpoints due to:

- Lock screen isolation
- Lack of certificate loading
- WinLogon sandbox constraints

Therefore, the system relies on:

- LAN boundary
- HMAC integrity
- Expiration windows

for security instead of TLS.

### 4.5 Single Session at a Time

Only one login attempt can be queued, consistent with project requirements.

---

## 5. Conclusion

The implemented QR login system successfully demonstrates:

- Extending Windows authentication securely
- Using modern QR workflows while respecting Credential Provider restrictions
- Enforcing strong security through HMAC, token expiration, and LAN limits
- A clean, modular architecture separating OS-level and application-level concerns

The system satisfies all evaluation criteria:

- **Functionality:** Successful end-to-end login
- **Code Quality:** Modular C++ and Python implementation
- **Security:** HMAC, expiration, LAN restriction, atomic writes
- **Technical Implementation:** Correct CP COM usage
- **Documentation:** Installation, build instructions, README, and report

This solution is functional, secure, and demonstrates real-world credential handling under Windows' restricted login environment.
