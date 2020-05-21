---
layout: post
title: Deploy Harbor with Helm and Custom CA Certs
categories:
header_image: "/img/harbor2.jpg"
header_permalink: "https://commons.wikimedia.org/wiki/File:Fossa-drawing.jpg"
summary: "Configuring tls secrets and helm values for harbor and custom tls"

---

# {{ page.title }}

This is just a quick post on how to use a custom CA with Helm and Harbor.

Below are the versions deployed. Note that Harbor 2.0 has recently been released, but here we are using Harbor 1.10.2.

```
$ helm ls
NAME          	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART       	APP VERSION
harbor-central	harbor   	1       	2020-05-21 07:20:04.248551864 -0700 PDT	deployed	harbor-1.3.2	1.10.2 
```

I'm using mkcert, and I've generated the cert files. Note that I'm creating a cert for both the harbor and notary service.

```
mkcert harbor.example.com notary.example.com
```

Now we need to create a tls secret. NOTE: "*tls*" secret not a generic secret.

```
kubectl create secret tls harbor-certs \
  --cert=harbor.example.com+1.pem \
  --key=harbor.example.com+1-key.pem 
```

Export the values file.

```
helm show values harbor/harbor > harbor-values.yaml
```

And edit that file so that it knows about "harbor-certs". E.g.

```
$ grep -A 2 -B 2 harbor-certs harbor-values.yaml 
    # link on portal to download the certificate of CA
    # These files will be generated automatically if the "secretName" is not set
    secretName: "harbor-certs"
    # By default, the Notary service will use the same cert and key as
    # described above. Fill the name of secret if you want to use a
```

And deploy!

```
helm install harbor-central harbor/harbor -f central-harbor-values.yaml
```

There are probably several other ways to do this, but this is certainly one! Of course this CA is going to have to be distributed as well.