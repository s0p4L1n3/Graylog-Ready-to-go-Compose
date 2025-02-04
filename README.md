# Graylog Project with Docker Compose behind Traefik Reverse Proxy for TLS

This use case is when you want to test Graylog localy

### Initial Step (only required if you want to generate your own certs)

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

## 1 Create a bridge networks for graylog/traefik
```
docker network create -d bridge graylog
docker network create -d bridge traefik-tls
```

## 2 Clone the traefik docker compose project, to start it up before graylog stack.
```
git clone https://github.com/s0p4L1n3/traefik-tls.git
cd traefik-tls && docker compose up -d
```

## 3 Clone the project and start the compose for graylog and its components

```
cd .. && git clone https://github.com/s0p4L1n3/Graylog-Ready-to-go-Compose.git
cd Graylog-Ready-to-go-Compose/graylog
docker compose up -d
```

`docker ps` to display status
```
CONTAINER ID   IMAGE                                 COMMAND                  CREATED             STATUS                       PORTS                                                                                                                                                                                                                                                                                                    NAMES
663c70777656   graylog/graylog:6.1.5                 "/usr/bin/tini -- wa…"   About an hour ago   Up About an hour (healthy)   0.0.0.0:1514->1514/udp, :::1514->1514/udp, 0.0.0.0:1514-1515->1514-1515/tcp, :::1514-1515->1514-1515/tcp, 0.0.0.0:1518->1518/udp, :::1518->1518/udp, 0.0.0.0:5044->5044/tcp, :::5044->5044/tcp, 0.0.0.0:12202->12202/tcp, 0.0.0.0:12201->12201/udp, :::12202->12202/tcp, :::12201->12201/udp, 9000/tcp   graylog-graylog-1
398e4457743d   opensearchproject/opensearch:2.15.0   "./opensearch-docker…"   About an hour ago   Up About an hour             9300/tcp, 9600/tcp, 0.0.0.0:9200->9200/tcp, :::9200->9200/tcp, 9650/tcp                                                                                                                                                                                                                                  graylog-opensearch-1
46f2df5746f4   traefik:3.2.0                        "/entrypoint.sh --pr…"   2 hours ago         Up 2 hours                   80/tcp, 0.0.0.0:443->443/tcp, :::443->443/tcp                                                                                                                                                                                                                                                            graylog-reverse-proxy-1
67eb2ff8f2b1   mongo:7.0.14                          "docker-entrypoint.s…"   20 hours ago        Up 20 hours                  27017/tcp                                                                                                                                                                                                                                                                                                graylog-mongo-1
```

You can now access your Graylog instance from: https://graylog.sopaline.lan !
Default creds: admin / admin

# Migration from Opensearch to Datanode

Graylog released datanode docker image which is an Opensearch 2.15 fork. It is a seamless integration and the migration process is pretty straitforward.

If you want to migrate Go to System > Data Node > Migration, follow the steps

![image](https://github.com/user-attachments/assets/9aaf2ff0-838f-4ffb-aab3-4c63a35fcc72)

In order to Graylog to detect the datanode, edit the docker-compose.yml and add the datanode service in the service code block of the yml file.
Do not forget to declare the volume along with other one

```
graylog_datanode:
    driver: local

services:

  datanode:
    image: "graylog/graylog-datanode:6.1"
    hostname: "datanode"
    environment:
      GRAYLOG_DATANODE_NODE_ID_FILE: "/var/lib/graylog-datanode/node-id"
      GRAYLOG_DATANODE_PASSWORD_SECRET: "somepasswordpepper"
      GRAYLOG_DATANODE_ROOT_PASSWORD_SHA2: "8c6976e5b5410415bde908bd4dee15dfb167a9c873fc4bb8a81f6f2ab448a918"
      GRAYLOG_DATANODE_MONGODB_URI: "mongodb://mongodb:27017/graylog"
    ulimits:
      memlock:
        hard: -1
        soft: -1
      nofile:
        soft: 65536
        hard: 65536
    networks:
      - graylog
    volumes:
      - "graylog_datanode:/var/lib/graylog-datanode"
```

And `docker compose up -d`

The datanode should be detected and Follow then the process.

At the end, when Graylog display you should remove all reference to Opensearch, replace you docker-compose.yml file with this one: https://github.com/s0p4L1n3/Graylog-Ready-to-go-Compose/blob/main/graylog/docker-compose-datanode.yml
rename it with the original file name `docker-compose.yml`.

And `docker compose up -d`

It should display like this after starting:

![image](https://github.com/user-attachments/assets/babf9352-81f1-47f6-9202-5487a9d86a79)

# Additionnal info

You can go further with Content Pack i've made for you !

## Windows Security
https://github.com/s0p4L1n3/Graylog_Content_Pack_Windows_Security
## VCEnter / ESXI v8 Security
https://github.com/s0p4L1n3/Graylog_Content_Pack_VMWare-8.X-forVCSA-ESXI

## Stormshield SNS Firewall
https://github.com/s0p4L1n3/Graylog_Content_Pack_Stormshield_Firewall



## Reference
- https://gist.github.com/KeithYeh/bb07cadd23645a6a62509b1ec8986bbc
