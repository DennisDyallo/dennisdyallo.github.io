Response: #3 

# Cryptography Key Classes in Yubico .NET SDK

The Yubico .NET SDK has introduced a new key hierarchy to enhance key management, improve type safety, and support additional key types including standard RSA, EC, and Curve25519 keys. This guide provides comprehensive information on using these new classes with the PIV application.

## Overview of Changes

The SDK is transitioning from the legacy `PivPrivateKey`/`PivPublicKey` classes to the new interface-based `IPrivateKey`/`IPublicKey` approach. This change brings several benefits:

- Support for additional key types (RSA, EC, and Curve25519)
- Simplified disposal of sensitive data within new key classes
- Common interface to work with different key types throughout the SDK
- Improved type safety and consistency
- Reduced reliance on the `PivAlgorithm` enum
- Better integration with standard cryptographic operations

> **Important:** The new key classes require YubiKey version 5.7 or greater.

## Key Type Hierarchy

All keys implement either the `IPublicKey` or `IPrivateKey` interfaces, with specific implementations:

- **Public Keys**
  - `RSAPublicKey` - For RSA keys
  - `ECPublicKey` - For standard elliptic curve keys
  - `Curve25519PublicKey` - For Ed25519 and X25519 keys

- **Private Keys**
  - `RSAPrivateKey` - For RSA keys
  - `ECPrivateKey` - For standard elliptic curve keys
  - `Curve25519PrivateKey` - For Ed25519 and X25519 keys

## Available Key Types

The SDK supports the following key types, accessible via the `KeyType` enum:

| KeyType          | Description                      | Supported Operations   |
|------------------|----------------------------------|------------------------|
| `KeyType.RSA1024` | 1024-bit RSA key                | Sign, Decrypt          |
| `KeyType.RSA2048` | 2048-bit RSA key                | Sign, Decrypt          |
| `KeyType.RSA3072` | 3072-bit RSA key                | Sign, Decrypt          |
| `KeyType.RSA4096` | 4096-bit RSA key                | Sign, Decrypt          |
| `KeyType.ECP256`  | NIST P-256 elliptic curve       | Sign, Key Agreement    |
| `KeyType.ECP384`  | NIST P-384 elliptic curve       | Sign, Key Agreement    |
| `KeyType.Ed25519` | Edwards curve (for EdDSA)       | Sign                   |
| `KeyType.X25519`  | Montgomery curve (for ECDH)     | Key Agreement          |

## Factory Methods for Key Creation

### RSA Public Keys

```csharp
// Create from PKCS#8 SubjectPublicKeyInfo format
byte[] encodedKey = /* your SubjectPublicKeyInfo bytes */;
IPublicKey rsaPublicKey = RSAPublicKey.CreateFromPkcs8(encodedKey);

// Create from parameters
RSAParameters parameters = /* your RSA parameters */;
RSAPublicKey rsaPublicKey = RSAPublicKey.CreateFromParameters(parameters);
```

### RSA Private Keys

```csharp
// Create from PKCS#8 PrivateKeyInfo format
byte[] encodedKey = /* your PKCS#8 PrivateKeyInfo bytes */;
RSAPrivateKey rsaPrivateKey = RSAPrivateKey.CreateFromPkcs8(encodedKey);

// Create from parameters
RSAParameters parameters = /* your RSA parameters */;
RSAPrivateKey rsaPrivateKey = RSAPrivateKey.CreateFromParameters(parameters);
```

### EC Public Keys

```csharp
// Create from PKCS#8 SubjectPublicKeyInfo format
byte[] encodedKey = /* your SubjectPublicKeyInfo bytes */;
IPublicKey ecPublicKey = ECPublicKey.CreateFromPkcs8(encodedKey);

// Create from EC parameters
ECParameters parameters = /* your EC parameters */;
ECPublicKey ecPublicKey = ECPublicKey.CreateFromParameters(parameters);

// Create from public point (0x04 || X-coordinate || Y-coordinate)
ReadOnlyMemory<byte> publicPoint = /* uncompressed EC point with 0x04 prefix */;
IPublicKey ecPublicKey = ECPublicKey.CreateFromValue(publicPoint, KeyType.ECP256);
```

### EC Private Keys

```csharp
// Create from PKCS#8 PrivateKeyInfo format
byte[] encodedKey = /* your PKCS#8 PrivateKeyInfo bytes */;
ECPrivateKey ecPrivateKey = ECPrivateKey.CreateFromPkcs8(encodedKey);

// Create from EC parameters
ECParameters parameters = /* your EC parameters with D value */;
ECPrivateKey ecPrivateKey = ECPrivateKey.CreateFromParameters(parameters);

// Create from private scalar value
ReadOnlyMemory<byte> privateValue = /* your private scalar value */;
ECPrivateKey ecPrivateKey = ECPrivateKey.CreateFromValue(privateValue, KeyType.ECP256);
```

### Curve25519 Public Keys

```csharp
// Create from PKCS#8 SubjectPublicKeyInfo format
byte[] encodedKey = /* your SubjectPublicKeyInfo bytes */;
Curve25519PublicKey curve25519PublicKey = Curve25519PublicKey.CreateFromPkcs8(encodedKey);

// Create from public point bytes
ReadOnlyMemory<byte> publicPoint = /* your public point bytes */;
Curve25519PublicKey curve25519PublicKey = Curve25519PublicKey.CreateFromValue(publicPoint, KeyType.Ed25519);
// Or for X25519
Curve25519PublicKey x25519PublicKey = Curve25519PublicKey.CreateFromValue(publicPoint, KeyType.X25519);
```

### Curve25519 Private Keys

```csharp
// Create from PKCS#8 PrivateKeyInfo format
byte[] encodedKey = /* your PKCS#8 PrivateKeyInfo bytes */;
Curve25519PrivateKey curve25519PrivateKey = Curve25519PrivateKey.CreateFromPkcs8(encodedKey);

// Create from private scalar value
ReadOnlyMemory<byte> privateKey = /* your private key bytes */;
Curve25519PrivateKey ed25519PrivateKey = Curve25519PrivateKey.CreateFromValue(privateKey, KeyType.Ed25519);
// Or for X25519
Curve25519PrivateKey x25519PrivateKey = Curve25519PrivateKey.CreateFromValue(privateKey, KeyType.X25519);
```

## Using with PIV Session

### Generating Key Pairs

The `GenerateKeyPair` method now accepts a `KeyType` parameter:

```csharp
using Yubico.YubiKey;
using Yubico.YubiKey.Piv;
using Yubico.YubiKey.Cryptography;

using var pivSession = new PivSession(yubiKey)
// Setup key collector
pivSession.KeyCollector = yourKeyCollector;

// Authenticate with management key
pivSession.AuthenticateManagementKey();

// Generate an Ed25519 key pair
IPublicKey publicKey = pivSession.GenerateKeyPair(
    PivSlot.Authentication,
    KeyType.Ed25519,
    PivPinPolicy.Once,
    PivTouchPolicy.Never);

// Check the returned key type
if (publicKey is Curve25519PublicKey ed25519PublicKey)
{
    // Access the public point
    ReadOnlyMemory<byte> publicPoint = ed25519PublicKey.PublicPoint;
    
    // Export the key in SubjectPublicKeyInfo format
    byte[] exportedKey = ed25519PublicKey.ExportSubjectPublicKeyInfo();
}
```

### Importing Private Keys

Import private keys with the `ImportPrivateKey` method:

```csharp
// Create a private key to import
byte[] pkcs8EncodedKey = /* Load PKCS#8 private key */;
IPrivateKey privateKey = Curve25519PrivateKey.CreateFromPkcs8(pkcs8EncodedKey);

// Import into the YubiKey
using var pivSession = new PivSession(yubiKey)

pivSession.KeyCollector = yourKeyCollector;
pivSession.AuthenticateManagementKey();

pivSession.ImportPrivateKey(
    PivSlot.KeyManagement,
    privateKey,
    PivPinPolicy.Once,
    PivTouchPolicy.Never);

```

### Key Agreement with X25519

Perform key agreement operations with X25519 keys:

```csharp
// Generate or import an X25519 key
using var pivSession = new PivSession(yubiKey)

// Setup and authenticate
pivSession.KeyCollector = yourKeyCollector;
pivSession.VerifyPin();  // Required for cryptographic operations

// Get other party's public key
byte[] otherPartyPublicKeyBytes = /* Other party's X25519 public key */;

// Create a key agreement object (details depend on your implementation)
// This is a simplified example
Memory<byte> sharedSecret = new byte[32];
pivSession.KeyAgree(
    PivSlot.KeyManagement,  // Slot containing your private key
    otherPartyPublicKeyBytes, 
    sharedSecret);

// Use the shared secret for key derivation or encryption

```

## Exporting Keys

Export keys in standard formats:

```csharp
// Export public key in SubjectPublicKeyInfo format
byte[] spki = ecPublicKey.ExportSubjectPublicKeyInfo();

// Export private key in PKCS#8 PrivateKeyInfo format
byte[] pkcs8 = ecPrivateKey.ExportPkcs8PrivateKey();
```

## Handling Key Data Securely

All private key classes implement secure cleanup methods:

```csharp

// Using disposable pattern to ensure sensitive data is cleared
using (ECPrivateKey privateKey = ECPrivateKey.CreateFromPkcs8(encodedKey))
{
    // Use the private key
}

// The private key data is securely cleared from memory after disposal

```

```csharp
// Using try-finally and clearing explicityly
try
{
    ECPrivateKey privateKey = ECPrivateKey.CreateFromPkcs8(encodedKey);
    // Use the private key
}
finally
{
    // Ensure sensitive data is cleared
    privateKey.Clear();
}

```


## Migration Guide

The old PIV key types will still function, but are deprecated. The new classes are designed to be more usable and they support the new key types. Most users will find the new keys The old classes will be removed in a future version of the SDK. 
When updating your code from the old API to the new one:

1. Replace `PivAlgorithm` values with the corresponding `KeyType`:
   - `PivAlgorithm.Rsa2048` → `KeyType.RSA2048`
   - `PivAlgorithm.EccP256` → `KeyType.ECP256`

2. Update key type handling:
   - `PivPublicKey` → `IPublicKey` (base interface)
   - `PivPrivateKey` → `IPrivateKey` (base interface)
   - Use type checking (e.g., `is ECPublicKey`) to access specific properties

3. Use the new factory methods for key creation:
   - `RSAPublicKey.CreateFromPkcs8()`
   - `ECPrivateKey.CreateFromValue()`
   - `Curve25519PublicKey.CreateFromValue()`

These new key classes provide a more robust, type-safe approach to cryptographic operations with YubiKeys, while supporting modern key types like Ed25519 and X25519.