The following Caddy v2 custom binaries were built on an Intel Core i7 4790K Haswell based server using go 1.14.2 with [xcaddy](https://github.com/caddyserver/xcaddy) builder tool based on guide at https://blog.jitdor.com/2020/05/06/get-caddy-2-0-now-with-cloudflare-dns-provider-module-for-tls/ on CentOS 7.8 system with the [Cloudfalre DNS provider](https://github.com/caddy-dns) added.

* caddy is caddy 2.0.0 GA release binary from https://github.com/caddyserver/caddy/releases
* caddy-gcc8 is caddy v2 binary built with GCC 8.3.1 compiler
* caddy-gcc9 is caddy v2 binary built with GCC 9.1.1 compiler

```
mkdir -p /var/log/caddy /usr/share/caddy /etc/caddy
wget -4 -O /usr/share/caddy/index.html https://github.com/centminmod/centminmod-caddy-v2/raw/master/xcaddy-binaries/welcome.html
cp -a /usr/share/caddy/index.html /usr/share/caddy/caddy-index.html
wget -4 -O /etc/caddy/Caddyfile https://github.com/centminmod/centminmod-caddy-v2/raw/master/xcaddy-binaries/Caddyfile

wget -4 https://github.com/centminmod/centminmod-caddy-v2/raw/master/xcaddy-binaries/caddy-gcc9.zip
# or
# wget -4 https://github.com/centminmod/centminmod-caddy-v2/raw/master/xcaddy-binaries/caddy.zip

wget -4 -O /usr/lib/systemd/system/caddy.service https://github.com/centminmod/centminmod-caddy-v2/raw/master/xcaddy-binaries/caddy.service
groupadd --system caddy
useradd --system  --gid caddy  --create-home  --home-dir /var/lib/caddy  --shell /usr/sbin/nologin  --comment "Caddy web server"  caddy
chown -R caddy:caddy /var/log/caddy /usr/share/caddy

unzip -o caddy-gcc9.zip -d /usr/bin
rm -f caddy-gcc9.zip
# or
# unzip -o caddy.zip -d /usr/bin
# rm -f caddy.zip

if [ -f /usr/bin/caddy ]; then cp -a /usr/bin/caddy /usr/bin/caddy-orig; fi
mv -f /usr/bin/caddy-gcc9 /usr/bin/caddy
chmod +x /usr/bin/caddy
caddy trust
sed -i 's|^:80|:81\n\nheader x-powered-by "caddy centminmod"\nencode gzip\n|' /etc/caddy/Caddyfile
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

# caddygcc service

For `caddy-gcc9` binary run as `caddygcc` service if you intend to run it and do comparison testing beside caddy binary as `caddy` service to compare their performance. Where you end up only starting one of the services to and stopping the other.

If intending to use `caddy-gcc9` beside caddy official binary setup command shortcuts

```
echo "service caddygcc stop" >/usr/bin/caddygccstop ; chmod 700 /usr/bin/caddygccstop
echo "service caddygcc start" >/usr/bin/caddygccstart ; chmod 700 /usr/bin/caddygccstart
echo "service caddygcc status" >/usr/bin/caddygccstatus ; chmod 700 /usr/bin/caddygccstatus
echo "service caddygcc restart" >/usr/bin/caddygccrestart ; chmod 700 /usr/bin/caddygccrestart
echo "service caddygcc reload" >/usr/bin/caddygccreload ; chmod 700 /usr/bin/caddygccreload
````

To test `caddy-gcc9` binary would be

```
caddystop; caddygccstart
```

To test `caddy` binary would be

```
caddygccstop; caddystart
```

```
diff -u <(caddy list-modules) <(caddy-gcc9 list-modules)
--- /dev/fd/63  2020-05-11 11:35:30.685585903 +0000
+++ /dev/fd/62  2020-05-11 11:35:30.686585909 +0000
@@ -14,6 +14,7 @@
 caddy.logging.writers.stderr
 caddy.logging.writers.stdout
 caddy.storage.file_system
+dns.providers.cloudflare
 http
 http.authentication.hashes.bcrypt
 http.authentication.hashes.scrypt
```

`/usr/lib/systemd/system/caddygcc.service` which references the `caddy-gcc9` binary

```
[Unit]
Description=Caddy GCC 9
Documentation=https://caddyserver.com/docs/
After=network.target

[Service]
User=caddy
Group=caddy
ExecStart=/usr/bin/caddy-gcc9 run --environ --config /etc/caddy/Caddyfile
ExecReload=/usr/bin/caddy-gcc9 reload --config /etc/caddy/Caddyfile
TimeoutStopSec=5s
LimitNOFILE=1048576
LimitNPROC=512
PrivateTmp=true
ProtectSystem=full
AmbientCapabilities=CAP_NET_BIND_SERVICE

[Install]
WantedBy=multi-user.target
```
and also have `caddy` service

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

h2load HTTP/2 HTTPS test `caddy` binary and stop nginx + stop `caddy-gcc9` binary service on Intel Core i7 4790K Haswell based server

```
ngxstop; caddygccstop; caddystop; caddystart; sleep 30; h2load -t1 -c50 -D 20 --warm-up-time=5 -m50 -H "Accept-Encoding:gzip" https://caddy2.domain.com:4444/caddy-index.html
```

1st run

```
ngxstop; caddygccstop; caddystop; caddystart; sleep 30; h2load -t1 -c50 -D 20 --warm-up-time=5 -m50 -H "Accept-Encoding:gzip" https://caddy2.domain.com:4444/caddy-index.html
Stopping nginx (via systemctl):                            [  OK  ]
Redirecting to /bin/systemctl stop caddygcc.service
Redirecting to /bin/systemctl stop caddy.service
Redirecting to /bin/systemctl start caddy.service
starting benchmark...
spawning thread #0: 50 total client(s). Timing-based test with 5s of warm-up time and 20s of main duration for measurements.
Warm-up started for thread #0.
progress: 10% of clients started
progress: 20% of clients started
progress: 30% of clients started
progress: 40% of clients started
progress: 50% of clients started
progress: 60% of clients started
progress: 70% of clients started
progress: 80% of clients started
progress: 90% of clients started
progress: 100% of clients started
TLS Protocol: TLSv1.2
Cipher: ECDHE-ECDSA-AES256-GCM-SHA384
Server Temp Key: ECDH P-256 256 bits
Application protocol: h2
Warm-up phase is over for thread #0.
Main benchmark duration is started for thread #0.
Main benchmark duration is over for thread #0. Stopping all clients.
Stopped all clients for thread #0

finished in 25.01s, 10928.55 req/s, 55.01MB/s
requests: 218571 total, 218571 started, 218571 done, 218571 succeeded, 0 failed, 0 errored, 0 timeout
status codes: 218868 2xx, 0 3xx, 0 4xx, 0 5xx
traffic: 1.07GB (1153654428) total, 2.84MB (2982880) headers (space savings 95.91%), 1.30GB (1399851624) data
                     min         max         mean         sd        +/- sd
time for request:     3.78ms       1.40s    223.93ms     97.00ms    75.66%
time for connect:        0us         0us         0us         0us     0.00%
time to 1st byte:        0us         0us         0us         0us     0.00%
req/s           :     202.43      244.38      223.00        9.60    73.47%
```

2nd run

```
ngxstop; caddygccstop; caddystop; caddystart; sleep 30; h2load -t1 -c50 -D 20 --warm-up-time=5 -m50 -H "Accept-Encoding:gzip" https://caddy2.domain.com:4444/caddy-index.html   
Stopping nginx (via systemctl):                            [  OK  ]
Redirecting to /bin/systemctl stop caddygcc.service
Redirecting to /bin/systemctl stop caddy.service
Redirecting to /bin/systemctl start caddy.service
starting benchmark...
spawning thread #0: 50 total client(s). Timing-based test with 5s of warm-up time and 20s of main duration for measurements.
Warm-up started for thread #0.
progress: 10% of clients started
progress: 20% of clients started
progress: 30% of clients started
progress: 40% of clients started
progress: 50% of clients started
progress: 60% of clients started
progress: 70% of clients started
progress: 80% of clients started
progress: 90% of clients started
progress: 100% of clients started
TLS Protocol: TLSv1.2
Cipher: ECDHE-ECDSA-AES256-GCM-SHA384
Server Temp Key: ECDH P-256 256 bits
Application protocol: h2
Warm-up phase is over for thread #0.
Main benchmark duration is started for thread #0.
Main benchmark duration is over for thread #0. Stopping all clients.
Stopped all clients for thread #0

finished in 25.01s, 11004.50 req/s, 55.43MB/s
requests: 220090 total, 220090 started, 220090 done, 220090 succeeded, 0 failed, 0 errored, 0 timeout
status codes: 220142 2xx, 0 3xx, 0 4xx, 0 5xx
traffic: 1.08GB (1162356910) total, 2.87MB (3014168) headers (space savings 95.91%), 1.32GB (1416370808) data
                     min         max         mean         sd        +/- sd
time for request:     3.38ms    950.88ms    227.05ms     93.79ms    75.35%
time for connect:        0us         0us         0us         0us     0.00%
time to 1st byte:        0us         0us         0us         0us     0.00%
req/s           :     199.81      243.34      220.06       10.22    72.00%
```

h2load HTTP/2 HTTPS test `caddy-gcc9` binary and stop nginx + stop `caddy` binary service on Intel Core i7 4790K Haswell based server

```
ngxstop; caddystop; caddygccstop; caddygccstart; sleep 30; h2load -t1 -c50 -D 20 --warm-up-time=5 -m50 -H "Accept-Encoding:gzip" https://caddy2.domain.com:4444/caddy-index.html
```

1st run

```
ngxstop; caddystop; caddygccstop; caddygccstart; sleep 30; h2load -t1 -c50 -D 20 --warm-up-time=5 -m50 -H "Accept-Encoding:gzip" https://caddy2.domain.com:4444/caddy-index.html
Stopping nginx (via systemctl):                            [  OK  ]
Redirecting to /bin/systemctl stop caddy.service
Redirecting to /bin/systemctl stop caddygcc.service
Redirecting to /bin/systemctl start caddygcc.service
starting benchmark...
spawning thread #0: 50 total client(s). Timing-based test with 5s of warm-up time and 20s of main duration for measurements.
Warm-up started for thread #0.
progress: 10% of clients started
progress: 20% of clients started
progress: 30% of clients started
progress: 40% of clients started
progress: 50% of clients started
progress: 60% of clients started
progress: 70% of clients started
progress: 80% of clients started
progress: 90% of clients started
progress: 100% of clients started
TLS Protocol: TLSv1.2
Cipher: ECDHE-ECDSA-AES256-GCM-SHA384
Server Temp Key: ECDH P-256 256 bits
Application protocol: h2
Warm-up phase is over for thread #0.
Main benchmark duration is started for thread #0.
Main benchmark duration is over for thread #0. Stopping all clients.
Stopped all clients for thread #0

finished in 25.01s, 11065.35 req/s, 55.71MB/s
requests: 221307 total, 221307 started, 221307 done, 221307 succeeded, 0 failed, 0 errored, 0 timeout
status codes: 221326 2xx, 0 3xx, 0 4xx, 0 5xx
traffic: 1.09GB (1168381725) total, 2.88MB (3015481) headers (space savings 95.91%), 1.32GB (1417116656) data
                     min         max         mean         sd        +/- sd
time for request:     3.76ms       5.79s    232.39ms    212.36ms    96.94%
time for connect:        0us         0us         0us         0us     0.00%
time to 1st byte:        0us         0us         0us         0us     0.00%
req/s           :     192.54      239.41      221.28        8.41    72.00%
```

2nd run

```
ngxstop; caddystop; caddygccstop; caddygccstart; sleep 30; h2load -t1 -c50 -D 20 --warm-up-time=5 -m50 -H "Accept-Encoding:gzip" https://caddy2.domain.com:4444/caddy-index.html
Stopping nginx (via systemctl):                            [  OK  ]
Redirecting to /bin/systemctl stop caddy.service
Redirecting to /bin/systemctl stop caddygcc.service
Redirecting to /bin/systemctl start caddygcc.service
starting benchmark...
spawning thread #0: 50 total client(s). Timing-based test with 5s of warm-up time and 20s of main duration for measurements.
Warm-up started for thread #0.
progress: 10% of clients started
progress: 20% of clients started
progress: 30% of clients started
progress: 40% of clients started
progress: 50% of clients started
progress: 60% of clients started
progress: 70% of clients started
progress: 80% of clients started
progress: 90% of clients started
progress: 100% of clients started
TLS Protocol: TLSv1.2
Cipher: ECDHE-ECDSA-AES256-GCM-SHA384
Server Temp Key: ECDH P-256 256 bits
Application protocol: h2
Warm-up phase is over for thread #0.
Main benchmark duration is started for thread #0.
Main benchmark duration is over for thread #0. Stopping all clients.
Stopped all clients for thread #0

finished in 25.01s, 11105.60 req/s, 55.90MB/s
requests: 222112 total, 222112 started, 222112 done, 222112 succeeded, 0 failed, 0 errored, 0 timeout
status codes: 222036 2xx, 0 3xx, 0 4xx, 0 5xx
traffic: 1.09GB (1172377416) total, 2.89MB (3029849) headers (space savings 95.91%), 1.33GB (1424265240) data
                     min         max         mean         sd        +/- sd
time for request:     3.64ms       1.26s    220.95ms     92.35ms    75.18%
time for connect:        0us         0us         0us         0us     0.00%
time to 1st byte:        0us         0us         0us         0us     0.00%
req/s           :     205.36      248.13      226.62        9.51    69.39%
```

**Summary:** Seems there's only 0.9 to 1.25% difference between `caddy-gcc9` vs `caddy` binary.

And quick compare to Centmin Mod Nginx 1.17.10 on same Intel Core i7 4790K Haswell based server

```
caddystop; caddygccstop; ngxrestart; sleep 30; h2load -t1 -c50 -D 20 --warm-up-time=5 -m50 -H "Accept-Encoding:gzip" https://ngx2.domain.com/caddy-index.html
```

1st run for Nginx

```
caddystop; caddygccstop; ngxrestart; sleep 30; h2load -t1 -c50 -D 20 --warm-up-time=5 -m50 -H "Accept-Encoding:gzip" https://ngx2.domain.com/caddy-index.html
Redirecting to /bin/systemctl stop caddy.service
Redirecting to /bin/systemctl stop caddygcc.service
Restarting nginx (via systemctl):                          [  OK  ]
starting benchmark...
spawning thread #0: 50 total client(s). Timing-based test with 5s of warm-up time and 20s of main duration for measurements.
Warm-up started for thread #0.
progress: 10% of clients started
progress: 20% of clients started
progress: 30% of clients started
progress: 40% of clients started
progress: 50% of clients started
progress: 60% of clients started
progress: 70% of clients started
progress: 80% of clients started
progress: 90% of clients started
progress: 100% of clients started
TLS Protocol: TLSv1.2
Cipher: ECDHE-ECDSA-AES128-GCM-SHA256
Server Temp Key: ECDH P-256 256 bits
Application protocol: h2
Warm-up phase is over for thread #0.
Main benchmark duration is started for thread #0.
Main benchmark duration is over for thread #0. Stopping all clients.
Stopped all clients for thread #0

finished in 25.01s, 23102.85 req/s, 113.09MB/s
requests: 462057 total, 462057 started, 462057 done, 462057 succeeded, 0 failed, 0 errored, 0 timeout
status codes: 462057 2xx, 0 3xx, 0 4xx, 0 5xx
traffic: 2.21GB (2371761581) total, 6.09MB (6390797) headers (space savings 96.09%), 2.75GB (2947951982) data
                     min         max         mean         sd        +/- sd
time for request:     4.33ms    623.48ms    108.06ms     61.57ms    76.38%
time for connect:        0us         0us         0us         0us     0.00%
time to 1st byte:        0us         0us         0us         0us     0.00%
req/s           :     320.15     1033.84      462.01      216.06    88.00%
```

2nd run for Nginx

```
caddystop; caddygccstop; ngxrestart; sleep 30; h2load -t1 -c50 -D 20 --warm-up-time=5 -m50 -H "Accept-Encoding:gzip" https://ngx2.domain.com/caddy-index.html
Redirecting to /bin/systemctl stop caddy.service
Redirecting to /bin/systemctl stop caddygcc.service
Restarting nginx (via systemctl):                          [  OK  ]
starting benchmark...
spawning thread #0: 50 total client(s). Timing-based test with 5s of warm-up time and 20s of main duration for measurements.
Warm-up started for thread #0.
progress: 10% of clients started
progress: 20% of clients started
progress: 30% of clients started
progress: 40% of clients started
progress: 50% of clients started
progress: 60% of clients started
progress: 70% of clients started
progress: 80% of clients started
progress: 90% of clients started
progress: 100% of clients started
TLS Protocol: TLSv1.2
Cipher: ECDHE-ECDSA-AES128-GCM-SHA256
Server Temp Key: ECDH P-256 256 bits
Application protocol: h2
Warm-up phase is over for thread #0.
Main benchmark duration is started for thread #0.
Main benchmark duration is over for thread #0. Stopping all clients.
Stopped all clients for thread #0

finished in 25.01s, 23135.30 req/s, 113.25MB/s
requests: 462706 total, 462706 started, 462706 done, 462706 succeeded, 0 failed, 0 errored, 0 timeout
status codes: 462705 2xx, 0 3xx, 0 4xx, 0 5xx
traffic: 2.21GB (2375088791) total, 6.10MB (6399806) headers (space savings 96.09%), 2.75GB (2952133184) data
                     min         max         mean         sd        +/- sd
time for request:     2.89ms    494.52ms    107.78ms     74.84ms    62.32%
time for connect:        0us         0us         0us         0us     0.00%
time to 1st byte:        0us         0us         0us         0us     0.00%
req/s           :     227.36     1438.22      462.66      306.72    92.00%
```