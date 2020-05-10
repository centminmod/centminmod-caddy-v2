The following Caddy v2 custom binaries were built using go 1.14.2 with [xcaddy](https://github.com/caddyserver/xcaddy) builder tool based on guide at https://blog.jitdor.com/2020/05/06/get-caddy-2-0-now-with-cloudflare-dns-provider-module-for-tls/ on CentOS 7.8 system with the [Cloudfalre DNS provider](https://github.com/caddy-dns) added.

* caddy is caddy 2.0.0 GA release binary from https://github.com/caddyserver/caddy/releases
* caddy-gcc8 is caddy v2 binary built with GCC 8.3.1 compiler
* caddy-gcc9 is caddy v2 binary built with GCC 9.1.1 compiler

```
mkdir -p /var/log/caddy /usr/share/caddy /etc/caddy
wget -4 -O /usr/share/caddy/index.html https://github.com/centminmod/centminmod-caddy-v2/raw/master/xcaddy-binaries/welcome.html
wget -4 -O /etc/caddy/Caddyfile https://github.com/centminmod/centminmod-caddy-v2/raw/master/xcaddy-binaries/Caddyfile
wget -4 https://github.com/centminmod/centminmod-caddy-v2/raw/master/xcaddy-binaries/caddy-gcc9.zip
wget -4 -O /usr/lib/systemd/system/caddy.service https://github.com/centminmod/centminmod-caddy-v2/raw/master/xcaddy-binaries/caddy.service
chown -R caddy:caddy /var/log/caddy /usr/share/caddy
unzip caddy-gcc9.zip -d /usr/bin
rm -f caddy-gcc9.zip
if [ -f /usr/bin/caddy ]; then cp -a /usr/bin/caddy /usr/bin/caddy-orig; fi
mv -f /usr/bin/caddy-gcc9 /usr/bin/caddy
chmod +x /usr/bin/caddy
caddy trust
sed -i 's|:80|:81\n\nheader x-powered-by "caddy centminmod"\nnencode gzip\n|' /etc/caddy/Caddyfile
service caddy start
service caddy status
chkconfig caddy on
journalctl -u caddy --no-pager

echo "service caddy stop" >/usr/bin/caddystop ; chmod 700 /usr/bin/caddystop
echo "service caddy start" >/usr/bin/caddystart ; chmod 700 /usr/bin/caddystart
echo "service caddy status" >/usr/bin/caddystatus ; chmod 700 /usr/bin/caddystatus
echo "service caddy restart" >/usr/bin/caddyrestart ; chmod 700 /usr/bin/caddyrestart
echo "service caddy reload" >/usr/bin/caddyreload ; chmod 700 /usr/bin/caddyreload
```

```
./caddy version
v2.0.0 h1:pQSaIJGFluFvu8KDGDODV8u4/QRED/OPyIR+MWYYse8=

v2.0.0 h1:pQSaIJGFluFvu8KDGDODV8u4/QRED/OPyIR+MWYYse8=

./caddy-gcc9 version
v2.0.0 h1:pQSaIJGFluFvu8KDGDODV8u4/QRED/OPyIR+MWYYse8=
```

```diff
 diff -u <(./caddy list-modules) <(./caddy-gcc9 list-modules)  
--- /dev/fd/63  2020-05-10 13:57:42.972146109 +0000
+++ /dev/fd/62  2020-05-10 13:57:42.972146109 +0000
@@ -14,6 +14,7 @@
 caddy.logging.writers.stderr
 caddy.logging.writers.stdout
 caddy.storage.file_system
+dns.providers.cloudflare
 http
 http.authentication.hashes.bcrypt
 http.authentication.hashes.scrypt
```

```
./caddy-gcc9 list-modules
admin.api.load
caddy.adapters.caddyfile
caddy.listeners.tls
caddy.logging.encoders.console
caddy.logging.encoders.filter
caddy.logging.encoders.filter.delete
caddy.logging.encoders.filter.ip_mask
caddy.logging.encoders.json
caddy.logging.encoders.logfmt
caddy.logging.encoders.single_field
caddy.logging.writers.discard
caddy.logging.writers.file
caddy.logging.writers.net
caddy.logging.writers.stderr
caddy.logging.writers.stdout
caddy.storage.file_system
dns.providers.cloudflare
http
http.authentication.hashes.bcrypt
http.authentication.hashes.scrypt
http.authentication.providers.http_basic
http.encoders.gzip
http.encoders.zstd
http.handlers.authentication
http.handlers.encode
http.handlers.error
http.handlers.file_server
http.handlers.headers
http.handlers.request_body
http.handlers.reverse_proxy
http.handlers.rewrite
http.handlers.static_response
http.handlers.subroute
http.handlers.templates
http.handlers.vars
http.matchers.expression
http.matchers.file
http.matchers.header
http.matchers.header_regexp
http.matchers.host
http.matchers.method
http.matchers.not
http.matchers.path
http.matchers.path_regexp
http.matchers.protocol
http.matchers.query
http.matchers.remote_ip
http.matchers.vars
http.matchers.vars_regexp
http.reverse_proxy.selection_policies.first
http.reverse_proxy.selection_policies.header
http.reverse_proxy.selection_policies.ip_hash
http.reverse_proxy.selection_policies.least_conn
http.reverse_proxy.selection_policies.random
http.reverse_proxy.selection_policies.random_choose
http.reverse_proxy.selection_policies.round_robin
http.reverse_proxy.selection_policies.uri_hash
http.reverse_proxy.transport.fastcgi
http.reverse_proxy.transport.http
pki
tls
tls.certificates.automate
tls.certificates.load_files
tls.certificates.load_folders
tls.certificates.load_pem
tls.handshake_match.sni
tls.issuance.acme
tls.issuance.internal
tls.stek.distributed
tls.stek.standard
```

`/usr/lib/systemd/system/caddy.service`

```
[Unit]
Description=Caddy
Documentation=https://caddyserver.com/docs/
After=network.target

[Service]
User=caddy
Group=caddy
ExecStart=/usr/bin/caddy run --environ --config /etc/caddy/Caddyfile
ExecReload=/usr/bin/caddy reload --config /etc/caddy/Caddyfile
TimeoutStopSec=5s
LimitNOFILE=1048576
LimitNPROC=512
PrivateTmp=true
ProtectSystem=full
AmbientCapabilities=CAP_NET_BIND_SERVICE

[Install]
WantedBy=multi-user.target
```