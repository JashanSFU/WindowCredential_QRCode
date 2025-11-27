# Windows QR Code Login – Secure Credential Provider + Local Auth Server

**Author:** Jashanraj Singh  
**Course Project:** Secure Authentication System using Windows Credential Providers

---

## Overview

This project implements a **secure QR-based Windows login system** using:

- A **custom Windows Credential Provider (C++)**
- A **local authentication server (Python Flask)**

The workflow:

1. Windows lock screen displays a QR code.
2. User scans QR with a phone.
3. A mobile-friendly page opens on the LAN.
4. User enters username/password.
5. Server transfers credentials securely to the Provider.
6. Provider logs the user into Windows.

This system demonstrates:

- Windows Credential Provider COM integration
- Secure token-based authentication
- LAN-only access to prevent unauthorized use
- Atomic credential passing
- Clean, modular C++ and Python design

---

## Features

### Credential Provider

- Displays a QR login tile
- Generates a unique 128-bit session token
- Renders QR code using `qrcodegen`
- Polls a secure JSON file for submitted credentials
- Validates expiration (≤ 60 seconds)
- Deletes sensitive data immediately
- Logs in using the Windows LogonUI pipeline

### Local Authentication Server

- Runs on `0.0.0.0:5000`
- Serves mobile-friendly login UI
- Only accepts LAN clients (`192.168.1.x` / localhost)
- Validates HMAC signature
- Checks timestamp expiration
- Writes credentials atomically (`.tmp` → final)
- Never stores passwords long-term

---

## Security Model

### Session Token

- Random 32-character hexadecimal value
- Encoded in QR
- Valid for a single login attempt

### Secure LAN Communication

- Server only accepts requests from LAN
- HMAC verifies request integrity
- 60-second expiration prevents replay attacks

### Temporary Credential File

Stored at:
C:\qrlogin\pending_login.json

File is:

- Written atomically
- Deleted after Credential Provider processes it

---

## Architecture

flowchart TD
    A[Windows Lock Screen] --> B[Credential Provider creates session_id]
    B --> C[QR Code displayed with LAN URL]
    C --> D[User scans QR on phone]
    D --> E[Phone loads mobile.html from server]
    E --> F[User submits username and password]
    F --> G[Server validates and writes JSON]
    G --> H[Credential Provider reads JSON and logs in]
    H --> I[Server deletes JSON]

---

## Directory Structure

Submission/
│── build_instructions.md
│── install.bat
│
├── CredentialProviderSecure/
│   ├── README.md
│   ├── cpp/
│   │   ├── *.cpp / *.h      (Credential Provider source files)
│   │   ├── qrcodegen/       (QR code generation library)
│   │   ├── register.reg
│   │   ├── unregister.reg
│   │   └── x64/Debug/SampleV2CredentialProvider.dll
│   │
│
└── qrlogin-server/
    ├── server.py
    └── secure_server.py
    
---

## Build Instructions (Visual Studio)

### Requirements

- Visual Studio 2022
- Windows 10+ SDK
- Desktop C++ Development workload

### Steps

1. Open:
   CredentialProviderSecure/cpp/SampleV2CredentialProvider.sln
2. Select:
   x64 → Debug
3. Build solution:
   Build → Build Solution
4. DLL output:
   cpp/x64/Debug/SampleV2CredentialProvider.dll

---

## Installation (Run in Administrator Terminal)

### Step 1 — Copy DLL

copy ".\CredentialProviderSecure\cpp\x64\Debug\SampleV2CredentialProvider.dll" C:\Windows\System32\

### Step 2 — Register Provider

reg import ".\CredentialProviderSecure\cpp\register.reg"

### Step 3 — Lock/Reboot

Press **Win + L** to see QR tile.

---

## Uninstall

reg import ".\CredentialProviderSecure\cpp\Unregister.reg"
del C:\Windows\System32\SampleV2CredentialProvider.dll

Reboot recommended.

---

## Running the QR Login Server

### Install Dependencies

pip install flask

### Run Server

python secure_server.py

Server becomes available at:

http://<PC_IP>:5000

Scan the QR code from your phone.

---

## Authentication Flow

1. Provider generates session ID
2. QR code embeds:
   http://<PC_IP>:5000/session?id=<session_id>

3. Mobile device loads UI
4. User enters credentials
5. HMAC generated on device
6. JSON is POSTed to `/unlock`
7. Server validates signature + expiration
8. Credentials written atomically
9. Provider reads file, logs in, deletes file

---

## Security Considerations

- **LAN-only** — prevents outside access
- **HMAC signatures** — prevent tampering
- **Expiration** — prevents replay
- **No persistent password storage**
- **Atomic file writes** ensure safe data handoff

This implementation meets all security requirements:
✔ Token validation  
✔ LAN boundary  
✔ Timeout enforcement  
✔ Credential protection

---

## Known Limitations

- Works only on same LAN
- User must type credentials
- 60-second token lifespan
- iOS QR requires tapping to open
- HTTPS not available inside Credential Providers

---

## Demo Checklist

Include screenshots or video of:

- Windows lock screen with QR tile
- Mobile login page
- Server console logs
- Successful Windows login

---

## Technical Report Notes

Suggested topics for your PDF:

- Use of `qrcodegen`
- Credential Provider COM lifecycle
- How LogonUI isolates processes
- Why HMAC + LAN is secure
- Why atomic writes are required
- How expiration + deletion mitigates replay


