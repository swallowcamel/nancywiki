The `Nancy.Cryptography` namespace contains a set of helpers for working with encryption and digital signing, that you can use in your applications or extensions.

## The IEncryptionProvider interface

Defines the functionality for encrypting and decrypting string-data. 

```c#
/// <summary>
/// Provides symmetrical encryption support
/// </summary>
public interface IEncryptionProvider
{
    /// <summary>
    /// Encrypt and base64 encode the string
    /// </summary>
    /// <param name="data">Data to encrypt</param>
    /// <returns>Encrypted string</returns>
    string Encrypt(string data);

    /// <summary>
    /// Decrypt string
    /// </summary>
    /// <param name="data">Data to decrypt</param>
    /// <returns>Decrypted string</returns>
    string Decrypt(string data);
}
```
Nancy provides two default implementations that you can use

- `NoEncryptionProvider` - Does not apply any encryption, only base64-encodes the string
- `RijndaelEncryptionProvider ` - Uses the Rijndael algorithm, with a 256-bits key and a 128-bits initialization vector (IV), to encrypt and base64-encode the string

## The IHmacProvider interface

Defines the functionality for generating Hash-based Message Authentication Codes ([HMACs](http://en.wikipedia.org/wiki/Hash-based_message_authentication_code) digital signature, which can be used to sign encrypted data to ensure it has not been tampered with.

```c#
/// <summary>
/// Creates Hash-based Message Authentication Codes (HMACs)
/// </summary>
public interface IHmacProvider
{
    /// <summary>
    /// Gets the length of the HMAC signature in bytes
    /// </summary>
    int HmacLength { get; }

    /// <summary>
    /// Create a hmac from the given data
    /// </summary>
    /// <param name="data">Data to create hmac from</param>
    /// <returns>Hmac bytes</returns>
    byte[] GenerateHmac(string data);

    /// <summary>
    /// Create a hmac from the given data
    /// </summary>
    /// <param name="data">Data to create hmac from</param>
    /// <returns>Hmac bytes</returns>
    byte[] GenerateHmac(byte[] data);
}
```
Nancy ships with a default implementation called `DefaultHmacProvider` that uses an `IKeyGenerator` to generate a key that is then hashed using SHA-256.

## The IKeyGenerator interface

Defines the functionality for generating a key that can be used when encrypting and digitally signing data.

```c#
/// <summary>
/// Provides key byte generation
/// </summary>
public interface IKeyGenerator
{
    /// <summary>
    /// Generate a sequence of bytes
    /// </summary>
    /// <param name="count">Number of bytes to return</param>
    /// <returns>Array <see cref="count"/> bytes</returns>
    byte[] GetBytes(int count);
}
```
Nancy provides two default implementations that you can use

- `RandomKeyGenerator ` - Generates a random key, of a specified length, using the `RNGCryptoServiceProvider`
- `PassphraseKeyGenerator  ` - Uses a passphrase, static salt and optional number of iterations, together with `Rfc2898DeriveBytes` to generate a key

NOTE: If you are using the `PassphraseKeyGenerator` then it should be initialized at application startup because the algorithm is too slow to do per-request. This means that your salt will be static so the passphrase should be long and complex.

## The CryptographyConfiguration type

This is just a convenient way to store an `IEncryptionProvider` and `IHmacProvider`, so they can be passed around in code more easily. An instance of this type does not add any additional functionality.

Although it does not add any additional functionality, it offers two static properties from which you can get pre-configured instances.

These properties are

- `Default` - Uses the `RijndaelEncryptionProvider` and `DefaultHmacProvider`, both with the `RandomKeyGenerator`. This should only be used as a default `CryptographyConfiguration` in places where you accept one as a parameter but have it as optional.
- `NoEncryption` - Uses the `NoEncryption` provider and the `DefaultHmacProvider` with a `RandomKeyGenerator`


[<< Part 17. Forms authentication](Forms authentication) - [Documentation overview](Documentation)