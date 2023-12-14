# Graylog Project with Docker Compose behind Traefik Reverse Proxy for TLS



### Initial Step

Self-signed certificate without SAN (Subject Alternative Name) is not valid anymore. 
You will need to generate one for graylog to access it securely.

1. Generate a Private Key
```shell
openssl genrsa -out graylog.lab.lan_withPass.key 2048
```

2. Generate a CSR (Certificate Signing Request)
```shell
openssl req -new -key graylog.lab.lan_withPass.key -out graylog.lab.lan.csr
```
Common Name: choose `graylog.lab.lan`
Leave blank for challenge password.

3. Remove Passphrase from Key
```shell
openssl rsa -in graylog.lab.lan_withPass.key -out graylog.lab.lan.key && rm -rf graylog.lab.lan_withPass.key
```

4. Create config file for SAN
```shell
cat <<EOF > v3.ext
subjectKeyIdentifier   = hash
authorityKeyIdentifier = keyid:always,issuer:always
basicConstraints       = CA:TRUE
keyUsage               = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment, keyAgreement, keyCertSign
subjectAltName         = DNS:graylog.lab.lan
issuerAltName          = issuer:copy
EOF
```


5. Generating a Self-Signed Certificate
```shell
openssl x509 -req -in graylog.lab.lan.csr -signkey graylog.lab.lan.key -out graylog.lab.lan.crt -days 365 -sha256 -extfile v3.ext && rm -rf v3.ext graylog.lab.lan.csr
```


Move the two files into secrets/

## Start the compose

```
docker compose up -d
```

`docker ps` to display status
```
CONTAINER ID   IMAGE                                 COMMAND                  CREATED       STATUS                PORTS               NAMES
077ed8773efd   traefik:2.10.7                        "/entrypoint.sh --pr…"   3 days ago    Up 3 days             80/tcp, 0.0.0.0:443->443/tcp, :::443->443/tcp     graylog-reverse-proxy-1
3d1a08399205   graylog/graylog:5.2.2                 "/usr/bin/tini -- wa…"   3 days ago    Up 2 days (healthy)   0.0.0.0:1514-1515->1514-1515/tcp, :::1514-1515->1514-1515/tcp, 0.0.0.0:1514->1514/udp, :::1514->1514/udp, 0.0.0.0:5044->5044/tcp, :::5044->5044/tcp, 9000/tcp graylog-graylog-1
c055f448a399   opensearchproject/opensearch:2.11.0   "./opensearch-docker…"   2 weeks ago   Up 2 weeks            9300/tcp, 9600/tcp, 0.0.0.0:9200->9200/tcp, :::9200->9200/tcp, 9650/tcp  graylog-opensearch-1
162fa8f6b746   mongo:6.0.11                          "docker-entrypoint.s…"   2 weeks ago   Up 2 weeks            27017/tcp        graylog-mongo-1
```












## Reference
- https://gist.github.com/KeithYeh/bb07cadd23645a6a62509b1ec8986bbc
