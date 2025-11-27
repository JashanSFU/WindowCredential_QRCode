## Credential Provider Build Instructions

1. Open SampleCredentialProvider.sln in Visual Studio 2022 (Insiders).
2. Select "x64" and "Release" configuration.
3. Build the solution (Ctrl+Shift+B).
4. Output DLL will be in:
   ./x64/Release/SampleCredentialProvider.dll

## Installation

Run the provided install.bat as Administrator:

install.bat

This will:

- Copy DLL to C:\Windows\System32
- Register CLSID in registry
- Enable CredentialProvider for logon

## Uninstall

Run uninstall.bat to remove registry keys and DLL.
