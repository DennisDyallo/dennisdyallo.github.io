# Generating Crypto Test Data with OpenSSL and AI

## Overview
I recently needed to generate a comprehensive set of cryptographic test data, including private keys, public keys, Certificate Signing Requests (CSRs), and certificates. The data set covered multiple key types and sizes:

### RSA Keys
- RSA 1024-bit
- RSA 2048-bit
- RSA 3072-bit
- RSA 4096-bit

### Elliptic Curve Keys
- P-256
- P-384
- P-521
- Ed25519
- X25519

## The Challenge
Manually generating these files would have been time-consuming, potentially taking an hour or more. Instead, I leveraged AI to automate the process.

## Solution
I used Claude (an AI assistant) to generate a bash script utilizing OpenSSL commands. The prompt was straightforward:

```
Create a bash script that creates keys, certificates and certificates 
with attestation using OpenSSL commands.

I want:
RSA: RSA1024, 2048, 3072 and 4096
EC: P256, P384, P521, ED25519 and X25519
```

## Results
- Generated 42 files containing various cryptographic assets in less than a minute
- Script executed successfully without modifications
- Data ready for immediate use in unit and integration tests

## Future Use
The script has been added to our repository to:
- Allow developers to regenerate test data when needed
- Serve as a reference for similar cryptographic operations
- Provide a template for custom key generation scenarios

## Next Steps
I'll be incorporating this generated data into our test suite and documenting the process for team reference.

## The Script

```bash
#!/bin/bash

# Function to generate RSA keys and certificates
generate_rsa() {
    local bits=$1
    echo "Generating RSA-$bits keys and certificates..."
    
    # Generate private key
    openssl genrsa -out rsa${bits}_private.pem $bits
    
    # Generate public key
    openssl rsa -in rsa${bits}_private.pem -pubout -out rsa${bits}_public.pem
    
    # Generate CSR
    openssl req -new -key rsa${bits}_private.pem -out rsa${bits}.csr -subj "/C=US/ST=State/L=City/O=Organization/OU=Unit/CN=rsa${bits}.example.com"
    
    # Generate self-signed certificate
    openssl x509 -req -days 365 -in rsa${bits}.csr -signkey rsa${bits}_private.pem -out rsa${bits}_cert.pem
    
    # Generate certificate with attestation
    openssl x509 -req -days 365 -in rsa${bits}.csr -signkey rsa${bits}_private.pem -out rsa${bits}_cert_attest.pem \
        -extfile <(printf "keyUsage=digitalSignature,keyEncipherment\nsubjectKeyIdentifier=hash\nauthorityKeyIdentifier=keyid:always,issuer\nbasicConstraints=CA:TRUE")
}

# Function to generate EC keys and certificates
generate_ec() {
    local curve=$1
    local curve_param
    
    case $curve in
        "P256") curve_param="prime256v1" ;;
        "P384") curve_param="secp384r1" ;;
        "P521") curve_param="secp521r1" ;;
        "ED25519") curve_param="ed25519" ;;
        "X25519") curve_param="x25519" ;;
        *) echo "Invalid curve"; return 1 ;;
    esac
    
    echo "Generating $curve keys and certificates..."
    
    if [[ $curve == "ED25519" || $curve == "X25519" ]]; then
        # Generate private key
        openssl genpkey -algorithm $curve_param -out ${curve,,}_private.pem
        
        # Generate public key
        openssl pkey -in ${curve,,}_private.pem -pubout -out ${curve,,}_public.pem
    else
        # Generate private key
        openssl ecparam -name $curve_param -genkey -noout -out ${curve,,}_private.pem
        
        # Generate public key
        openssl ec -in ${curve,,}_private.pem -pubout -out ${curve,,}_public.pem
    fi
    
    # Generate CSR
    openssl req -new -key ${curve,,}_private.pem -out ${curve,,}.csr -subj "/C=US/ST=State/L=City/O=Organization/OU=Unit/CN=${curve,,}.example.com"
    
    # Generate self-signed certificate
    openssl x509 -req -days 365 -in ${curve,,}.csr -signkey ${curve,,}_private.pem -out ${curve,,}_cert.pem
    
    # Generate certificate with attestation
    openssl x509 -req -days 365 -in ${curve,,}.csr -signkey ${curve,,}_private.pem -out ${curve,,}_cert_attest.pem \
        -extfile <(printf "keyUsage=digitalSignature,keyEncipherment\nsubjectKeyIdentifier=hash\nauthorityKeyIdentifier=keyid:always,issuer\nbasicConstraints=CA:TRUE")
}

# Create directory for keys and certificates
mkdir -p crypto_keys
cd crypto_keys

# Generate RSA keys and certificates
for bits in 1024 2048 3072 4096; do
    generate_rsa $bits
done

# Generate EC keys and certificates
for curve in P256 P384 P521 ED25519 X25519; do
    generate_ec $curve
done

echo "All keys and certificates have been generated in the crypto_keys directory."

```

## Implementation Notes
- Script requires OpenSSL installed on your system
- Outputs are generated in the current directory
- Files are named systematically based on key type and size

---
*Last updated: February 4, 2025*