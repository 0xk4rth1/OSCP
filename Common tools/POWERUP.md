
**PowerUp** (part of the PowerSploit framework) enumerates a wide range of **Windows privilege escalation vulnerabilities** caused by misconfigurations. It scans the system for common flaws that allow a low-privilege user to elevate to local Administrator or `SYSTEM` status


What PowerUp Enumerates

1. Service Misconfigurations

- **Unquoted Service Paths**: Finds services with spaces in their paths that lack quotation marks (like the flaw discussed earlier).
- **Modifiable Service Executables**: Identifies service binaries that a low-privilege user can overwrite or modify.
- **Modifiable Service Registries**: Locates registry paths (e.g., under `HKLM\SYSTEM\CurrentControlSet\Services`) where permissions allow non-admins to change the `ImagePath`.
- **Permission issues (`Get-ServiceDetail`)**: Scans for weak service permissions that allow users to restart or reconfigure services.

2. Registry & Policy Settings

- **AlwaysInstallElevated**: Checks if this policy is enabled in `HKCU` or `HKLM`, which allows regular users to install `.msi` files with full system privileges. 
- **AutoLogon Credentials**: Extracts plaintext passwords stored in the registry for automatic user logins.
- **Modifiable Registry Autoruns**: Identifies applications that launch automatically at startup whose executable files or registry paths can be modified by low-privilege users.

3. Password & Credential Hunting

- **Unattend/Sysprep Files**: Scans the hard drive for leftover deployment files (like `unattend.xml` or `sysprep.inf`) that might contain plaintext local administrator credentials.
- **Encrypted GPO Passwords**: Searches for `Groups.xml` or other Group Policy Preference files containing encrypted (`cPassword`) strings, which can be easily decrypted using known keys.

4. System & Driver Vulnerabilities

- **Vulnerable Drivers / Software**: Identifies outdated or known vulnerable third-party services running on the machine.
- **DLL Hijacking Opportunities**: Searches for missing DLL paths within running processes or services that a user could plant a malicious DLL into.

---

**Syntax:**

`Import-Module .\PowerUp.ps1`             (powershell)

`Invoke-AllChecks`           (The primary command. It runs every enumeration check simultaneously and outputs a detailed risk report.)

`Get-UnquotedService`     (Specifically hunts down unquoted service paths.)

`Get-ModifiableServiceFile`   (Specifically checks for service binaries with weak file permissions.)

If we encounter any problem while executing scripts

`Set-ExecutionPolicy Bypass -Scope Process`