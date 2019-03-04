---
title: Design issues for hybrid key exchange in TLS
abbrev: hybridization-tls
docname: draft-hybridization-tls-latest
date: 2019-02-28
category: info

ipr: trust200902
keyword: Internet-Draft

stand_alone: yes
pi: [toc, sortrefs, symrefs]

author:
  -
    ins: D. Stebila
    name: Douglas Steblia
    organization: University of Waterloo
    email: dstebila@uwaterloo.ca

normative:
  RFC2119:

--- abstract

TODO

--- middle

# Introduction {#introduction}

TODO

**Terminology.** For the purposes of this document, it is helpful to be able to divide algorithms into two classes:

- "Traditional" algorithms: Algorithms which are widely deployed today, but which may be deprecated in the future.  In the context of TLS 1.3 in 2019, examples including ECDH using nistp256 or Curve25519.
- "Next-generation" (or "next-gen") algorithms: Algorithms which are not yet widely deployed, but which may eventually be widely deployed.  An additional facet of these algorithms may be that we have less confidence in their security due to them being relatively new or less studied.  This includes "post-quantum" algorithms.

The primary motivation of this document is preparing for post-quantum algorithms.  However, it is possible that public key cryptography based on alternative mathematical constructions will be required independent of the advent of a quantum computer, for example because of a cryptanalytic breakthrough.  As such we opt for the more generic term "next-generation" algorithms rather than exclusively "post-quantum" algorithms.

## Scope

This document focuses on hybrid ephemeral key exchange in TLS 1.3.  

It intentionally does not address:

- Which next-generation algorithms to use in TLS 1.3, nor algorithm identifiers nor encoding mechanisms for next-generation algorithms.  (The outcomes of the NIST Post-Quantum Cryptography Standardization Project will inform this choice.)
- Authentication using next-generation algorithms.  (If a cryptographic assumption is broken due to the advent of a quantum computer or some other cryptanalytic breakthrough, confidentiality of information can be broken retroactively by any adversary who has passively recorded handshakes and encrypted communications.  But session authentication cannot be retroactively broken.)

## Related work

Experimental implementations:

- TLS 1.2
    - https://github.com/dstebila/openssl-rlwekex/tree/OpenSSL_1_0_1-stable
    - https://github.com/open-quantum-safe/openssl/tree/OQS-OpenSSL_1_0_2-stable
- TLS 1.3
    - https://github.com/open-quantum-safe/openssl/tree/OQS-OpenSSL_1_1_1-stable

Internet-Drafts:

- https://tools.ietf.org/html/draft-whyte-qsh-tls13-06
- https://tools.ietf.org/id/draft-kiefer-tls-ecdhe-sidh-00.html
- https://tools.ietf.org/html/draft-schanck-tls-additional-keyshare-00
- Amazon's SIKE and BIKE Hybrid Key Exchange Cipher Suites for Transport Layer Security (TLS)

Literature:

- Combiners
    - Robust combiners paper
    - https://eprint.iacr.org/2018/024
    - https://eprint.iacr.org/2018/903

## Requirements

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in {{RFC2119}}.

# Design considerations

## (Neg) How to negotiate hybridization and hybrid algorithms?

### (Neg-Ind) Negotiating hybrid algorithms individually

In this approach, the parties negotiate which traditional algorithm and which next-gen algorithm to use independently.  

**(Neg-Ind-1)** One approach would be for the client to advertise two lists to the server: one list containing its supported traditional mechanisms (e.g. via the existing `ClientHello` `supported_groups` extension), and a second list containing its supported next-generation mechanisms (e.g., via an additional extension).  A server could then select one algorithm for the traditional list, and one algorithm from the next-generation list.

**(Neg-Ind-2)** Another approach would be for the client to advertise a single list to the server which contains both its traditional and next-generation mechanisms (e.g., all in the existing `ClientHello` `supported_groups` extension), but with the standard providing an external mapping of those groups as either "traditional" or "next-generation".  A server could then select two algorithms from this list, one from each category.

### (Neg-Comb): Negotiating hybrid algorithms as a combination

## (Num) How many hybrid algorithms to combine?

- 2
- more than 2

## (Ctxt) How to convey public keys/ciphertexts?

- concatenate ciphertexts
- extension carrying concatenated key shares
- extension carrying extra key shares

## (Comb) How to use keys?

- concatenate keys then hash
- chain hashing (e.g. subsequent stages of TLS 1.3 key schedule)
- do you need to include the ciphertexts in the hash calculation? (More of a theoretical question, TLS does this.)

# IANA Considerations

TODO

# Security Considerations

## Preserving IND-CCA security

*Here is where we cite various pieces of literature about preserving IND-CCA security, including https://eprint.iacr.org/2018/024, https://eprint.iacr.org/2018/903*

# Acknowledgements

TODO

--- back

