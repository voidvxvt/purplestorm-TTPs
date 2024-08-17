# Pivoting



#### SOCKS Tunneling / Dynamic Port Forwarding
```bash
ssh -D 1080 user@10.129.X.X -i id_rsa -fN
```
`/etc/proxychains4.conf` -> `socks5 127.0.0.1 1080`
```bash
proxychains4 -f /etc/proxychains4.conf rdp -i '172.16.5.19' -u 'user' -p 'pass'
```
-> you can only perform a `full TCP connect scan` over proxychains ??
