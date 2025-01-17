[//]: # "Copyright Amazon.com Inc. or its affiliates. All Rights Reserved."
[//]: # "SPDX-License-Identifier: CC-BY-SA-4.0"

# Cryptographic Materials Cache Interface

## Version

0.3.0

### Changelog

- 0.3.0
  - [Return Nothing from Put Cache Entry in Cryptographic Materials Cache](../changes/2020-07-20_put-cache-entry-returns-nothing/change.md)
- 0.2.0
  - [Refactor Cryptographic Materials Cache Specification](../changes/2020-07-14_refactor-cmc-spec/change.md)
- 0.1.0-preview
  - Initial record

## Implementations

| Language   | Confirmed Compatible with Spec Version | Minimum Version Confirmed | Implementation                                                                                                                                                       |
| ---------- | -------------------------------------- | ------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| C          | 0.1.0-preview                          | 0.1.0                     | [cache.h](https://github.com/aws/aws-encryption-sdk-c/blob/master/include/aws/cryptosdk/cache.h)                                                                     |
| Javascript | 0.1.0-preview                          | 0.1.0                     | [cryptographic_materials_cache.ts](https://github.com/awslabs/aws-encryption-sdk-javascript/blob/master/modules/cache-material/src/cryptographic_materials_cache.ts) |
| Python     | 0.1.0-preview                          | 1.3.0                     | [caches/base.py](https://github.com/aws/aws-encryption-sdk-python/blob/master/src/aws_encryption_sdk/caches/base.py)                                                 |
| Java       | 0.1.0-preview                          | 1.3.0                     | [CryptoMaterialsCache.java](https://github.com/aws/aws-encryption-sdk-java/blob/master/src/main/java/com/amazonaws/encryptionsdk/caching/CryptoMaterialsCache.java)  |

## Overview

Cryptographic materials cache (CMC) is used by the [caching cryptographic materials manager (CMM)](caching-cmm.md)
to store cryptographic materials for reuse.
This document describes the interface that all CMCs MUST implement.

## Definitions

### Conventions used in this document

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL"
in this document are to be interpreted as described in [RFC 2119](https://tools.ietf.org/html/rfc2119).

## Concepts

### Cache Identifier

The cache identifier used to uniquely identify a single cache entry in the cryptographic materials cache.

### Cache Entry

A cache entry represents an entry in the cryptographic materials cache.
A cache entry contains the cryptographic materials, [encryption](structures.md#encryption-materials) or [decryption](structures.md#decryption-materials)
materials to be cached along with other metadata.
Some implementations of the CMC have the following metadata:

- [Creation Time](#creation-time)
- [Expiry Time](#expiry-time)
- [Usage Metadata](#usage-metadata)

#### Creation Time

Timestamp when the cache entry was created.

#### Expiry Time

Timestamp when the cache entry expires.

#### Usage Metadata

The usage metadata is of the following two types:

- [Messages Usage](#message-usage)
- [Bytes Usage](#bytes-usage)

##### Message Usage

The number of messages encrypted by the [encryption](structures.md#encryption-materials) materials cached in this cache entry.

##### Bytes Usage

The number of bytes encrypted by the [encryption](structures.md#encryption-materials) materials cached in this cache entry.

## Supported CMCs

The AWS Encryption SDK provides a built-in [local cryptographic materials cache](local-cryptographic-materials-cache.md) (local CMC).
The local CMC is a configurable, in-memory, least recently used (LRU) cache.
It provides non-blocking, locking, [cache entries](#cache-entry) per [cache identifier](#cache-identifier).

## Behaviors

The Cryptographic Materials Cache provides behaviours for putting cache entries, getting cache entries and deleting cache entries.

### Put Cache Entry

Attempts to put a cache entry for the specified cache ID.
If a cache entry for the given cache ID does not exists in the cache, the CMC creates a new cache entry.
This operation MUST NOT return the inserted cache entry.

### Get Cache Entry

Attempts to get a cache entry for the specified cache ID.
The CMC MUST validate that the cache entry is valid prior to returning the cache entry.
A successful call to Get Entry returns the [cache entry](#cache-entry) and an unsuccessful call returns a cache miss.

### Delete Cache Entry

Attempts to delete a cache entry from the CMC.
