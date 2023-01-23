---
title: "COSE Key and JSON Web Key Representation for Key Encapsulation Mechanism (KEM) of Hybrid Public Key Encryption (HPKE)"
abbrev: "COSE Key and JWK Representation for HPKE KEM"
category: std

docname: draft-ajitomi-cose-key-jwk-hpke-latest
submissiontype: IETF  # also: "independent", "IAB", or "IRTF"
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
  latest: "https://dajiaji.github.io/i-d-cose-key-jwk-hpke/draft-ajitomi-cose-key-jwk-hpke.html"

author:
 -
    fullname: Daisuke Ajitomi
    organization: Independent
    email: "ajitomi@gmail.com"

normative:

informative:


--- abstract

This document defines an additional key parameter and a new key type for CBOR Object Signing and Encryption (COSE) Key and JSON Web Key (JWK) to represent a Key Encapsulated Mechanism (KEM) key configuration of Hybrid Public Key Encryption (HPKE).

--- middle

# Introduction

Standardized by the Internet Research Task Force (IRTF), HPKE has already been adopted in several communication protocol specifications such as Encrypted Client Hello (ECH), Oblivious DNS over HTTP (ODoH) and Oblivious HTTP (OHTTP). The HPKE itself is communication protocol independent and can be widely used as a standard scheme for public key based end-to-end encryption in various applications, not only in communication protocols.

In HPKE, the sender sending a ciphertext needs to know in advance the KEM supported by the recipient and its public key, as well as the HPKE mode, the set of supported KDF and AEAD algorithms. For example, ECH defines this information as a structure called HpkeKeyConfig. The way in which this information is exchanged is out-of-scope of the specification, but it is assumed that it is obtained in advance by the sender, for example via DNS HTTPS RRs.

This document defines how to represent the HPKE KEM key configuration information corresponding to the ECH HpkeKeyConfig in COSE_Key and JWK. Specifically, this document defines (1) a key parameter for defining the HPKE KEM configuration information in existing key types that can be used for key derivation and (2) a generic key type for HPKE that can also be used to represent a post-quantum KEM to be specified in the future.

The ability to include HPKE-related information in JWKs, which are widely used as public key representations at the application layer, and their binary representation, COSE_Key, will facilitate the use of HPKE in a wide variety of web applications and communication systems for constrained devices.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Common Key Parameter for HPKE KEM

The KEM key configuration information is defined as a common key parameter of JWK and COSE_Key. The parameter can be specified in the key that can be used for key derivation. In addition, the handling of existing common key parameters is also defined.

## JWK Parameter

The KEM key configuration parameter for JWK is defined as follows:

- "hkc" (HPKE Key Configuration): The parameter MUST contain the object consisting of the following three attributes. A JWK used for HPKE KEM MUST have this parameter.
  - "kem": The HPKE KEM identifier, which is a two-byte value registered in the IANA HPKE registry.
  - "kdfs": The array of the HPKE KDF identifiers supported by the recipient. The KDF identifier is also a two-byte value registered in the IANA HPKE registry.
  - "aeads": The array of the HPKE AEAD identifiers supported by the recipient. The AEAD identifier is also a two-byte value registered in the IANA HPKE registry.

The restrictions on the use of existing common key parameters in a JWK for the HPKE KEM are as follows:

- "alg": The parameter MUST be one of the following values if specified. If omitted, it MUST be treated as "HPKE-v1-Base".
    - "HPKE-v1-Base"
    - "HPKE-v1-PSK"
    - "HPKE-v1-Auth"
    - "HPKE-v1-AuthPSK"
- "use": The parameter SHOULD NOT be specified. If specified, it MUST be "enc".
- "key_ops": The parameter SHOULD NOT be specified. If specified, it MUST include "deriveKey" and/or "deriveBits".
- etc.

## COSE Key Common Parameter

The KEM key configuration parameter for COSE_Key is defined as follows:

- hkc (HPKE Key Configuration): The parameter MUST contain an array structure named HPKE_Key_COnfiguration, which contains the same information as "hkc" in JWK above. The CDDL grammar describing the HPKE_Key_Configuration structure is:

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

## Key Type "HPKE-KEM" for JWK

A new generic key type (kty) value "HPKE-KEM" is defined to represent the private and public key used for the HPKE KEM.
A key with this kty has the following parameters:

- The parameter "kty" MUST be "HPKE-KEM".
- The parameter "hkc" MUST be present and contains the HPKE Key Configuration defined in Section X.X.
- The parameter "pub" MUST be present and contains the public key encoded using the base64url [RFC4648] encoding.
- The parameter "priv" MUST be present if the key is private key and contains the private key encoded using the base64url [RFC4648] encoding.

## Key Type HPKE-KEM(T.B.D.) for COSE_Key

A new generic kty(1) value HPKE-KEM(T.B.D.) is defined to represent the private and public key used for the HPKE KEM.
A key with this kty has the following parameters:

- The parameter kty(1) MUST be HPKE-KEM(T.B.D).
- The parameter hkc(T.B.D.) MUST be present and contains the HPKE Key Configuration defined in Section X.X.
- The parameter pub(T.B.D.) MUST be present and contains the public key encoded using the base64url [RFC4648] encoding.
- The parameter priv(T.B.D.) MUST be present if the key is private key and contains the private key encoded using the base64url [RFC4648] encoding.

# Security Considerations

TODO


# IANA Considerations

TODO


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
