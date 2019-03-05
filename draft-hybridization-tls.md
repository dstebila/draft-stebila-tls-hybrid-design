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
  TLS13: RFC8446

informative:
  EXTERN-PSK: I-D.ietf-tls-tls13-cert-with-extern-psk
  OQS-111:
    target: https://github.com/open-quantum-safe/openssl/tree/OQS-OpenSSL_1_1_1-stable
    title: "OQS-OpenSSL-1-1-1_stable"
    author:
      org: Open Quantum Safe Project
    date: 2018-11
  SCHANCK: I-D.schanck-tls-additional-keyshare
  WHYTE: I-D.whyte-qsh-tls13

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

This document focuses on hybrid ephemeral key exchange in TLS 1.3 {{TLS13}}.

It intentionally does not address:

- Which next-generation algorithms to use in TLS 1.3, nor algorithm identifiers nor encoding mechanisms for next-generation algorithms.  (The outcomes of the NIST Post-Quantum Cryptography Standardization Project will inform this choice.)
- Authentication using next-generation algorithms.  (If a cryptographic assumption is broken due to the advent of a quantum computer or some other cryptanalytic breakthrough, confidentiality of information can be broken retroactively by any adversary who has passively recorded handshakes and encrypted communications.  But session authentication cannot be retroactively broken.)

## Related work

Experimental implementations:

- TLS 1.2
    - https://github.com/dstebila/openssl-rlwekex/tree/OpenSSL_1_0_1-stable
    - https://github.com/open-quantum-safe/openssl/tree/OQS-OpenSSL_1_0_2-stable
- TLS 1.3
    - {{OQS-111}}
    - Google CECPQ1
    - Google CECPQ2?

Internet-Drafts:

- {{WHYTE}}
- https://tools.ietf.org/id/draft-kiefer-tls-ecdhe-sidh-00.html
- {{SCHANCK}}
- Amazon's SIKE and BIKE Hybrid Key Exchange Cipher Suites for Transport Layer Security (TLS)

Literature:

- Combiners
    - Robust combiners paper
    - https://eprint.iacr.org/2018/024
    - https://eprint.iacr.org/2018/903

TLS working group document on adding an external PSK - {{EXTERN-PSK}}

Also IKE/IPsec - Scott Fluhrer

## Requirements

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in {{RFC2119}}.

# Design considerations

## (Neg) How to negotiate hybridization and hybrid algorithms?

### Key exchange negotiation in TLS 1.3

Recall that in TLS 1.3, the key exchange mechanism is negotiated via the `supported_groups` extension.  The `NamedGroup` enum is a list of standardized groups for Diffie--Hellman key exchange, such as `secp256r1`, `x25519`, and `ffdhe2048`.  

The client, in its `ClientHello` message, lists its supported mechanisms in the `supported_groups` extension.  The client also optionally includes the public key of one of these groups in the `key_share` extension as a guess of which mechanism the server might accept in hopes of reducing the number of round trips.  

If the server is willing to use one of the client's requested mechanisms, it responds with a `key_share` extension containing its public key for the desired mechanism.

If the server is not willing to use any of the client's requested mechanisms, the server responds with a `HelloRetryRequest` message that includes an extension indicating its preferred mechanism.

### (Neg-Ind) Negotiating hybrid algorithms individually

In these two approaches, the parties negotiate which traditional algorithm and which next-gen algorithm to use independently.  The `NamedGroup` enum is extended to include algorithm identifiers for each next-gen algorithm.

**(Neg-Ind-1)** The client advertises two lists to the server: one list containing its supported traditional mechanisms (e.g. via the existing `ClientHello` `supported_groups` extension), and a second list containing its supported next-generation mechanisms (e.g., via an additional `ClientHello` extension).  A server could then select one algorithm from the traditional list, and one algorithm from the next-generation list.  (This is the approach in {{SCHANCK}}.)

**(Neg-Ind-2)** The client advertises a single list to the server which contains both its traditional and next-generation mechanisms (e.g., all in the existing `ClientHello` `supported_groups` extension), but with some external table provides a standardized mapping of those mechanisms as either "traditional" or "next-generation".  A server could then select two algorithms from this list, one from each category.

### (Neg-Comb): Negotiating hybrid algorithms as a combination

In these two approaches, combinations of key exchange mechanisms appear as a single monolithic block; the parties negotiate which of several combinations they wish to use.

**(Neg-Comb-1)** The `NamedGroup` enum is extended to include algorithm identifiers for each **combination** of algorithms desired by the working group.  There is no "internal structure" to the algorithm identifiers for each combination, they are simply new code points assigned arbitrarily.  The client includes any desired combinations in its `ClientHello` `supported_groups` list, and the server picks one of these.  This is the approach in {{OQS-111}}.

**(Neg-Comb-2)** The `NamedGroup` enum is extended to include algorithm identifiers for each next-gen algorithm.  Some additional field/extension is used to convey which combinations the parties wish to use.  For example, in {{WHYTE}}, there are distinguished `NamedGroup` called `hybrid_marker 0`, `hybrid_marker 1`, `hybrid_marker 2`, etc.  This is complemented by a `HybridExtension` which contains mappings for each numbered `hybrid_marker` to the set of key exchange algorithms (2 or more) that comprise that proposed combination.

### Benefits and Drawbacks

**Combinatorial explosion.** (Neg-Comb-1) requires new identifiers to be defined for each desired combination.  The other 3 options in this section do not.

**Extensions.** (Neg-Comb-1) does not require any new extensions to be defined.  (Neg-Ind-2) also does not require any new extensions to be defined, but does change the processing logic of an existing extension.  (Neg-Ind-1) and (Neg-Comb-2) do require new extensions.

**0-RTT.**

## (Num) How many hybrid algorithms to combine?

- 2
- more than 2

## (Ctxt) How to convey public keys/ciphertexts?

- concatenate ciphertexts
- extension carrying concatenated key shares
- extension carrying extra key shares

## (Comb) How to use keys?

- concatenate keys then hash (benefit: minimal changes)
- chain hashing (e.g. subsequent stages of TLS 1.3 key schedule) (benefit: modularity)
- do you need to include the ciphertexts in the hash calculation? (More of a theoretical question, TLS does this.)

# Candidate instantiations

*Select one or two combinations of the choices above*

# Other TLS considerations

- Chaining properties during resumption: does the ephemeral key exchange in a resumed session need to use the same algorithms as in the initial one?

- 0-RTT

- Dealing with failures

# IANA Considerations

TODO

# Security Considerations

## Is IND-CCA required?

## Preserving IND-CCA security

*Here is where we cite various pieces of literature about preserving IND-CCA security, including https://eprint.iacr.org/2018/024, https://eprint.iacr.org/2018/903*

# Acknowledgements

TODO

--- back

