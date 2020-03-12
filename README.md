---
page: https://idle.run/catalina-ssl
title: "Helpful Scripts for Self-Signed SSL On Catalina"
tags: ssl openssl osx catalina
date: 2020-03-09
---

## Catalina SSL

Catalina introduced some new requirements that can cause issues with previous methods of creating self-signed SSL certs.

If certs are generated without meeting the requirements, the Chrome self-signed cert warning does not have an option to proceed with the invalid cert. (It actually *is* still possible to bypass the warning by typing the secret code "thisisunsafe" on the page).

The key requirements are outlined here: [https://support.apple.com/en-ca/HT210176]

A minimal command for creating a self-signed SSL cert in OpenSSL is as follows:

```
openssl genrsa -out server.key 2048

SERVERNAME="MyTestServer"
printf "[EXT]\nsubjectAltName=DNS:$SERVERNAME\nextendedKeyUsage=serverAuth" > .ext-data
cat /etc/ssl/openssl.cnf .ext-data > .ext-full

openssl req -new -sha256 -key server.key -subj "/CN=$SERVERNAME" -reqexts EXT -config .ext-full -out server.csr
openssl x509 -signkey server.key -in server.csr -req -days 825 -out server.cert -extensions EXT -extfile .ext-data -sha256
openssl x509 -text -in server.cert -noout

rm -f server.csr .ext-data .ext-full
```

### Verify Cert Details

`openssl x509 -in server.cert -noout -text`

Check for the key lines:

```
Signature Algorithm: sha256WithRSAEncryption
...
RSA Public-Key: (2048 bit)
...
X509v3 extensions:
  X509v3 Subject Alternative Name:
    DNS:MyTestServer
  X509v3 Extended Key Usage:
    TLS Web Server Authentication
```
