# Generate wildcard SSL Cert
https://gist.github.com/KeithYeh/bb07cadd23645a6a62509b1ec8986bbc 

## Step 1: Generate a Private Key
```
openssl genrsa -out server.key 2048
```

## Step 2: Generate a CSR (Certificate Signing Request)
```
openssl req -new -key server.key -out server.csr
```
Specify "pomerium.localhost" as Common Name  

## Step 3: Create config file for SAN
```
touch v3.ext
```

Filecontent
```
subjectKeyIdentifier   = hash
authorityKeyIdentifier = keyid:always,issuer:always
basicConstraints       = CA:TRUE
keyUsage               = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment, keyAgreement, keyCertSign
subjectAltName         = DNS:pomerium.localhost, DNS:*.pomerium.localhost
issuerAltName          = issuer:copy
```

## Step 4: Generating a Self-Signed Certificate
```
openssl x509 -req -in server.csr -signkey server.key -out server.crt -days 3650 -sha256 -extfile v3.ext
``` 

## Step 5: Convert to PEM
```
openssl x509 -in server.crt -out server.pem -outform PEM
```

## Step 6: Cleanup
```
rm server.crt server.csr v3.ext
```