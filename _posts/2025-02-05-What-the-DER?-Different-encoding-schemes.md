[Home](/) | [Blog](/blog) | [About](/about) | [GitHub](https://github.com/dennisdyallo)

# From JSON to Crypto: A Web Developer's Guide to Encoding Standards

As a web developer transitioning into cryptographic development at Yubico, I've had to expand my encoding knowledge significantly beyond the comfortable world of JSON. Working with cryptographic protocols has introduced me to a variety of encoding methods that serve different purposes in the security ecosystem.

This post aims to demystify these encoding standards and provide a practical reference for developers in similar situations.

## The web developer's perspective

Coming from web development, JSON was my go-to format for data exchange. It's human-readable, well-supported, and straightforward. However, in the cryptographic world, we need formats that can handle binary data efficiently, ensure deterministic encoding, and maintain compatibility with legacy systems.

## Understanding the encoding landscape

Let's break down the main encoding methods you'll encounter in cryptographic development:

### Quick reference table

| Format | Structure                | Use Cases          | Advantages          | Disadvantages                  |
| ------ | ------------------------ | ------------------ | ------------------- | ------------------------------ |
| TLV    | Tag-Length-Value         | Smart cards, EMV   | Simple, lightweight | No type info, minimal metadata |
| BER    | Tree structure, flexible | Legacy systems     | Flexible encoding   | Ambiguous, larger size         |
| DER    | Strict BER subset        | X.509, PKI         | Canonical form      | Complex parsing                |
| PEM    | Base64(DER) + headers    | Certificates, keys | Human readable      | Larger size                    |
| COSE   | CBOR maps                | WebAuthn, IoT      | Compact, modern     | Newer, less supported          |

## Deep dive into each format

### TLV (Tag-Length-Value)

Think of TLV as the assembly language of encoding - it's bare-metal simple but powerful. Here's an Ed25519 public key in TLV:

```
02                    # Tag (public key)
20                    # Length (32 bytes)
183DD78A...A9AA       # Value (key bytes)
```

### BER/DER (ASN.1)

BER (Basic Encoding Rules) and DER (Distinguised Encoding Rules) implementations of ASN.1 (Abstract Syntax Notation 1), which is a standard interface description language (IDL) for defining data structures that can be serialized and deserialized in a cross-platform way. BER and DER are like strict JSON with a binary twist. DER is just BER with strict rules to ensure there's only one valid way to encode data:

```
30 2A                 # Sequence
   30 05              # Inner sequence
      06 03           # OID
         2B 65 70     # Ed25519
   03 21              # BitString
      00              # Padding
      [32 key bytes]
```

### PEM (Privacy-Enhanced Mail)

PEM is your friendly neighborhood format - it wraps binary DER data in Base64 with clear headers:

```
-----BEGIN PUBLIC KEY-----
MCowBQYDK2VwAyEAGD3XimAmDGEYgGkS7BQIi6YJsm8bkJdo6rbHl2z4qao=
-----END PUBLIC KEY-----
```

### COSE (CBOR Object Signing and Encryption)

COSE is the modern challenger, using CBOR to create compact, web-friendly encodings:

```
A4                    # Map(4)
   01 01             # kty:OKP
   03 38 07          # alg:EdDSA
   20 06             # crv:Ed25519
   21 58 20          # x:bytes(32)
      [32 key bytes]
```

## Working with different formats

Here's a practical example of converting between formats in C#:

```csharp
// PEM to Raw bytes
byte[] rawKey = Convert.FromBase64String(pemString
    .Replace("-----BEGIN PUBLIC KEY-----", "")
    .Replace("-----END PUBLIC KEY-----", "")
    .Replace("\n", ""))
    .Skip(12)  // Skip ASN.1 metadata
    .ToArray();

// Raw to COSE
byte[] coseKey = new byte[] {
    0xA4,       // Map of 4 pairs
    0x01,       // Key 1 (kty)
    0x01,       // Value: OKP (Octet Key Pair)
    /* ... */   // Additional COSE map headers
    }
    .Concat(rawKey) // Append raw 32 key bytes
    .ToArray();
```

## When to use what

1. **TLV**: Perfect for resource-constrained environments or when working directly with hardware.
2. **BER/DER**: The standard choice for PKI and certificate operations.
3. **PEM**: When you need human-readable formats or are working with text-based protocols.
4. **COSE**: The modern choice for WebAuthn and IoT applications.

## Security best practices

Remember these key points when working with encoded data:

1. **Always Validate Input**

   - Check lengths match declared values
   - Verify structure before parsing
   - Don't assume valid formatting

2. **Handle Memory Carefully**
   - Clear sensitive data after use
   - Use fixed-length buffers where appropriate
   - Be aware of encoding size changes

## Learning from experience

Working at Yubico has taught me that understanding these encoding formats is crucial for cryptographic development. While they might seem complex at first, each serves a specific purpose in the security ecosystem.

The key is to understand the context and requirements of your specific use case. Are you working with hardware? Need web compatibility? Dealing with certificates? Let these requirements guide your choice of encoding format.

---

_Last updated: February 5, 2025_
