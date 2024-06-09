---
title: "Create a SSL certificate for your home network Part 2"
slug: create-a-ssl-certificate-for-your-home-network-part-2
aliases:
- /2022/11/25/create-a-ssl-cert-for-your-home-network-part2.html
date: 2022-11-25
---

As a follow up to my previous post [Create a SSL certificate for your home network](https://www.alexoswald.com/2022/08/22/create-a-ssl-cert-for-your-home-network.html), I've created a simple shell script you can to use to quickly create ssl certificates for a local domain.

Create a `create_ssl.sh` file with the following contents:

```bash
# GENERATE SSL CERTIFICATE 
# pi.lan, *.lan
#
# PARAMETERS
# 1: Root Password
# 2: Validity in days
# 3: PFX Password
#
# EXECUTION
# chomd +x create_ssl.sh
# ./create_ssl.sh rootkeypass 365 pfxpass

echo "Generating RSA-2048 key file 'root.key'"
openssl genrsa -des3 -passout pass:$1 -out root.key 4096

echo "Generating SSL configuration file: 'ssl.config'"
cat >> ssl.config <<EOL
default_bits = 4096
prompt = no
default_md = sha256
distinguished_name = dn

[dn]
C=US
ST=WA
L=Seattle
O=Local LAN Cert
OU=Local LAN Cert
emailAddress=admin@pi.lan
CN = Local LAN Cert
EOL

echo "Generating Root Certificate file 'root.pem'"
openssl req -x509 -new -nodes -key root.key -sha256 -days $2 -out root.pem -passin pass:$1 -config ssl.config

echo "Generating Certificate Extensions file 'cert.ext'"
cat >> cert.ext <<EOL
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
subjectAltName = @alt_names

[alt_names]
DNS.1 = *.pi.lan
DNS.2 = pi.lan
EOL

echo "Generating Certificate Signing Request (CSR) files 'sslcert.csr', 'sslcert.key'"
openssl req -new -sha256 -nodes -out sslcert.csr -newkey rsa:4096 -keyout sslcert.key -config ssl.config

echo "Generating Certificate file 'sslcert.crt'"
openssl x509 -req -in sslcert.csr -CA root.pem -CAkey root.key -CAcreateserial -out sslcert.crt -days 365 -sha256 -extfile cert.ext -passin pass:$1

echo "Generating PFX file 'sslcert.pfx'"
openssl pkcs12 -export -out sslcert.pfx -inkey sslcert.key -in sslcert.crt -passout pass:$3
```