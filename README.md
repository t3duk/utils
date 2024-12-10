# Better Auth Utils

A simple typescript API for common auth utilities like hashing, encryption, encoding, and OTP generation.

It integrates [uncrypto](https://github.com/unjs/uncrypto) to provide a unified API for both Node.js (using the Crypto module) and web environments (using the Web Crypto API) through Conditional Exports.

```bash
pnpm add @better-auth/utils
```

## Utilities at a Glance

utilities provided by `@better-auth/utils`:

| Utility          | Description                                        |
|-------------------|----------------------------------------------------|
| [**Digest**](#digest) | Hash inputs using sha family hash functions.      |
| [**HMAC**](#hmac) | Hash inputs using HMAC with a secret key.          |
| [**Random String**](#random-string) | Generate random strings with a specified length and charset. |
| [**RSA**](#rsa)   | Perform encryption, decryption, signing, and verification with RSA keys. |
| [**ECDSA**](#ecdsa) | Perform signing and verification with ECDSA keys. |
| [**Base64**](#base64) | Encode and decode data in base64 format.          |
| [**Hex**](#hex)   | Encode and decode data in hexadecimal format.      |
| [**OTP**](#otp) | Generate and verify one-time passwords.            |

## Digest

Digest provides a way to hash an input using sha family hash functions. It wraps over `crypto.digest` and provide utilities to encode output in hex or base 64.

```ts
import { digest } from "@better-auth/utils/digest"

const hashBuffer = await digest("text", "SHA-256"); // "SHA1" | "SHA512" | "SHA382"
const hashInHex = await digest("text", "SHA-512", "hex"); // "raw" (default) 
```

To encode output in base64, you should use `base64` utility on the output buffer.

```ts
import { base64 } from "@better-auth/utils/base64"
const hashInBase64 = await base64.encode(await digest("text", "SHA-512"));
```

## HMAC

The HMAC utility allows you to securely hash data using a secret key and SHA family hash functions. It provides methods to `sign`, `verify`, and `create` a customized HMAC instance with specific hashing algorithms and encoding formats.

### Create HMAC

To create an HMAC instance, use the createHMAC function. You can specify the SHA family algorithm ("SHA-256", "SHA-384", or "SHA-512") and the desired encoding format ("none", "hex", "base64", "base64url", or "base64urlnopad").

It takes a secret key and returns a key object which could be used to sign and verify data.

```ts
import { createHMAC } from './hmac';

const hmac = createHMAC("SHA-256", "hex"); // Customize algorithm and encoding
```

### Import Key

The importKey method takes a secret key (string, buffer, or typed array) and returns a CryptoKey object that can be used for signing and verifying data.
  
```ts
const secretKey = "my-secret-key"; // Can also be a buffer or TypedArray
const key = await hmac.importKey(secretKey);
```

### Sign

The sign method takes a secret key (or CryptoKey) and data to generate a signature. If you provide a raw secret key, it will automatically be imported.

```ts
const key = await hmac.importKey("my-secret-key");
const signature = await hmac.sign(key, "text to sign");
console.log(signature); // Encoded based on the selected encoding format (e.g., hex)
```

You could also directly sign using the raw string secret key.

```ts
const signature2 = await hmac.sign("secret-key",{
    data: "text"
});
```

### Verify

The verify method checks if a given signature matches the data using the secret key. You can provide either a raw secret key or a CryptoKey.

```ts
const key = await hmac.importKey("my-secret-key");
const isValid = await hmac.verify(key, "text to sign", signature);
console.log(isValid); // true or false
```

## Random String

Random crypto secure string generator. It wraps over `crypto.getRandomValues` and provide utilities to generator based on length and charset.

1. first create a random string generator with desired charset.
```ts
import { createRandomStringGenerator } from "@better-auth/utils/random-string"

export const generateRandomString = createRandomStringGenerator("A-Z", "0-9", "a-z", "-_") 
```

2. generate random string based on length.
```ts
const randomString = generateRandomString(32)
const randomString2 = generateRandomString(32, "A-Z", "0-9") // override charset
```

## RSA

RSA utilities provide a simple interface to work with RSA cryptographic operations, such as generating key pairs, encrypting and decrypting data, and signing and verifying messages.

### Key Pair Generation

You can generate RSA key pairs with specified parameters. By default, the `modulusLength` is 2048 bits and the hash algorithm is `SHA-256`.

```ts
import { rsa } from "@better-auth/utils/rsa";

const keyPair = await rsa.generateKeyPair(2048, "SHA-256");
const { publicKey, privateKey } = keyPair;
```

### Exporting Keys

Export a public or private key in your preferred format.

```ts
const jwk = await rsa.exportKey(publicKey, "jwk");
const spki = await rsa.exportKey(publicKey, "spki");
```

### Importing Keys

Import a key in the `jwk` format for specific usage (`encrypt`, `decrypt`, `sign`, or `verify`).

```ts
const importedKey = await rsa.importKey(jwk, "encrypt");
```

### Encryption

Encrypt sensitive data using an RSA public key. Input can be a string, `ArrayBuffer`, `TypedArray` or `string`.

```ts
const encryptedData = await rsa.encrypt(publicKey, "Sensitive data");
```

### Decryption

Decrypt encrypted data using the corresponding RSA private key.

```ts
const decryptedData = await rsa.decrypt(privateKey, encryptedData);
const originalText = new TextDecoder().decode(decryptedData);
```

### Signing

Sign a message using the RSA private key. Input can be a string, `ArrayBuffer`, or `TypedArray`.

```ts
const signature = await rsa.sign(privateKey, "Message to sign");
```

### Verifying

Verify a signature against the original data using the RSA public key.

```ts
const isValid = await rsa.verify(publicKey, {
  signature,
  data: "Message to sign",
});
```

## ECDSA

ECDSA utilities provide a simple interface to perform key pair generation, signing, and verification using elliptic curve cryptography.

### Key Pair Generation

You can generate ECDSA key pairs with your preferred curve. Supported curves are `"P-256"`, `"P-384"`, and `"P-521"`.

```ts
import { ecdsa } from "@better-auth/utils/ecdsa";

const { privateKey, publicKey } = await ecdsa.generateKeyPair("P-256");
```

### Exporting Keys

Export a public or private key in your preferred format, such as `pkcs8` or `spki`.

```ts
const exportedPrivateKey = await ecdsa.exportKey(privateKey, "pkcs8");
const exportedPublicKey = await ecdsa.exportKey(publicKey, "spki");
```

### Importing Keys

Import an ECDSA private or public key in the appropriate format. Public keys can also be provided as strings.

```ts
const importedPrivateKey = await ecdsa.importPrivateKey(exportedPrivateKey, "P-256");
const importedPublicKey = await ecdsa.importPublicKey(exportedPublicKey, "P-256");
```

### Signing

Sign data using the ECDSA private key. The input can be a string or `ArrayBuffer`. You can specify the hash algorithm, which defaults to `"SHA-256"`.

```ts
const signature = await ecdsa.sign(privateKey, "Message to sign", "SHA-256");
```

### Verifying

Verify a signature against the original data using the ECDSA public key. Input can be a string or `ArrayBuffer`. Signature verification requires providing the signature, data, and hash algorithm (default: `"SHA-256"`).

```ts
const isValid = await ecdsa.verify(publicKey, {
  signature,
  data: "Message to verify",
  hash: "SHA-256",
});
```

## Base64

Base64 utilities provide a simple interface to encode and decode data in base64 format.

### Encoding

Encode data in base64 format. Input can be a string, `ArrayBuffer`, or `TypedArray`.

```ts
import { base64 } from "@better-auth/utils/base64";

const encodedData = base64.encode("Data to encode");
```

options:
- `urlSafe` - URL-safe encoding, replacing `+` with `-` and `/` with `_`.
- `padding` - Include padding characters (`=`) at the end of the encoded string

```ts
const encodedData = base64.encode("Data to encode", { url: true, padding: false });
```

### Decoding

Decode base64-encoded data. Input can be a string or `ArrayBuffer`.

```ts
const decodedData = await base64.decode(encodedData);
```

It automatically detects if the input is URL-safe and includes padding characters.


## Hex

Hex utilities provide a simple interface to encode and decode data in hexadecimal format.

### Encoding

Encode data in hexadecimal format. Input can be a string, `ArrayBuffer`, or `TypedArray`.

```ts
import { hex } from "@better-auth/utils/hex";

const encodedData = hex.encode("Data to encode");
```

### Decoding

Decode hexadecimal-encoded data. Input can be a string or `ArrayBuffer`.

```ts
const decodedData = hex.decode(encodedData);
```

## OTP

The OTP utility provides a simple and secure way to generate and verify one-time passwords (OTPs), commonly used in multi-factor authentication (MFA) systems. It includes support for both HOTP (HMAC-based One-Time Password) and TOTP (Time-based One-Time Password) standards.

It's implemented based on [RFC 4226](https://tools.ietf.org/html/rfc4226) and [RFC 6238](https://tools.ietf.org/html/rfc6238).

### Generating HOTP

HOTP generates a one-time password based on a counter value and a secret key. The counter should be incremented for each new OTP.

```ts
import { generateHOTP } from "@better-auth/utils/otp";
const secret = "my-super-secret-key";
const counter = 1234;
const otp = generateHOTP(secret, counter);
``` 

### Generating TOTP

TOTP generates a one-time password based on the current time and a secret key. The time step is typically 30 seconds.

```ts
import { generateTOTP } from "@better-auth/utils/otp";
const secret = "my-super-secret-key"
const otp = generateTOTP(secret);
```

### Verifying TOTP

Verify a TOTP against the secret key and a specified time window. The default time window is 30 seconds.

```ts
import { verifyTOTP } from "@better-auth/utils/otp";
const secret = "my-super-secret-key"
const isValid = verifyTOTP(secret, otp);
```

You can also specify the time window in seconds.

```ts
import { verifyTOTP } from "@better-auth/utils";
const isValid = verifyTOTP(secret, otp, { window: 60 });
```

### Generate QR Code

Generate a QR code URL for provisioning a TOTP secret key in an authenticator app.

```ts
import { generateQRCode } from "@better-auth/utils/otp";

const secret = "my-super-secret-key";
const label = "My Account";

const qrCodeUrl = generateQRCode(secret, label);
```

## License

MIT