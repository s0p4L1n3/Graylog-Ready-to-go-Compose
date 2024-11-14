# Graylog Project with Docker Compose behind Traefik Reverse Proxy for TLS

### Initial Step

Self-signed certificate without SAN (Subject Alternative Name) is not valid anymore. 
You will need to generate one for graylog to access it securely.

1. Generate a Private Key
```shell
openssl genrsa -out graylog.sopaline.lan_withPass.key 2048
```

2. Generate a CSR (Certificate Signing Request)
```shell
openssl req -new -key graylog.sopaline.lan_withPass.key -out graylog.sopaline.lan.csr
```
Common Name: choose `graylog.sopaline.lan`
Leave blank for challenge password.

3. Remove Passphrase from Key
```shell
openssl rsa -in graylog.sopaline.lan_withPass.key -out graylog.sopaline.lan.key
```

4. Create config file for SAN
```shell
cat <<EOF > v3.ext
subjectKeyIdentifier   = hash
authorityKeyIdentifier = keyid:always,issuer:always
basicConstraints       = CA:TRUE
keyUsage               = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment, keyAgreement, keyCertSign
subjectAltName         = DNS:graylog.sopaline.lan
issuerAltName          = issuer:copy
EOF
```


5. Generating a Self-Signed Certificate
```shell
openssl x509 -req -in graylog.sopaline.lan.csr -signkey graylog.sopaline.lan.key -out graylog.sopaline.lan.crt -days 365 -sha256 -extfile v3.ext
```

Move the two files into traefik/secrets/traefik

## Create a bridge network for graylog/traefik
```
docker network create -d bridge graylog
```

## Start the compose for traefik

Download this project and start the docker compose with the command below

```
cd traefik
docker compose up -d
```

## Start the compose for graylog and its components

```
cd graylog
docker compose up -d
```

`docker ps` to display status
```
CONTAINER ID   IMAGE                                 COMMAND                  CREATED             STATUS                       PORTS                                                                                                                                                                                                                                                                                                    NAMES
663c70777656   graylog/graylog:6.1.2                 "/usr/bin/tini -- wa…"   About an hour ago   Up About an hour (healthy)   0.0.0.0:1514->1514/udp, :::1514->1514/udp, 0.0.0.0:1514-1515->1514-1515/tcp, :::1514-1515->1514-1515/tcp, 0.0.0.0:1518->1518/udp, :::1518->1518/udp, 0.0.0.0:5044->5044/tcp, :::5044->5044/tcp, 0.0.0.0:12202->12202/tcp, 0.0.0.0:12201->12201/udp, :::12202->12202/tcp, :::12201->12201/udp, 9000/tcp   graylog-graylog-1
398e4457743d   opensearchproject/opensearch:2.15.0   "./opensearch-docker…"   About an hour ago   Up About an hour             9300/tcp, 9600/tcp, 0.0.0.0:9200->9200/tcp, :::9200->9200/tcp, 9650/tcp                                                                                                                                                                                                                                  graylog-opensearch-1
46f2df5746f4   traefik:3.2.0                        "/entrypoint.sh --pr…"   2 hours ago         Up 2 hours                   80/tcp, 0.0.0.0:443->443/tcp, :::443->443/tcp                                                                                                                                                                                                                                                            graylog-reverse-proxy-1
67eb2ff8f2b1   mongo:7.0.14                          "docker-entrypoint.s…"   20 hours ago        Up 20 hours                  27017/tcp                                                                                                                                                                                                                                                                                                graylog-mongo-1
```

You can now access your Graylog instance from: https://graylog.sopaline.lan !
Default creds: admin / admin

You can go further with Content Pack i've made for you !

## Windows Security
https://github.com/s0p4L1n3/Graylog_Content_Pack_Windows_Security
## VCEnter / ESXI v8 Security
https://github.com/s0p4L1n3/Graylog_Content_Pack_VMWare-8.X-forVCSA-ESXI

## Stormshield SNS Firewall
https://github.com/s0p4L1n3/Graylog_Content_Pack_Stormshield_Firewall



## Reference
- https://gist.github.com/KeithYeh/bb07cadd23645a6a62509b1ec8986bbc
