# internal-ca
My internal CA used for managing certificates for internal infrastructure and services.

Steps for creating certificates that are signed by a custom (self signed) CA and trusted by the system.

Based on https://jamielinux.com/docs/openssl-certificate-authority/introduction.html, but with no revocation list, ocsp, simpler folder structure etc. Obviously, use the resource above for a more complete guide.

# Root CA

1. Create Root CA private key

```
openssl genrsa -aes256 -out mdf-root-ca-private-key.pem 4096
Enter PEM pass phrase: <encryption key for the private key>
```

2. Create Root CA certificate

```
openssl req -config openssl.cnf -key mdf-root-ca-private-key.pem -new -x509 -days 7300 -sha256 -extensions v3_ca -out mdf-root-ca-cert.pem -subj "/C=HR/L=Zagreb/CN=Mattia de Filippo Root CA"

openssl x509 -noout -text -in mdf-root-ca-cert.pem
```

# Intermediate CA

1. Create Intermediate CA private key

```
openssl genrsa -aes256 -out mdf-intermediate-ca-private-key.pem 4096 
Enter PEM pass phrase: <encryption key for the private key>
``` 

2. Create Intermediate CSR: 
 
```
openssl req -config openssl.cnf -new -sha256 -key mdf-intermediate-ca-private-key.pem -out mdf-intermediate-ca-csr.pem -subj "/C=HR/L=Zagreb/CN=Mattia de Filippo Intermediate CA"
openssl req -in mdf-intermediate-ca-csr.pem -text -noout 
```

3. Sign Intermediate CSR with Root CA private key (create Intermediate CA certificate)

```
openssl x509 -req -in mdf-intermediate-ca-csr.pem -CA mdf-root-ca-cert.pem -CAkey mdf-root-ca-private-key.pem -out mdf-intermediate-ca-cert.pem -days 3650 -sha256 -extensions v3_intermediate_ca -extfile openssl.cnf

openssl x509 -noout -text -in mdf-intermediate-ca-cert.pem 
openssl verify -CAfile mdf-root-ca-cert.pem mdf-intermediate-ca-cert.pem 
```

4. Create chain file

```
cat mdf-intermediate-ca-cert.pem mdf-root-ca-cert.pem > mdf-ca-chain.pem
```

# Certificates Examples

## Specific Server Certificate

1. Create Server private key

```
openssl genrsa -out mdf-server-private-key.pem 2048
```

2. Create Server CSR

```
openssl req -config openssl.cnf -key mdf-server-private-key.pem -new -sha256 -out server-csr.pem -subj "/C=HR/ST=Zagreb/L=Zagreb/O=Local Lab/OU=Local Lab/CN=server.mdf.internal" -addext "subjectAltName=DNS:server.mdf.internal"
openssl req -in server-csr.pem -text -noout
```
3. Sign Server CSR with Intermediate CA private key (create Server certificate)

```
openssl x509 -req -in server-csr.pem -CA mdf-intermediate-ca-cert.pem -CAkey mdf-intermediate-ca-private-key.pem -out mdf-server-cert.pem -days 365 -sha256 -extfile openssl.cnf -extensions server_cert -copy_extensions copyall
openssl x509 -noout -text -in mdf-server-cert.pem
openssl verify -CAfile mdf-ca-chain.pem mdf-server-cert.pem
```

## Wildcard Server Certificate

1. Create Wildcard Server private key

```
openssl genrsa -out mdf-star-private-key.pem 2048
```

2. Create Wildcard Server CSR

```
openssl req -config openssl.cnf -key mdf-star-private-key.pem -new -sha256 -out star-local-lab-csr.pem -subj "/C=HR/ST=Zagreb/L=Zagreb/O=Local Lab/OU=Local Lab/CN=*.mdf.internal" -addext "subjectAltName=DNS:*.mdf.internal,DNS:mdf.internal"
openssl req -in star-local-lab-csr.pem -text -noout
```

3. Sign Wildcard Server CSR with Intermediate CA private key (create Wildcard Server certificate)


```
openssl x509 -req -in star-local-lab-csr.pem -CA mdf-intermediate-ca-cert.pem -CAkey mdf-intermediate-ca-private-key.pem -out star-local-lab-cert.pem -days 365 -sha256 -extfile openssl.cnf -extensions server_cert -copy_extensions copyall
openssl x509 -noout -text -in star-local-lab-cert.pem
openssl verify -CAfile mdf-ca-chain.pem star-local-lab-cert.pem
```

## Client Certificate

1. Create Client private key

```
openssl genrsa -out mdf-client-private-key.pem 2048
```
2. Create Client CSR

```
openssl req -config openssl.cnf -key mdf-client-private-key.pem -new -sha256 -out client-csr.pem -subj "/C=HR/ST=Zagreb/L=Zagreb/O=Local Lab/OU=Local Lab/CN=Mattia de Filippo Client Certificate"
openssl req -in client-csr.pem -text -noout
```
3. Sign Client CSR with Intermediate CA private key (create Client certificate)

```
openssl x509 -req -in client-csr.pem -CA mdf-intermediate-ca-cert.pem -CAkey mdf-intermediate-ca-private-key.pem -out mdf-client-cert.pem -days 365 -sha256 -extfile openssl.cnf -extensions usr_cert -copy_extensions copyall
openssl x509 -noout -text -in mdf-client-cert.pem
openssl verify -CAfile mdf-ca-chain.pem mdf-client-cert.pem
```
