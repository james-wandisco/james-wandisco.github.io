---
layout: post
title:  "Welcome to Jekyll!"
date:   2022-10-15 00:06:10 +0100
categories: jekyll update
---
# Generate self signed TLS certificates

Commands to generate self-signed certs and bundle them in keystore and truststores ready for use with LDM.

Example here: Server hostname is server_a, ip is 10.10.10.10

### Create your CA

1. Create a private key for your CA.
```
openssl genrsa -aes256 -out ca-private-key.pem 4096
```
2. Create your CA public cert with key from step 1.
```
openssl req -new -x509 -sha256 -days 365 -key ca-private-key.pem -out ca-public-cert.pem
```
----
### Create your LDM server key and certificate

3. Create a server private key.
```
openssl genrsa -out server-private-key.pem 4096
```

4. Create a signing request. (Replace with your values!)
```
openssl req -new -sha256 -subj "/CN=server_a" -key server-private-key.pem -out cert_request.csr
```

5. Create a config file to provide subjectAltName's, Add all/any IPs or DNS you need.(Replace with your values!)
```
echo "subjectAltName=DNS:server_a,DNS:localhost,IP:10.10.10.10" >> extfile.cnf
```
6. Create your new signed certificate from the csr
```
openssl x509 -req -sha256 -days 365 -in cert_request.csr -CA ca-public-cert.pem -CAkey ca-private-key.pem -out signed-cert.pem -extfile extfile.cnf -CAcreateserial
```
---
### Package keys and certs. (Package them whatever way your application supports! I use JKS here)

7. Chain your CA and server certificates.
```
cat signed-cert.pem > server_a_chain.pem
cat ca-public-cert.pem >> server_a_chain.pem
```
8. Bundle your server key and chained certificate together
```
openssl pkcs12 -export -in server_a_chain.pem -inkey server-private-key.pem -name server_a -out server_a.p12
```
9. Package your cert and key bundle into a java keystore for LDM.
```
keytool -importkeystore -deststorepass wandisco -destkeystore keystore.jks -srckeystore server_a.p12 -srcstoretype PKCS12
```

10. Package your CA cert into a java truststore for LDM.
```
keytool -import -alias CA -file ca-public-cert.pem -keystore truststore.jks -storepass wandisco
```

---
You will now have your **truststore.jks** and **keystore.jks**.
Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
