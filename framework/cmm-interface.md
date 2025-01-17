[//]: # "Copyright Amazon.com Inc. or its affiliates. All Rights Reserved."
[//]: # "SPDX-License-Identifier: CC-BY-SA-4.0"

# Cryptographic Materials Manager Interface

## Version

0.3.0

### Changelog

- 0.3.0

  - Clarify handling of the `aws-crypto-public-key` encryption context key.

- 0.2.0

  - [Remove Keyring Trace](../changes/2020-05-13_remove-keyring-trace/change.md)

- 0.1.0-preview

  - Initial record

## Implementations

| Language   | Confirmed Compatible with Spec Version | Minimum Version Confirmed | Implementation                                                                                                                                                  |
| ---------- | -------------------------------------- | ------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| C          | 0.1.0-preview                          | 0.1.0                     | [materials.h](https://github.com/aws/aws-encryption-sdk-c/blob/master/include/aws/cryptosdk/materials.h)                                                        |
| Javascript | 0.1.0-preview                          | 0.1.0                     | [materials_manager.ts](https://github.com/awslabs/aws-encryption-sdk-javascript/blob/master/modules/material-management/src/materials_manager.ts)               |
| Python     | 0.1.0-preview                          | 1.3.0                     | [materials_managers](https://github.com/aws/aws-encryption-sdk-python/blob/master/src/aws_encryption_sdk/materials_managers/__init__.py)                        |
| Java       | 0.1.0-preview                          | 1.3.0                     | [CryptoMaterialsManager.java](https://github.com/aws/aws-encryption-sdk-java/blob/master/src/main/java/com/amazonaws/encryptionsdk/CryptoMaterialsManager.java) |

## Overview

The Cryptographic Materials Manager (CMM) assembles the cryptographic materials used to encrypt the [message](../data-format/message.md) and decrypt the encrypted messages.
The CMM interface describes the interface that all CMMs MUST implement.

## Definitions

### Conventions used in this document

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL"
in this document are to be interpreted as described in [RFC 2119](https://tools.ietf.org/html/rfc2119).

## Supported CMMs

The AWS Encryption SDK provides the following built-in CMM types:

- [Default CMM](default-cmm.md)
- [Caching CMM](caching-cmm.md)

Note: A user MAY create their own custom CMM.

## Interface

### Inputs

The inputs to the CMM are groups of related fields, referred to as:

- [Encryption Materials Request](#encryption-materials-request)
- [Decrypt Materials Request](#decrypt-materials-request)

#### Encryption Materials Request

This is the input to the [get encryption materials](#get-encryption-materials) behavior.

The encryption materials request MUST include the following:

- [Encryption Context](structures.md#encryption-context)
  - The encryption context provided MAY be empty.
- [Commitment Policy](../client-apis/client.md#commitment-policy)

The encryption request MAY include the following:

- [Algorithm Suite](algorithm-suites.md)
- Max Plaintext Length
  - This value represents the maximum length of the plaintext to be encrypted
    using the returned materials.
    The length of the plaintext to be encrypted MUST not be larger than this value.

#### Decrypt Materials Request

This is the input to the [decrypt materials](#decrypt-materials) behavior.

The decrypt materials request MUST include the following:

- [Algorithm Suite](algorithm-suites.md)
- [Commitment Policy](../client-apis/client.md#commitment-policy)
- [Encrypted Data Keys](structures.md#encrypted-data-keys)
- [Encryption Context](structures.md#encryption-context)
  - The encryption context provided MAY be empty.

### Behaviors

The CMM Interface MUST support the following behaviors:

- [Get Encryption Materials](#get-encryption-materials)
- [Decrypt Materials](#decrypt-materials)

#### Get Encryption Materials

When the CMM gets an [encryption materials request](#encryption-materials-request),
it MUST return [encryption materials](structures.md#encryption-materials) appropriate for the request.

The encryption materials returned MUST include the following:

- [Algorithm Suite](algorithm-suites.md)
  - If the encryption materials request contains an algorithm suite, the encryption materials returned SHOULD contain the same algorithm suite.
- Plaintext Data Key
- [Encrypted Data Keys](structures.md#encrypted-data-keys)
  - Every encrypted data key in this list MUST correspond to the above plaintext data key.
- [Encryption Context](structures.md#encryption-context)
  - The CMM MAY modify the encryption context.

If the algorithm suite contains a [signing algorithm](algorithm-suites.md#signature-algorithm):

- The CMM MUST include a [signing key](structures.md#signing-key).
- The CMM SHOULD also add a key-value pair using the reserved key `aws-crypto-public-key` to the encryption context.
  If it does, the mapped value SHOULD be the signature verification key corresponding to the signing key.

If the algorithm suite does not contain a [signing algorithm](algorithm-suites.md#signature-algorithm):

- The CMM SHOULD NOT add a key-value pair using the reserved key `aws-crypto-public-key` to the encryption context.

The CMM MUST ensure that the encryption materials returned are valid.

- The encryption materials returned MUST follow the specification for [encryption-materials](structures.md#encryption-materials).
- The value of the plaintext data key MUST be non-NULL.
- The plaintext data key length MUST be equal to the [key derivation input length](algorithm-suites.md#key-derivation-input-length).
- The encrypted data keys list MUST contain at least one encrypted data key.
- If the algorithm suite contains a signing algorithm, the encryption materials returned MUST include the generated signing key.

#### Decrypt Materials

When the CMM gets a [decrypt materials request](#decrypt-materials-request),
it MUST return [decryption materials](structures.md#decryption-materials) appropriate for the request.

If the requested algorithm suite does not include a signing algorithm but the encryption context includes the reserved `aws-crypto-public-key` key, the operation SHOULD fail.
Likewise, if the requested algorithm suite includes a signing algorithm but the encryption context does not include the reserved `aws-crypto-public-key` key, the operation SHOULD fail.

The decryption materials returned MUST include the following:

- Plaintext Data Key
- [Encryption Context](structures.md#encryption-context)
  - The CMM MAY modify the encryption context.
  - The operations made on the encryption context on the Get Encryption Materials call SHOULD be inverted on the Decrypt Materials call.
- [Algorithm Suite](algorithm-suites.md)
  - If the decrypt materials request contains an algorithm suite, the decryption materials returned SHOULD contain the same algorithm suite.

If the algorithm suite obtained from the decryption request contains a [signing algorithm](algorithm-suites.md#signature-algorithm),
the decryption materials MUST include the [signature verification key](structures.md#verification-key).

The CMM MUST ensure that the decryption materials returned are valid.

- The decryption materials returned MUST follow the specification for [decryption-materials](structures.md#decryption-materials).
- The value of the plaintext data key MUST be non-NULL.
- The plaintext data key returned MUST correspond with at least one of the encrypted data keys.
  - The is typically done by constructing a CMM that uses keyrings/master keys.

## Customization

The CMM is an ideal point for customization and extension.

Example scenarios include:

- Interacting with other CMMs
- Using [Keyring(s)](keyring-interface.md)
- Modifying the encryption context
- Managing the signing/verification keys
- Data key Caching
- Providing support for policy enforcement
