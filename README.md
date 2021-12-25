# OSCP Notes
***
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
```
powershell -command wget -o <file_name> <full_url_path>
powershell -command curl -o <file_name> <full_url_path>
powershell -command Invoke-WebRequest <url> -O <file.jpg>
```

---
