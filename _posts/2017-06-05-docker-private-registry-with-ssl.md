---
layout: post
title: Build a Docker Private Registry with Self-Signed SSL
categories:
header_image: /img/boat2.jpg
header_permalink: http://www.metmuseum.org/art/collection/search/12611
---

# {{ page.title }}

Recently I've been getting back into Kubernetes, which, for the time being, uses Docker as the underlying container CRUD system. At some point when using k8s one will likely need a private Docker registry.

Frankly the hardest part of this is getting the SSL certificates to work.

## Assumptions

* Ubuntu 16.04
* Docker is installed, in this post it's `17.03.1~ce-0~ubuntu-xenial`

## Create SSL Certificates

I'm not 100% sure of the model I'm using to create the SSL certificates, but it is working with Docker. I don't know if you'd want to use it in production. :)

First, I create a `ca.conf` file. You might want to edit the distinguished name variables as well as the CN and alt_names.

```
[req]
distinguished_name = req_distinguished_name
req_extensions  = v3_req
x509_extensions = v3_ca
prompt = no
[req_distinguished_name]
C = CA
ST = Alberta
L = Edmonton
O = Example.com
OU = CA
CN = ca.example.com
[v3_req]
keyUsage = keyEncipherment, dataEncipherment, keyCertSign
extendedKeyUsage = serverAuth
subjectAltName = @alt_names
[ v3_ca ]
subjectKeyIdentifier=hash
authorityKeyIdentifier=keyid:always,issuer
basicConstraints = CA:true
[alt_names]
DNS.1 = ca.example.com
```

Next I also created a `server.conf`. You will want to change the CN and IP.1 or DNS.1. I am using an IP.

```
[req]
distinguished_name = req_distinguished_name
x509_extensions = v3_req
prompt = no
[req_distinguished_name]
C = CA
ST = Alberta
L = Edmonton
O = Example.com
OU = Docker
CN = registry.example.com
[v3_req]
keyUsage = keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names
basicConstraints = CA:FALSE

[alt_names]
#DNS.1 = registry.example.com
IP.1 = <ip of registry server>
```

Now that we have those configured, we can run this script. Note that this will update the local CA repository and restart Docker.

```
#!/bin/bash

set -e

openssl genrsa -out ca-privkey.pem 2048
openssl req -config ./ca.conf -new -x509 -key ca-privkey.pem \
     -out cacert.pem -days 365
openssl req -config ./server.conf -newkey rsa:2048 -days 365 \
     -nodes -keyout server-key.pem -out server-req.pem
openssl rsa -in server-key.pem -out server-key.pem
openssl x509 -req -in server-req.pem -days 365 \
      -CA cacert.pem -CAkey ca-privkey.pem \
      -set_serial 01 -out server-cert.pem  \
      -extensions v3_req \
      -extfile server.conf

echo "INFO: print cacert.pem..."
openssl x509 -text -in cacert.pem -noout
echo "INFO: print server-req.pem..."
openssl req -text -in server-req.pem -noout
echo "INFO: print server-cert.pem..."
openssl x509 -text -in server-cert.pem -noout
openssl verify -verbose -CAfile ./cacert.pem server-cert.pem

echo "INFO: updating local CA..."

# Have to use .crt file name for update command to work
sudo cp cacert.pem /usr/local/share/ca-certificates/cacert.crt
sudo update-ca-certificates
echo "INFO: restarting docker"
sudo service docker restart
```

The cacert.pem file would need to be distributed to all hosts that would use the private repository.

## Create a Docker registry

The [Docker documentation](https://docs.docker.com/registry/deploying/) has an example of doing this.

```
docker run -d -p 5000:5000 --restart=always --name registry \
  -v `pwd`/certs:/certs \
  -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/server-cert.pem \
  -e REGISTRY_HTTP_TLS_KEY=/certs/server-key.pem \
  registry:2
```

The above assumes the certs that were created with the `ssl.sh` script are in a `certs` directory.

## Push an Image

Now that a docker registry is running, I can push an image to it, and have done so. I find the Docker tagging and pushing system very awkward.

```
$ docker images
REPOSITORY                    TAG                 IMAGE ID            CREATED             SIZE
10.70.0.28:5000/static-site   latest              fcaa5e0ee8f2        9 hours ago         109 MB
static-site                   latest              fcaa5e0ee8f2        9 hours ago         109 MB
10.70.0.28:5000/static-site   <none>              60ceae523ef0        14 hours ago        109 MB
registry                      2                   9d0c4eabab4d        3 weeks ago         33.2 MB
```

Without the right SSL setup I wouldn't be able to push images.

```
$ docker -D push 10.70.0.28:5000/static-site
The push refers to a repository [10.70.0.28:5000/static-site]
6ce8e637d806: Layer already exists
a552ca691e49: Layer already exists
7487bf0353a7: Layer already exists
8781ec54ba04: Layer already exists
latest: digest: sha256:eb52222d9a7e00426ad94eacaf442dd07e52243ecec7f328537515f0b4c035da size: 1155
```

## Conclusion

The hardest part of this is SSL, which I'm sure I've done wrong but is working. Please let me know of any ways to do this better. :)
