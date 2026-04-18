+++
title = 'TLS'
date = '2025-11-15T09:39:02-05:00'
weight = 60
draft = false
+++

Transport Security Layer (TLS)---also called a Secure Socket Layer (SSL) certificate---encrypts communications between the client and server. It serves two purposes:
- **Encryption**: Ecrypts the data in transit between client and server.
- **Authentication**: Verifies the identity of the server to client, and vice versa.

A TLS certificate is signed by a trusted Certificate Authority (CA). This "trust" comes from a root CA cert that is already installed in the client's trust store, which is in the OS or browser. In reality, the cert is signed by an intermediate CA, which chains up to a trusted root. 

The cert contains the server's public key and identity (domain name). The domain name is indicted by the Subject Alternative Names (SANs). Previously, this was indicated by the Common Name (CN).

When a client connects to a server, the server presents its certificate. Next, the client verifies the server certificate signature to ensure it was signed by a trusted CA, its expiration date, and that the domain name in the cert matches the client's hostname.

HTTPS is layering HTTP on top of the TLS layer.

## File formats

Both `.crt` and `.pem` files can contain the same types of data encoded in different ways. PEM files are more widely supported across different platforms and software.

| Aspect                       | .crt                                | .pem                         |
| :--------------------------- | :---------------------------------- | :--------------------------- |
| File extension meaning       | Conventional name for a certificate | Indicates PEM encoding       |
| Encoding format              | DER (binary) or PEM                 | PEM only (Base64 ASCII)      |
| Human-readable               | Sometimes                           | Yes                          |
| Can contain multiple objects | Rarely                              | Yes                          |
| Typical contents             | X.509 certificate                   | Certs, keys, chains          |
| Platform support             | Depends on encoding                 | Widely supported             |
| Used by                      | OpenSSL, browsers, servers          | OpenSSL, servers, containers |
| Parsing determined by        | File contents                       | File contents                |

### `.crt` files

`.crt` files are used to store certificates that contain public keys to verify the ownership of a public key with the identity of the certificate holder.

The `.crt` extension is traditionally used for certificate files. The file extension does not matter---the content and encoding format matter. `.crt` files are typically encoded in binary form in the Distinguished Encoding Rules (DER) format, but they can be encoded in the human-readable Privacy Enhanced Mail (PEM) format.




### PEM files

As its name suggests, Privacy Enhanced Mail (PEM) is a format that was originally used in email encryption but has become a standard format for storing and exchanging cyrptographic material such as certificates, private keys, and intermediate certificates.

PEM files are ASCII-encoded and use Base64 encoding in the following format:

```bash
-----BEGIN CERTIFICATE-----

# base64 data

-----END CERTIFICATE-----
```

This format makes them more human readable.

`.pem` keys can contain multiple certs and keys in the same file, which makes them suitable for various configurations, such as certificate chains.


## Generating certs and keys

For testing purposes, you can generate a certificate and a self-signed key with SSL. In a production environment, you want to get your private key signed by a real CA.

### Command options

Here are the common `openssl` command options to generate a certificiate and key for testing:

| Option               | Meaning                     | Description                                                                                                                    |
| -------------------- | --------------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| `req`                | Certificate request command | Tells OpenSSL to use the X.509 certificate request and creation tool.                                                          |
| `-x509`              | Create a self-signed cert   | Outputs a self-signed X.509 certificate instead of generating a CSR.                                                           |
| `-newkey rsa:4096`   | Generate a new key pair     | Creates a new private key and certificate at the same time. Here it uses a 4096-bit RSA key.                                   |
| `-keyout <filename>` | Output private key file     | Saves the generated private key to `key.pem`.                                                                                  |
| `-out <filename>`    | Output certificate file     | Writes the generated certificate to `cert.pem`.                                                                                |
| `-days 365`          | Validity period             | Certificate will be valid for 365 days.                                                                                        |
| `-nodes`             | No DES (no encryption)      | Saves the private key without a passphrase (unencrypted). Necessary for servers that must start without manual password entry. |


### PEM

The following command generates a private, self-signed CA certificate and key. A self-signed certificate is signed with its own key. `key.epm` is the private key, and `cert.pem` is the private certificate:

```bash
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365 -nodes
...
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:US
State or Province Name (full name) [Some-State]:
Locality Name (eg, city) []:
Organization Name (eg, company) [Internet Widgits Pty Ltd]:Go Stuff
Organizational Unit Name (eg, section) []:
Common Name (e.g. server FQDN or YOUR name) []:localhost
Email Address []:example@example.com
```

### CRT

Generate a 2048-bit RSA private key and save it to a file named `mydomain.key`:

```bash
openssl genrsa -out mydomain.key 2048
```

Create a certificate signing request (CSR), which is a request to a CA to sign your public key and create a certificate. The CSR includes information about your domain and organization. THis command initiates a series of prompts about your organization, including the Common Name (CN)---your domain name:

```bash
openssl req -new -key mydomain.key -out mydomain.csr
```

For development purposes, you can create a self-signed certificate by signing the CSR with your own private key:

```bash
openssl x509 -req -days 365 -in mydomain.csr -signkey mydomain.key -out mydomain.crt
```

## Serving with TLS

Go provides two functions for serving data over TLS:
- `ListenAndServeTLS`
- `LoadX509KeyPair`


### ListenAndServeTLS

`ListenAndServeTLS` is a convenience function for HTTPS that loads a certificate and key from disk, creates a TLS configuration, and starts an HTTPS server. Use this in the following scenarios:
  - You have one cert + one key
  - No custom TLS settings
  - No SNI, mTLS, hot reload, or custom cipher control
  - You want the simplest possible HTTPS server

Serve PEM files with `ListenAndServeTLS`. It has the following parameters:
- `cert.pem`: The SSL certificate that is issued by a Certificate Authority (CA) such as Let's Encrypt.
- `key.pem`: Private key for the server.

```go
func index(w http.ResponseWriter, r *http.Request) {
	w.Write([]byte("Hello TLS"))
}

func main() {
	http.HandleFunc("/", index)
	http.ListenAndServeTLS(":8000", "cert.pem", "key.pem", nil)
}
```

### LoadX509KeyPair

`LoadX509KeyPair` is a low-level utility that reads cert and key files, parses them, then returns a `tls.Certificate` struct. You can use `LoadX509KeyPair` beyond HTTP. For example, gRPC and mTLS.

This example sets up the TLS configuration for a TCP server:

```go
func main() {
	cert, err := tls.LoadX509KeyPair("keys/mydomain.crt", "keys/mydomain.key")
	if err != nil {
		panic(err)
	}

	config := &tls.Config{Certificates: []tls.Certificate{cert}}

	listener, err := tls.Listen("tcp", ":8443", config)
	if err != nil {
		panic(err)
	}

	// low-level TCP logic
}
```

## Reverse proxy (TLS)

