---
title: "COSE Key and JSON Web Key Representation for Key Encapsulation Mechanism (KEM) of Hybrid Public Key Encryption (HPKE)"
abbrev: "COSE Key and JWK Representation for HPKE KEM"
category: std

docname: draft-ajitomi-cose-cose-key-jwk-hpke-kem-latest
submissiontype: IETF
number:
date:
consensus: true
v: 3
area: "Security"
workgroup: "CBOR Object Signing and Encryption"
venue:
  group: "CBOR Object Signing and Encryption"
  type: "Working Group"
  mail: "cose@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/cose/"
  github: "dajiaji/i-d-cose-key-jwk-hpke"
  latest: "https://dajiaji.github.io/i-d-cose-key-jwk-hpke/draft-ajitomi-cose-cose-key-jwk-hpke-kem.html"

author:
 -
    fullname: Daisuke Ajitomi
    organization: Independent
    email: "dajiaji@gmail.com"

normative:

informative:


--- abstract

This document defines an additional key parameter and a new key type for CBOR Object Signing and Encryption (COSE) Key and JSON Web Key (JWK) to represent a Key Encapsulated Mechanism (KEM) key configuration of Hybrid Public Key Encryption (HPKE).

--- middle

# Introduction

Standardized by the Internet Research Task Force (IRTF), Hybrid Public Key Encryption (HPKE) has already been adopted in several communication protocol specifications such as TLS Encrypted Client Hello (ECH), Oblivious DNS over HTTPS (ODoH) and Oblivious HTTP (OHTTP).
HPKE itself is communication protocol independent and can be widely used as a standard scheme for public key based end-to-end encryption in various applications, not only in communication protocols.

In HPKE, the sender of a ciphertext needs to know in advance not only the recipient public key, but also the HPKE mode, the KEM associated with the key, and the set of supported KDF and AEAD algorithms.
The data structure of this information (hereafter referred to as HPKE key configuration information) is defined in each communication protocol specification that uses HPKE.
For example, the ECH defines it as a structure called HpkeKeyConfig.
When using HPKE in an application, it is necessary to define the data structure corresponding to the HpkeKeyConfig and how the information is transferred from the recipient to the sender.

This document defines how to represent the HPKE KEM key configuration information in COSE_Key and JWK.
Specifically, this document defines (1) a common key parameter for defining the HPKE KEM configuration information in existing key types that can be used for key derivation and (2) a generic key type for HPKE that can also be used to represent a post-quantum KEM to be specified in the future.

The ability to include HPKE-related information in JWK, which is widely used not only as the public key representation but also as the key publication method (via the JWK Set endpoint) at the application layer, and its binary representation, COSE_Key, will facilitate the use of HPKE in a wide variety of web applications and communication systems for constrained devices.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Common Key Parameter for HPKE Key Configuration

The HPKE key configuration information is defined as a common key parameter of JWK and COSE_Key.
The parameter can be specified in the key that can be used for key derivation. In addition, the handling of existing key parameters is also defined.

## JWK Parameter

### "hkc" (HPKE Key Configuration) Parameter

The "hkc" (KPKE key configuration) parameter identifies the KEM for the recipient key and the set of KDF and AEAD algorithms supported by the recipient.
It MUST contain the object consisting of the following three attributes.
A JWK used for HPKE KEM MUST have this parameter.

- "kem": The HPKE KEM identifier, which is a two-byte value registered in the IANA HPKE registry.
- "kdfs": The array of the HPKE KDF identifiers supported by the recipient. The KDF identifier is also a two-byte value registered in the IANA HPKE registry.
- "aeads": The array of the HPKE AEAD identifiers supported by the recipient. The AEAD identifier is also a two-byte value registered in the IANA HPKE registry.

### Restrictions on the Use of Existing Key Parameters

The restrictions on the use of existing common key parameters in a JWK for HPKE KEM are as follows:

- "alg": The parameter MUST be one of the following values if specified. If omitted, it MUST be treated as "HPKE-v1-Base".
    - "HPKE-v1-Base"
    - "HPKE-v1-PSK"
    - "HPKE-v1-Auth"
    - "HPKE-v1-AuthPSK"
- "use": The parameter SHOULD NOT be specified. If specified, it MUST be "enc".
- "key_ops": The parameter SHOULD NOT be specified. If specified, it MUST include "deriveKey" and/or "deriveBits".
- etc.

## COSE Key Common Parameter

### hkc (HPKE Key Configuration) Parameter

The HPKE key configuration parameter for COSE_Key is defined as follows:

- hkc (HPKE Key Configuration): The parameter MUST contain an array structure named HPKE_Key_Configuration, which contains the same information as "hkc" in JWK above. The CDDL grammar describing the HPKE_Key_Configuration structure is:

~~~
HPKE_Key_Configuration = [
    kem: uint,              ; KEM identifier
    kdfs: uint / [+uint],   ; KDF identifiers
    aeads: uint / [+uint],  ; AEAD identifiers
]
~~~

~~~
   +---------+----------------+-------------+----------------------+
   | Name    | CBOR Type      | Value       | Description          |
   |         |                | Registry    |                      |
   +---------+----------------+-------------+----------------------+
   | kem     | uint           | HPKE KEM    | The KEM identifier   |
   |         |                | Identifiers | bound to the key     |
   |         |                |             |                      |
   | kdfs    | uint / [+uint] | HPKE KDF    | The KDF identifiers  |
   |         |                | Identifiers | supported by the     |
   |         |                |             | recipient            |
   |         |                |             |                      |
   | aeads   | uint / [+uint] | HPKE AEAD   | The AEAD identifiers |
   |         |                | Identifiers | supported by the     |
   |         |                |             | recipient            |
   |         |                |             |                      |
   +---------+----------------+-------------+----------------------+
~~~

### Restrictions on the Use of Existing Key Parameters

The restrictions on the use of existing common key parameters in a COSE_Key for the HPKE KEM are as follows:

- alg(3): The parameter MUST be one of the following values if specified. If omitted, it MUST be treated as HPKE-v1-Base(T.B.D.).
  - HPKE-v1-Base (T.B.D.)
  - HPKE-v1-PSK (T.B.D.)
  - HPKE-v1-Auth (T.B.D.)
  - HPKE-v1-AuthPSK (T.B.D.)
- key_ops(4): The parameter SHOULD NOT be specified. If specified, it MUST include "derive key"(7) and/or "derive bits"(8).
- etc.

# Generic Key Type for HPKE KEM

A generic key type for the HPKE KEM keys including a post-quantum KEM defined in the future is defined.
Even KEM keys that can be represented by existing key types can use the generic key type defined here.

## Key Type for JWK

A new generic key type (kty) value "HPKE-KEM" is defined to represent the private and public key used for the HPKE KEM.
A key with this kty has the following parameters:

- The parameter "kty" MUST be "HPKE-KEM".
- The parameter "hkc" MUST be present and contains the HPKE Key Configuration defined in Section X.X.
- The parameter "pub" MUST be present and contains the public key encoded using the base64url [RFC4648] encoding.
- The parameter "priv" MUST be present if the key is private key and contains the private key encoded using the base64url [RFC4648] encoding.

## Key Type for COSE_Key

A new generic kty(1) value HPKE-KEM(T.B.D.) is defined to represent the private and public key used for the HPKE KEM.
A key with this kty has the following parameters:

- The parameter kty(1) MUST be HPKE-KEM(T.B.D).
- The parameter hkc(T.B.D.) MUST be present and contains the HPKE Key Configuration defined in Section X.X.
- The parameter pub(-1) MUST be present and contains the public key encoded using the base64url [RFC4648] encoding.
- The parameter priv(-2) MUST be present if the key is private key and contains the private key encoded using the base64url [RFC4648] encoding.

# Security Considerations

TODO


# IANA Considerations

TODO


--- back

# Examples

## JWK for DHKEM(X25519, KDF-SHA-256) Public Key with Key Type "OKP"

~~~
{
    "kty": "OKP",
    "kid": "01",
    "crv": "X25519",
    "alg": "HPKE-v1-Base",
    "hkc": {
        "kem": 0x020,
        "kdfs": [0x001, 0x002, 0x003],
        "kems": [0x001, 0x002]
    },
    "x": "y3wJq3uXPHeoCO4FubvTc7VcBuqpvUrSvU6ZMbHDTCI"
}
~~~

## JWK for DHKEM(X448, KDF-SHA-512) Private Key with Key Type "HPKE-KEM"

~~~
{
    "kty": "HPKE-KEM",
    "kid": "01",
    "alg": "HPKE-v1-Base",
    "hkc": {
        "kem": 0x021,
        "kdfs": [0x001, 0x002, 0x003],
        "kems": [0x001, 0x002]
    },
    "pub": "IkLmc0klvEMXYneHMKAB6ePohryAwAPVe2pRSffIDY6NrjeYNWVX5J-fG4NV2OoU77C88A0mvxI",
    "priv": "rJJRG3nshyCtd9CgXld8aNaB9YXKR0UOi7zj7hApg9YH4XdBO0G8NcAFNz_uPH2GnCZVcSDgV5c"
}
~~~

## COSE_Key for DHKEM(X25519, KDF-SHA-256) Public Key with Key Type OKP(1)

~~~
  {
    1:1,          // OKP
    2:'01',
    3:-1(T.B.D),  // HPKE-v1-Base
    -1:4,         // X25519
    6(T.B.D): [   // hkc (HPKE Key Configuration)
        0x0020,
        [0x0001, 0x0002, 0x0003],
        [0x0001, 0x0002]
    ],
    -2:h'd75a980182b10ab7d54bfed3c964073a0ee172f3daa62325af021a68f707511a'
  }
~~~

## COSE_Key for DHKEM(X448, KDF-SHA-512) Private Key with Key Type HPKE-KEM(T.B.D)

~~~
  {
    1:-1(T.B.D.),  // HPKE-KEM
    2:'01',
    3:-1(T.B.D),   // HPKE-v1-Base
    6(T.B.D): [    // hkc (HPKE Key Configuration)
        0x0021,                    // KEM id
        [0x0001, 0x0002, 0x0003],  // supported KDF ids
        [0x0001, 0x0002]           // supported AEAD ids
    ],
    -1:h'5fd7449b59b461fd2ce787ec616ad46a1da1342485a70e1f8a0ea75d80e96778edf124769b46c7061bd6783df1e50f6cd1fa1abeafe8256180',
    -2:h'6c82a562cb808d10d632be89c8513ebf6c929f34ddfa8c9f63c9960ef6e348a3528c8a3fcc2f044e39a3fc5b94492f8f032e7549a20098f95b'
  }
~~~

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
