# SSL Certificate Generation and JKS Key Store Creation

This document explains the process of generating SSL certificates and creating Java KeyStore (JKS) files for secure MongoDB connections.

## Certificate Generation Process

### 1. Generate CA (Certificate Authority) Key and Certificate

```bash
openssl genrsa -out ca.key 4096
openssl req -x509 -new -nodes -key ca.key -sha256 -days 1024 -out ca.crt -subj "/C=IN/ST=Tamil Nadu/L=Chennai/O=M2P/OU=Engineering/CN=M2P CA"
```

- Generates a 4096-bit RSA private key for the CA
- Creates a self-signed X.509 certificate for the CA, valid for 1024 days

### 2. Generate Server Key and Certificate Signing Request (CSR)

```bash
openssl genrsa -out server.key 4096
openssl req -new -key server.key -out server.csr -subj "/C=IN/ST=Tamil Nadu/L=Chennai/O=M2P/OU=Engineering/CN=mongodb"
```

- Generates a 4096-bit RSA private key for the server
- Creates a Certificate Signing Request (CSR) for the server

### 3. Generate Server Certificate with Subject Alternative Name (SAN)

```bash
echo "subjectAltName=DNS:mongodb,DNS:localhost,IP:127.0.0.1" > san.cnf
openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt -days 365 -sha256 -extfile san.cnf
```

- Creates a configuration file for SAN
- Generates the server certificate, signed by the CA, valid for 365 days, with SAN

### 4. Combine Server Key and Certificate

```bash
cat server.key server.crt > server.pem
```

- Combines the server's private key and certificate into a single PEM file

### 5. Generate Client Key and CSR

```bash
openssl genrsa -out client.key 4096
openssl req -new -key client.key -out client.csr -subj "/C=IN/ST=Tamil Nadu/L=Chennai/O=M2P/OU=Engineering/CN=nodejs-client"
```

- Generates a 4096-bit RSA private key for the client
- Creates a CSR for the client

### 6. Generate Client Certificate

```bash
openssl x509 -req -in client.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out client.crt -days 365 -sha256
```

- Generates the client certificate, signed by the CA, valid for 365 days

### 7. Combine Client Key and Certificate

```bash
cat client.key client.crt > client.pem
```

- Combines the client's private key and certificate into a single PEM file

## JKS Key Store Creation

### 1. Import CA Certificate into Trust Store

```bash
keytool -importcert -alias MongoDBCACert -file ./ssl/ca.crt -keystore ./ssl/truststore.jks -storepass mypassword
```

- Imports the CA certificate into a new JKS trust store

### 2. Convert Client Certificate to PKCS12 Format

```bash
openssl pkcs12 -export -in ./ssl/client.crt -inkey ./ssl/client.key -out ./ssl/certificate.p12 -name "client-cert"
```

- Converts the client certificate and key to PKCS12 format

### 3. Import PKCS12 into JKS Key Store

```bash
keytool -importkeystore -srckeystore ./ssl/certificate.p12 -srcstoretype pkcs12 -destkeystore ./ssl/client-cert.jks -deststorepass mypassword
```

- Imports the PKCS12 file into a JKS key store

## Purpose and Connections

1. **CA Certificate (ca.crt)**: This is the root of trust. It's used to sign both server and client certificates, establishing a chain of trust.

2. **Server Certificate (server.crt)**: Authenticates the MongoDB server to clients. It's signed by the CA, allowing clients to verify the server's identity.

3. **Client Certificate (client.crt)**: Authenticates the client to the MongoDB server. It's also signed by the CA, allowing the server to verify the client's identity.

4. **Trust Store (truststore.jks)**: Contains the CA certificate. Clients use this to verify the server's certificate.

5. **Key Store (client-cert.jks)**: Contains the client's private key and certificate. Clients use this for authentication to the server.

These certificates and keys work together to provide:
- Server Authentication: Clients verify the server's identity
- Client Authentication: The server verifies the client's identity
- Encryption: Secure communication between client and server

By using a common CA to sign both server and client certificates, we establish a trusted environment where both parties can authenticate each other.