The following Caddy v2 custom binaries were built using go 1.14.2 with [xcaddy](https://github.com/caddyserver/xcaddy) builder tool based on guide at https://blog.jitdor.com/2020/05/06/get-caddy-2-0-now-with-cloudflare-dns-provider-module-for-tls/ on CentOS 7.8 system with the [Cloudfalre DNS provider](https://github.com/caddy-dns) added.

* caddy is caddy 2.0.0 GA release binary from https://github.com/caddyserver/caddy/releases
* caddy-gcc8 is caddy v2 binary built with GCC 8.3.1 compiler
* caddy-gcc9 is caddy v2 binary built with GCC 9.1.1 compiler

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