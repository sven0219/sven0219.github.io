---
title: Setup xpack security for ELK 8.x
tags:
  - ELK
id: '12565'
categories:
  - - skills
date: 2022-07-15 15:13:58
---

# Setup xpack security for ELK 8.x

> I use docker to run ELK,so all of the following operations are based on docker container operations,If you are running elk in other ways, note the path to execute the command.

## Generate the certificate authority

1.  Use the `elasticsearch-certutil` tool on any single node to generate a CA for your cluster.

```shell
 # Docker 
 docker exec -it elasticsearch /bin/bash
./bin/elasticsearch-certutil ca
```

a.When prompted, accept the default file name, which is `elastic-stack-ca.p12`. This file contains the public certificate for your CA and the private key used to sign certificates for each node.

b.Enter a password for your CA. You can choose to leave the password blank if you’re not deploying to a production environment.

2.  On any single node, generate a certificate and private key for the nodes in your cluster. You include the `elastic-stack-ca.p12` output file that you generated in the previous step.

```shell
# Docker 
./bin/elasticsearch-certutil cert --ca elastic-stack-ca.p12
```

a. Enter the password for your CA, or press **Enter** if you did not configure one in the previous step.

b. Create a password for the certificate and accept the default file name.The output file is a keystore named `elastic-certificates.p12`. This file contains a node certificate, node key, and CA certificate.

3.  On **every** node in your cluster, copy the `elastic-certificates.p12` file to the `$ES_PATH_CONF` directory.

## Config elasticsearch.yml

1.  add the follow to `elasticsearch.yml`

```yaml
xpack.security.transport.ssl.enabled: true
xpack.security.transport.ssl.verification_mode: certificate 
xpack.security.transport.ssl.client_authentication: required
xpack.security.transport.ssl.keystore.path: elastic-certificates.p12
xpack.security.transport.ssl.truststore.path: elastic-certificates.p12
```

2.  If you entered a password when creating the node certificate, run the following commands to store the password in the Elasticsearch keystore:

```shell
./bin/elasticsearch-keystore add xpack.security.transport.ssl.keystore.secure_password
./bin/elasticsearch-keystore add xpack.security.transport.ssl.truststore.secure_password
```

3.  Complete the previous steps for each node in your cluster.
    
4.  On **every** node in your cluster, start Elasticsearch.
    

```shell
docker restart elasticsearch
```

## Generate password

Generate password and copy it

```bash
# Docker 
./bin/elasticsearch-setup-password auto
```

## Config kibana

add the follow to kibana.yml

```yaml
elasticsearch.username: [MY_ELASTIC_USERNAME]
elasticsearch.passowrd: [MY_ELASTIC_PASSWORD]
```