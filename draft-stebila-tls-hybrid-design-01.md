---
title: Design issues for hybrid key exchange in TLS 1.3
abbrev: stebila-tls-hybrid-design
docname: draft-stebila-tls-hybrid-design-01
date: 2019-07-08
category: info

ipr: trust200902
keyword: Internet-Draft

stand_alone: yes
pi: [toc, sortrefs, symrefs]

author:
  -
    ins: D. Stebila
    name: Douglas Stebila
    organization: University of Waterloo
    email: dstebila@uwaterloo.ca
  -
    ins: S. Fluhrer
    name: Scott Fluhrer
    organization: Cisco Systems
    email: sfluhrer@cisco.com
  -
    ins: S. Gueron
    name: Shay Gueron
    organization: University of Haifa and Amazon Web Services
    abbrev: U. Haifa, Amazon Web Services
    email: shay.gueron@gmail.com

normative:
  TLS13: RFC8446

informative:
  BCNS15: DOI.10.1109/SP.2015.40
  BERNSTEIN: DOI.10.1007/978-3-540-88702-7
  BINDEL:
    target: https://eprint.iacr.org/2018/903
    title: Hybrid Key Encapsulation Mechanisms and Authenticated Key Exchange
    author:
      -
        ins: N. Bindel
      -
        ins: J. Brendel
      -
        ins: M. Fischlin
      -
        ins: B. Goncalves
      -
        ins: D. Stebila
    seriesinfo: Post-Quantum Cryptography (PQCrypto)
    date: 2019
  CECPQ1:
    target: https://security.googleblog.com/2016/07/experimenting-with-post-quantum.html
    title: Experimenting with Post-Quantum Cryptography
    author:
      -
        ins: M. Braithwaite
    date: 2016-07-07
  CECPQ2:
    target: https://www.imperialviolet.org/2018/12/12/cecpq2.html
    title: CECPQ2
    author:
      -
        ins: A. Langley
    date: 2018-12-12
  DFGS15: DOI.10.1145/2810103.2813653
  DODIS: DOI.10.1007/978-3-540-30576-7_11
  DOWLING: DOI.10.5204/thesis.eprints.108960
  ETSI:
    target: https://www.etsi.org/images/files/ETSIWhitePapers/QuantumSafeWhitepaper.pdf
    title: "Quantum safe cryptography and security: An introduction, benefits, enablers and challengers"
    author:
      -
        role: editor
        ins: M. Campagna
      -
        ins: others
    seriesinfo: ETSI White Paper No. 8
    date: 2015-06
  EVEN: DOI.10.1007/978-1-4684-4730-9_4
  EXTERN-PSK: I-D.ietf-tls-tls13-cert-with-extern-psk
  FRODO: DOI.10.1145/2976749.2978425
  GIACON: DOI.10.1007/978-3-319-76578-5_7
  HARNIK: DOI.10.1007/11426639_6
  HOFFMAN: I-D.hoffman-c2pq
  IKE-HYBRID: I-D.tjhai-ipsecme-hybrid-qske-ikev2
  IKE-PSK: I-D.ietf-ipsecme-qr-ikev2
  KIEFER: I-D.kiefer-tls-ecdhe-sidh
  KPW13: DOI.10.1007/978-3-642-40041-4_24
  LANGLEY:
    target: https://www.imperialviolet.org/2018/04/11/pqconftls.html
    title: Post-quantum confidentiality for TLS
    author:
      -
        ins: A. Langley
    date: 2018-04-11
  NIELSEN:
    title: Quantum Computation and Quantum Information
    author:
      -
        ins: M. A. Nielsen
      -
        ins: I. L. Chuang
    seriesinfo: Cambridge University Press
    date: 2000
  NIST:
    target: https://www.nist.gov/pqcrypto
    title: Post-Quantum Cryptography
    author:
      org: National Institute of Standards and Technology (NIST)
  NIST-SP-800-56C:
    target: https://doi.org/10.6028/NIST.SP.800-56Cr1
    title: Recommendation for Key-Derivation Methods in Key-Establishment Schemes
    author:
      org: National Institute of Standards and Technology (NIST)
    date: 2018-04
  NIST-SP-800-135:
    target: https://doi.org/10.6028/NIST.SP.800-135r1
    title: Recommendation for Existing Application-Specific Key Derivation Functions
    author:
      org: National Institute of Standards and Technology (NIST)
    date: 2011-12
  OQS-102:
    target: https://github.com/open-quantum-safe/openssl/tree/OQS-OpenSSL_1_0_2-stable
    title: OQS-OpenSSL-1-0-2_stable
    author:
      org: Open Quantum Safe Project
    date: 2018-11
  OQS-111:
    target: https://github.com/open-quantum-safe/openssl/tree/OQS-OpenSSL_1_1_1-stable
    title: OQS-OpenSSL-1-1-1_stable
    author:
      org: Open Quantum Safe Project
    date: 2018-11
  SCHANCK: I-D.schanck-tls-additional-keyshare
  WHYTE12: I-D.whyte-qsh-tls12
  WHYTE13: I-D.whyte-qsh-tls13
  XMSS: RFC8391
  ZHANG: DOI.10.1007/978-3-540-24632-9_26

--- abstract

Hybrid key exchange refers to using multiple key exchange algorithms simultaneously and combining the result with the goal of providing security even if all but one of the component algorithms is broken, and is motivated by transition to post-quantum cryptography.  This document categorizes various design considerations for using hybrid key exchange in the Transport Layer Security (TLS) protocol version 1.3 and outlines two concrete instantiations for consideration.

--- middle

# Introduction {#introduction}

This document categorizes various design decisions one could make when implementing hybrid key exchange in TLS 1.3, with the goal of fostering discussion, providing options for short-term prototypes/experiments, and serving as a basis for eventual standardization.  This document also includes two concrete instantiations for consideration, following two different approaches; it is not our intention that both be standardized.

This document does not propose specific post-quantum mechanisms; see {{scope}} for more on the scope of this document.

Comments are solicited and should be addressed to the TLS working group mailing list at tls@ietf.org and/or the author(s).

## Revision history

- draft-00: Initial version.
- draft-01:
    - Add [(Comb-KDF-1)](#comb-kdf-1) and [(Comb-KDF-2)](#comb-kdf-2) options.
    - Add [Candidate Instantiation 1](#candidate-1).
    - Add [Candidate Instantiation 2](#candidate-2).

## Terminology {#terminology}

For the purposes of this document, it is helpful to be able to divide cryptographic algorithms into two classes:

- "Traditional" algorithms: Algorithms which are widely deployed today, but which may be deprecated in the future.  In the context of TLS 1.3 in 2019, examples of traditional key exchange algorithms include elliptic curve Diffie--Hellman using secp256r1 or x25519, or finite-field Diffie--Hellman.
- "Next-generation" (or "next-gen") algorithms: Algorithms which are not yet widely deployed, but which may eventually be widely deployed.  An additional facet of these algorithms may be that we have less confidence in their security due to them being relatively new or less studied.  This includes "post-quantum" algorithms.

"Hybrid" key exchange, in this context, means the use of two (or more) key exchange mechanisms based on different cryptographic assumptions (for example, one traditional algorithm and one next-gen algorithm), with the purpose of the final session key being secure as long as at least one of the component key exchange mechanisms remains unbroken.  We use the term "component" algorithms to refer to the algorithms that are being combined in a hybrid key exchange.

The primary motivation of this document is preparing for post-quantum algorithms.  However, it is possible that public key cryptography based on alternative mathematical constructions will be required independent of the advent of a quantum computer, for example because of a cryptanalytic breakthrough.  As such we opt for the more generic term "next-generation" algorithms rather than exclusively "post-quantum" algorithms.

## Motivation for use of hybrid key exchange {#motivation}

Ideally, one would not use hybrid key exchange: one would have confidence in a single algorithm and parameterization that will stand the test of time.  However, this may not be the case in the face of quantum computers and cryptanalytic advances more generally.  

Many (but not all) of the post-quantum algorithms currently under consideration are relatively new; they have not been subject to the same depth of study as RSA and finite-field / elliptic curve Diffie--Hellman, and thus we do not necessarily have as much confidence in their fundamental security, or the concrete security level of specific parameterizations.  

Early adopters eager for post-quantum security may want to use hybrid key exchange to have the potential of post-quantum security from a less-well-studied algorithm while still retaining at least the security currently offered by traditional algorithms.  (They may even need to retain traditional algorithms due to regulatory constraints, for example FIPS compliance.)  

Moreover, it is possible that even by the end of the NIST Post-Quantum Cryptography Standardization Project, and for a period of time thereafter, conservative users may not have full confidence in some algorithms.

As such, there may be users for whom hybrid key exchange is an appropriate step prior to an eventual transition to next-generation algorithms.

## Scope {#scope}

This document focuses on hybrid ephemeral key exchange in TLS 1.3 {{TLS13}}.  It intentionally does not address:

- Selecting which next-generation algorithms to use in TLS 1.3, nor algorithm identifiers nor encoding mechanisms for next-generation algorithms.  (The outcomes of the NIST Post-Quantum Cryptography Standardization Project {{NIST}} will inform this choice.)
- Authentication using next-generation algorithms.  (If a cryptographic assumption is broken due to the advent of a quantum computer or some other cryptanalytic breakthrough, confidentiality of information can be broken retroactively by any adversary who has passively recorded handshakes and encrypted communications.  But session authentication cannot be retroactively broken.)

## Goals {#goals}

The primary goal of a hybrid key exchange mechanism is to facilitate the establishment of a shared secret which remains secure as long as as one of the component key exchange mechanisms remains unbroken.

In addition to the primary cryptographic goal, there may be several additional goals in the context of TLS 1.3:

- **Backwards compatibility:** Clients and servers who are "hybrid-aware", i.e., compliant with whatever hybrid key exchange standard is developed for TLS, should remain compatible with endpoints and middle-boxes that are not hybrid-aware.  The three scenarios to consider are:
    1. Hybrid-aware client, hybrid-aware server: These parties should establish a hybrid shared secret.
    2. Hybrid-aware client, non-hybrid-aware server:  These parties should establish a traditional shared secret (assuming the hybrid-aware client is willing to downgrade to traditional-only).
    3. Non-hybrid-aware client, hybrid-aware server:  These parties should establish a traditional shared secret (assuming the hybrid-aware server is willing to downgrade to traditional-only).

    Ideally backwards compatibility should be achieved without extra round trips and without sending duplicate information; see below.

- **High performance:** Use of hybrid key exchange should not be prohibitively expensive in terms of computational performance.  In general this will depend on the performance characteristics of the specific cryptographic algorithms used, and as such is outside the scope of this document.  See {{BCNS15}}, {{CECPQ1}}, {{FRODO}} for preliminary results about performance characteristics.

- **Low latency:** Use of hybrid key exchange should not substantially increase the latency experienced to establish a connection.  Factors affecting this may include the following. 
    - The computational performance characteristics of the specific algorithms used.  See above.
    - The size of messages to be transmitted.  Public key / ciphertext sizes for post-quantum algorithms range from hundreds of bytes to over one hundred kilobytes, so this impact can be substantially.  See {{BCNS15}}, {{FRODO}} for preliminary results in a laboratory setting, and {{LANGLEY}} for preliminary results on more realistic networks.
    - Additional round trips added to the protocol.  See below.

- **No extra round trips:** Attempting to negotiate hybrid key exchange should not lead to extra round trips in any of the three hybrid-aware/non-hybrid-aware scenarios listed above.  

- **No duplicate information:** Attempting to negotiate hybrid key exchange should not mean having to send multiple public keys of the same type.

## Related work {#related-work}

Quantum computing and post-quantum cryptography in general are outside the scope of this document.  For a general introduction to quantum computing, see a standard textbook such as {{NIELSEN}}.  For an overview of post-quantum cryptography as of 2009, see {{BERNSTEIN}}.  For the current status of the NIST Post-Quantum Cryptography Standardization Project, see {{NIST}}.  For additional perspectives on the general transition from classical to post-quantum cryptography, see for example {{ETSI}} and {{HOFFMAN}}, among others.

There have been several Internet-Drafts describing mechanisms for embedding post-quantum and/or hybrid key exchange in TLS:

- Internet-Drafts for TLS 1.2: {{WHYTE12}}
- Internet-Drafts for TLS 1.3: {{KIEFER}}, {{SCHANCK}}, {{WHYTE13}}

There have been several prototype implementations for post-quantum and/or hybrid key exchange in TLS:

- Experimental implementations in TLS 1.2: {{BCNS15}}, {{CECPQ1}}, {{FRODO}}, {{OQS-102}}
- Experimental implementations in TLS 1.3: {{CECPQ2}}, {{OQS-111}}

These experimental implementations have taken an ad hoc approach and not attempted to implement one of the drafts listed above.

Unrelated to post-quantum but still related to the issue of combining multiple types of keying material in TLS is the use of pre-shared keys, especially the recent TLS working group document on including an external pre-shared key {{EXTERN-PSK}}.

Considering other IETF standards, there is work on post-quantum preshared keys in IKEv2 {{IKE-PSK}} and a framework for hybrid key exchange in IKEv2 {{IKE-HYBRID}}.  The XMSS hash-based signature scheme has been published as an informational RFC by the IRTF {{XMSS}}.

In the academic literature, {{EVEN}} initiated the study of combining multiple symmetric encryption schemes; {{ZHANG}}, {{DODIS}}, and {{HARNIK}} examined combining multiple public key encryption schemes, and {{HARNIK}} coined the term "robust combiner" to refer to a compiler that constructs a hybrid scheme from individual schemes while preserving security properties.  {{GIACON}} and {{BINDEL}} examined combining multiple key encapsulation mechanisms.

# Overview {#overview}

We identify four distinct axes along which one can make choices when integrating hybrid key exchange into TLS 1.3:

1. How to negotiate the use of hybridization in general and component algorithms specifically?
2. How many component algorithms can be combined?
3. How should multiple key shares (public keys / ciphertexts) be conveyed?
4. How should multiple shared secrets be combined?

The remainder of this document outlines various options we have identified for each of these choices.  Immediately below we provide a summary list.  Options are labelled with a short code in parentheses to provide easy cross-referencing.

1. [(Neg)](#neg) How to negotiate the use of hybridization in general and component algorithms specifically?

    - [(Neg-Ind)](#neg-ind) Negotiating component algorithms individually

      - [(Neg-Ind-1)](#neg-ind-1) Traditional algorithms in `ClientHello` `supported_groups` extension, next-gen algorithms in another extension
      - [(Neg-Ind-2)](#neg-ind-2) Both types of algorithms in `supported_groups` with external mapping to tradition/next-gen.
      - [(Neg-Ind-3)](#neg-ind-3) Both types of algorithms in `supported_groups` separated by a delimiter.

    - [(Neg-Comb)](#neg-comb) Negotiating component algorithms as a combination

      - [(Neg-Comb-1)](#neg-comb-1) Standardize `NamedGroup` identifiers for each desired combination.
      - [(Neg-Comb-2)](#neg-comb-2) Use placeholder identifiers in `supported_groups` with an extension defining the combination corresponding to each placeholder.
      - [(Neg-Comb-3)](#neg-comb-3) List combinations by inserting grouping delimiters into `supported_groups` list.

2. [(Num)](#num) How many component algorithms can be combined?

    - [(Num-2)](#num-2) Two.
    - [(Num-2+)](#num-2-plus) Two or more.

3. [(Shares)](#shares) How should multiple key shares (public keys / ciphertexts) be conveyed?

    - [(Shares-Concat)](#shares-concat) Concatenate each combination of key shares.
    - [(Shares-Multiple)](#shares-multiple) Send individual key shares for each algorithm.
    - [(Shares-Ext-Additional)](#shares-ext-additional) Use an extension to convey key shares for component algorithms.

4. [(Comb)](#comb) How should multiple shared secrets be combined?

    - [(Comb-Concat)](#comb-concat) Concatenate the shared secrets then use directly in the TLS 1.3 key schedule.
    - [(Comb-KDF-1)](#comb-kdf-1) and [(Comb-KDF-2)](#comb-kdf-2) KDF the shared secrets together, then use the output in the TLS 1.3 key schedule.
    - [(Comb-XOR)](#comb-xor) XOR the shared secrets then use directly in the TLS 1.3 key schedule.
    - [(Comb-Chain)](#comb-chain) Extend the TLS 1.3 key schedule so that there is a stage of the key schedule for each shared secret.
    - [(Comb-AltInput)](#comb-altinput) Use the second shared secret in an alternate (otherwise unused) input in the TLS 1.3 key schedule.

# Design options

## (Neg) How to negotiate hybridization and component algorithms? {#neg}

### Key exchange negotiation in TLS 1.3

Recall that in TLS 1.3, the key exchange mechanism is negotiated via the `supported_groups` extension.  The `NamedGroup` enum is a list of standardized groups for Diffie--Hellman key exchange, such as `secp256r1`, `x25519`, and `ffdhe2048`.  

The client, in its `ClientHello` message, lists its supported mechanisms in the `supported_groups` extension.  The client also optionally includes the public key of one or more of these groups in the `key_share` extension as a guess of which mechanisms the server might accept in hopes of reducing the number of round trips.  

If the server is willing to use one of the client's requested mechanisms, it responds with a `key_share` extension containing its public key for the desired mechanism.

If the server is not willing to use any of the client's requested mechanisms, the server responds with a `HelloRetryRequest` message that includes an extension indicating its preferred mechanism.

### (Neg-Ind) Negotiating component algorithms individually {#neg-ind}

In these three approaches, the parties negotiate which traditional algorithm and which next-gen algorithm to use independently.  The `NamedGroup` enum is extended to include algorithm identifiers for each next-gen algorithm.

#### (Neg-Ind-1) {#neg-ind-1}

The client advertises two lists to the server: one list containing its supported traditional mechanisms (e.g. via the existing `ClientHello` `supported_groups` extension), and a second list containing its supported next-generation mechanisms (e.g., via an additional `ClientHello` extension).  A server could then select one algorithm from the traditional list, and one algorithm from the next-generation list.  (This is the approach in {{SCHANCK}}.)

#### (Neg-Ind-2) {#neg-ind-2}

The client advertises a single list to the server which contains both its traditional and next-generation mechanisms (e.g., all in the existing `ClientHello` `supported_groups` extension), but with some external table provides a standardized mapping of those mechanisms as either "traditional" or "next-generation".  A server could then select two algorithms from this list, one from each category.

#### (Neg-Ind-3) {#neg-ind-3}

The client advertises a single list to the server delimited into sublists: one for its traditional mechanisms and one for its next-generation mechanisms, all in the existing `ClientHello` `supported_groups` extension, with a special code point serving as a delimiter between the two lists.  For example, `supported_groups = secp256r1, x25519, delimiter, nextgen1, nextgen4`.

### (Neg-Comb) Negotiating component algorithms as a combination {#neg-comb}

In these three approaches, combinations of key exchange mechanisms appear as a single monolithic block; the parties negotiate which of several combinations they wish to use.

#### (Neg-Comb-1) {#neg-comb-1}

The `NamedGroup` enum is extended to include algorithm identifiers for each **combination** of algorithms desired by the working group.  There is no "internal structure" to the algorithm identifiers for each combination, they are simply new code points assigned arbitrarily.  The client includes any desired combinations in its `ClientHello` `supported_groups` list, and the server picks one of these.  This is the approach in {{KIEFER}} and {{OQS-111}}.

#### (Neg-Comb-2) {#neg-comb-2}

The `NamedGroup` enum is extended to include algorithm identifiers for each next-gen algorithm.  Some additional field/extension is used to convey which combinations the parties wish to use.  For example, in {{WHYTE13}}, there are distinguished `NamedGroup` called `hybrid_marker 0`, `hybrid_marker 1`, `hybrid_marker 2`, etc.  This is complemented by a `HybridExtension` which contains mappings for each numbered `hybrid_marker` to the set of component key exchange algorithms (2 or more) for that proposed combination.

#### (Neg-Comb-3) {#neg-comb-3}

The client lists combinations in `supported_groups` list, using a special delimiter to indicate combinations.  For example,
`supported_groups = combo_delimiter, secp256r1, nextgen1, combo_delimiter, secp256r1, nextgen4, standalone_delimiter, secp256r1, x25519` would indicate that the client's highest preference is the combination secp256r1+nextgen1, the next highest preference is the combination secp2561+nextgen4, then the single algorithm secp256r1, then the single algorithm x25519.  A hybrid-aware server would be able to parse these; a hybrid-unaware server would see `unknown, secp256r1, unknown, unknown, secp256r1, unknown, unknown, secp256r1, x25519`, which it would be able to process, although there is the potential that every "projection" of a hybrid list that is tolerable to a client does not result in list that is tolerable to the client.

### Benefits and drawbacks

**Combinatorial explosion.** [(Neg-Comb-1)](#neg-comb-1) requires new identifiers to be defined for each desired combination.  The other 4 options in this section do not.

**Extensions.** [(Neg-Ind-1)](#neg-ind-1) and [(Neg-Comb-2)](#neg-comb-2) require new extensions to be defined.  The other options in this section do not.

**New logic.** All options in this section except [(Neg-Comb-1)](#neg-comb-1) require new logic to process negotiation.

**Matching security levels.** [(Neg-Ind-1)](#neg-ind-1), [(Neg-Ind-2)](#neg-ind-2), [(Neg-Ind-3)](#neg-ind-3), and [(Neg-Comb-2)](#neg-comb-2) allow algorithms of different claimed security level from their corresponding lists to be combined.  For example, this could result in combining ECDH secp256r1 (classical security level 128) with NewHope-1024 (classical security level 256).  Implementations dissatisfied with a mismatched security levels must either accept this mismatch or attempt to renegotiate.  [(Neg-Ind-1)](#neg-ind-1), [(Neg-Ind-2)](#neg-ind-2), and [(Neg-Ind-3)](#neg-ind-3) give control over the combination to the server; [(Neg-Comb-2)](#neg-comb-2) gives control over the combination to the client.  [(Neg-Comb-1)](#neg-comb-1) only allows standardized combinations, which could be set by TLS working group to have matching security (provided security estimates do not evolve separately).

**Backwards-compability.** TLS 1.3-compliant hybrid-unaware servers should ignore unreocgnized elements in `supported_groups` [(Neg-Ind-2)](#neg-ind-2), [(Neg-Ind-3)](#neg-ind-3), [(Neg-Comb-1)](#neg-comb-1), [(Neg-Comb-2)](#neg-comb-2) and unrecognized `ClientHello` extensions [(Neg-Ind-1)](#neg-ind-1), [(Neg-Comb-2)](#neg-comb-2).  In [(Neg-Ind-3)](#neg-ind-3) and [(Neg-Comb-3)](#neg-comb-3), a server that is hybrid-unaware will ignore the delimiters in `supported_groups`, and thus might try to negotiate an algorithm individually that is only meant to be used in combination; depending on how such an implementation is coded, it may also encounter bugs when the same element appears multiple times in the list.

## (Num) How many component algorithms to combine? {#num}

### (Num-2) Two {#num-2}

Exactly two algorithms can be combined together in hybrid key exchange.  This is the approach taken in {{KIEFER}} and {{SCHANCK}}.

### (Num-2+) Two or more {#num-2-plus}

Two or more algorithms can be combined together in hybrid key exchange.  This is the approach taken in {{WHYTE13}}.

### Benefits and Drawbacks

Restricting the number of component algorithms that can be hybridized to two substantially reduces the generality required.  On the other hand, some adopters may want to further reduce risk by employing multiple next-gen algorithms built on different cryptographic assumptions.

## (Shares) How to convey key shares? {#shares}

In ECDH ephmeral key exchange, the client sends its ephmeral public key in the `key_share` extension of the `ClientHello` message, and the server sends its ephmeral public key in the `key_share` extension of the `ServerHello` message.

For a general key encapsulation mechanism used for ephemeral key exchange, we imagine that that client generates a fresh KEM public key / secret pair for each connection, sends it to the client, and the server responds with a KEM ciphertext.  For simplicity and consistency with TLS 1.3 terminology, we will refer to both of these types of objects as "key shares".

In hybrid key exchange, we have to decide how to convey the client's two (or more) key shares, and the server's two (or more) key shares.

### (Shares-Concat) Concatenate key shares {#shares-concat}

The client concatenates the bytes representing its two key shares and uses this directly as the `key_exchange` value in a `KeyShareEntry` in its `key_share` extension.  The server does the same thing.  Note that the `key_exchange` value can be an octet string of length at most 2^16-1.  This is the approach taken in {{KIEFER}}, {{OQS-111}}, and {{WHYTE13}}.

### (Shares-Multiple) Send multiple key shares {#shares-multiple}

The client sends multiple key shares directly in the `client_shares` vectors of the `ClientHello` `key_share` extension.  The server does the same.  (Note that while the existing `KeyShareClientHello` struct allows for multiple key share entries, the existing `KeyShareServerHello` only permits a single key share entry, so some modification would be required to use this approach for the server to send multiple key shares.)

### (Shares-Ext-Additional) Extension carrying additional key shares {#shares-ext-additional}

The client sends the key share for its traditional algorithm in the original `key_share` extension of the `ClientHello` message, and the key share for its next-gen algorithm in some additional extension in the `ClientHello` message.  The server does the same thing.  This is the approach taken in {{SCHANCK}}.

### Benefits and Drawbacks

**Backwards compatibility.** [(Shares-Multiple)](#shares-multiple) is fully backwards compatible with non-hybrid-aware servers.  [(Shares-Ext-Additional)](#shares-ext-additional) is backwards compatible with non-hybrid-aware servers provided they ignore unrecognized extensions.  [(Shares-Concat)](#shares-concat) is backwards-compatible with non-hybrid aware servers, but may result in duplication / additional round trips (see below).

**Duplication versus additional round trips.** If a client wants to offer multiple key shares for multiple combinations in order to avoid retry requests, then the client may ended up sending a key share for one algorithm multiple times when using [(Shares-Ext-Additional)](#shares-ext-additional) and [(Shares-Concat)](#shares-concat).  (For example, if the client wants to send an ECDH-secp256r1 + McEliece123 key share, and an ECDH-secp256r1 + NewHope1024 key share, then the same ECDH public key may be sent twice.  If the client also wants to offer a traditional ECDH-only key share for non-hybrid-aware implementations and avoid retry requests, then that same ECDH public key may be sent another time.)  [(Shares-Multiple)](#shares-multiple) does not result in duplicate key shares.

## (Comb) How to use keys? {#comb}

Each component key exchange algorithm establishes a shared secret.  These shared secrets must be combined in some way that achieves the "hybrid" property: the resulting secret is secure as long as at least one of the component key exchange algorithms is unbroken.

### (Comb-Concat) Concatenate keys {#comb-concat}

Each party concatenates the shared secrets established by each component algorithm in an agreed-upon order, then feeds that through the TLS key schedule.  In the context of TLS 1.3, this would mean using the concatenated shared secret in place of the (EC)DHE input to the second call to `HKDF-Extract` in the TLS 1.3 key schedule:

~~~~
                                    0
                                    |
                                    v
                      PSK ->  HKDF-Extract = Early Secret
                                    |
                                    +-----> Derive-Secret(...)
                                    +-----> Derive-Secret(...)
                                    +-----> Derive-Secret(...)
                                    |
                                    v
                              Derive-Secret(., "derived", "")
                                    |
                                    v
concatenated_shared_secret -> HKDF-Extract = Handshake Secret
^^^^^^^^^^^^^^^^^^^^^^^^^^          |
                                    +-----> Derive-Secret(...)
                                    +-----> Derive-Secret(...)
                                    |
                                    v
                              Derive-Secret(., "derived", "")
                                    |
                                    v
                         0 -> HKDF-Extract = Master Secret
                                    |
                                    +-----> Derive-Secret(...)
                                    +-----> Derive-Secret(...)
                                    +-----> Derive-Secret(...)
                                    +-----> Derive-Secret(...)
~~~~

This is the approach used in {{KIEFER}}, {{OQS-111}}, and {{WHYTE13}}.  

{{GIACON}} analyzes the security of applying a KDF to concatenated KEM shared secrets, but their analysis does not exactly apply here since the transcript of ciphertexts is included in the KDF application (though it should follow relatively straightforwardly).  

{{BINDEL}} analyzes the security of the (Comb-Concat) approach as abstracted in their `dualPRF` combiner.  They show that, if the component KEMs are IND-CPA-secure (or IND-CCA-secure), then the values output by `Derive-Secret` are IND-CPA-secure (respectively, IND-CCA-secure).  An important aspect of their analysis is that each ciphertext is input to the final PRF calls; this holds for TLS 1.3 since the `Derive-Secret` calls that derive output keys (application traffic secrets, and exporter and resumption master secrets) include the transcript hash as input.

### (Comb-KDF-1) KDF keys {#comb-kdf-1}

Each party feeds the shared secrets established by each component algorithm in an agreed-upon order into a KDF, then feeds that through the TLS key schedule.  In the context of TLS 1.3, this would mean first applying `HKDF-Extract` to the shared secrets, then using the output in place of the (EC)DHE input to the second call to `HKDF-Extract` in the TLS 1.3 key schedule:

~~~~
                                    0
                                    |
                                    v
                      PSK ->  HKDF-Extract = Early Secret
                                    |
                                    +-----> Derive-Secret(...)
                                    +-----> Derive-Secret(...)
                                    +-----> Derive-Secret(...)
               Next-Gen             |
                   |                v
  (EC)DHE -> HKDF-Extract     Derive-Secret(., "derived", "")
                   |                |
                   v                v
                output -----> HKDF-Extract = Handshake Secret
                ^^^^^^              |
                                    +-----> Derive-Secret(...)
                                    +-----> Derive-Secret(...)
                                    |
                                    v
                              Derive-Secret(., "derived", "")
                                    |
                                    v
                         0 -> HKDF-Extract = Master Secret
                                    |
                                    +-----> Derive-Secret(...)
                                    +-----> Derive-Secret(...)
                                    +-----> Derive-Secret(...)
                                    +-----> Derive-Secret(...)
~~~~

### (Comb-KDF-2) KDF keys {#comb-kdf-2}

Each party concatenates the shared secrets established by each component algorithm in an agreed-upon order then feeds that into a KDF, then feeds the result through the TLS key schedule.

Compared with [(Comb-KDF-1)](#comb-kdf-1), this method concatenates the (2 or more) shared secrets prior to input to the KDF, whereas (Comb-KDF-1) puts the (exactly 2) shared secrets in the two different input slots to the KDF.

Compared with [(Comb-Concat)](#comb-concat), this method has an extract KDF application.  While this adds computational overhead, this may provide a cleaner abstraction of the hybridization mechanism for the purposes of formal security analysis. 

~~~~
                                    0
                                    |
                                    v
                      PSK ->  HKDF-Extract = Early Secret
                                    |
                                    +-----> Derive-Secret(...)
                                    +-----> Derive-Secret(...)
                                    +-----> Derive-Secret(...)
                                    |
                                   v
  concatenated     0
  shared           |
  secret  -> HKDF-Extract     Derive-Secret(., "derived", "")
  ^^^^^^           |                |
                   v                v
                output -----> HKDF-Extract = Handshake Secret
                ^^^^^^              |
                                    +-----> Derive-Secret(...)
                                    +-----> Derive-Secret(...)
                                    |
                                    v
                              Derive-Secret(., "derived", "")
                                    |
                                    v
                         0 -> HKDF-Extract = Master Secret
                                    |
                                    +-----> Derive-Secret(...)
                                    +-----> Derive-Secret(...)
                                    +-----> Derive-Secret(...)
                                    +-----> Derive-Secret(...)
~~~~

### (Comb-XOR) XOR keys {#comb-xor}

Each party XORs the shared secrets established by each component algorithm (possibly after padding secrets of different lengths), then feeds that through the TLS key schedule.  In the context of TLS 1.3, this would mean using the XORed shared secret in place of the (EC)DHE input to the second call to `HKDF-Extract` in the TLS 1.3 key schedule.

{{GIACON}} analyzes the security of applying a KDF to the XORed KEM shared secrets, but their analysis does not quite apply here since the transcript of ciphertexts is included in the KDF application (though it should follow relatively straightforwardly).

### (Comb-Chain) Chain of KDF applications for each key {#comb-chain}

Each party applies a chain of key derivation functions to the shared secrets established by each component algorithm in an agreed-upon order; roughly speaking: `F(k1 || F(k2))`.  In the context of TLS 1.3, this would mean extending the key schedule to have one round of the key schedule applied for each component algorithm's shared secret:

~~~~
                                    0
                                    |
                                    v
                      PSK ->  HKDF-Extract = Early Secret
                                    |
                                    +-----> Derive-Secret(...)
                                    +-----> Derive-Secret(...)
                                    +-----> Derive-Secret(...)
                                    |
                                    v
                              Derive-Secret(., "derived", "")
                                    |
                                    v
 traditional_shared_secret -> HKDF-Extract
 ^^^^^^^^^^^^^^^^^^^^^^^^^          |
                              Derive-Secret(., "derived", "")
                                    |
                                    v
    next_gen_shared_secret -> HKDF-Extract = Handshake Secret
    ^^^^^^^^^^^^^^^^^^^^^^          |                             
                                    +-----> Derive-Secret(...)
                                    +-----> Derive-Secret(...)
                                    |
                                    v
                              Derive-Secret(., "derived", "")
                                    |
                                    v
                         0 -> HKDF-Extract = Master Secret
                                    |
                                    +-----> Derive-Secret(...)
                                    +-----> Derive-Secret(...)
                                    +-----> Derive-Secret(...)
                                    +-----> Derive-Secret(...)
~~~~

This is the approach used in {{SCHANCK}}.

{{BINDEL}} analyzes the security of this approach as abstracted in their nested dual-PRF `N` combiner, showing a similar result as for the dualPRF combiner that it preserves IND-CPA (or IND-CCA) security. Again their analysis depends on each ciphertext being input to the final PRF (`Derive-Secret`) calls, which holds for TLS 1.3.

### (Comb-AltInput) Second shared secret in an alternate KDF input {#comb-altinput}

In the context of TLS 1.3, the next-generation shared secret is used in place of a currently unused input in the TLS 1.3 key schedule, namely replacing the `0` "IKM" input to the final `HKDF-Extract`:

~~~~
                                    0
                                    |
                                    v
                      PSK ->  HKDF-Extract = Early Secret
                                    |
                                    +-----> Derive-Secret(...)
                                    +-----> Derive-Secret(...)
                                    +-----> Derive-Secret(...)
                                    |
                                    v
                              Derive-Secret(., "derived", "")
                                    |
                                    v
 traditional_shared_secret -> HKDF-Extract = Handshake Secret
 ^^^^^^^^^^^^^^^^^^^^^^^^^          |
                                    +-----> Derive-Secret(...)
                                    +-----> Derive-Secret(...)
                                    |
                                    v
                              Derive-Secret(., "derived", "")
                                    |
                                    v
    next_gen_shared_secret -> HKDF-Extract = Master Secret
    ^^^^^^^^^^^^^^^^^^^^^^          |
                                    +-----> Derive-Secret(...)
                                    +-----> Derive-Secret(...)
                                    +-----> Derive-Secret(...)
                                    +-----> Derive-Secret(...)
~~~~

This approach is not taken in any of the known post-quantum/hybrid TLS drafts.  However, it bears some similarities to the approach for using external PSKs in {{EXTERN-PSK}}.

### Benefits and Drawbacks

**New logic.**  While [(Comb-Concat)](#comb-concat), [(Comb-KDF-1)](#comb-kdf-1), and [(Comb-KDF-2)](#comb-kdf-2) require new logic to compute the concatenated shared secret, this value can then be used by the TLS 1.3 key schedule without changes to the key schedule logic.  In contrast, [(Comb-Chain)](#comb-chain) requires the TLS 1.3 key schedule to be extended for each extra component algorithm.  

**Philosophical.**  The TLS 1.3 key schedule already applies a new stage for different types of keying material (PSK versus (EC)DHE), so [(Comb-Chain)](#comb-chain) continues that approach.

**Efficiency.** [(Comb-KDF-1)](#comb-kdf-1), [(Comb-KDF-2)](#comb-kdf-2), and [(Comb-Chain)](#comb-chain) increase the number of KDF applications for each component algorithm, whereas [(Comb-Concat)](#comb-concat) and [(Comb-AltInput)](#comb-altinput) keep the number of KDF applications the same (though with potentially longer inputs).

**Extensibility.**  [(Comb-AltInput)](#comb-altinput) changes the use of an existing input, which might conflict with other future changes to the use of the input.

**More than 2 component algorithms.**  The techniques in [(Comb-Concat)](#comb-concat) and [(Comb-Chain)](#comb-chain) can naturally accommodate more than 2 component shared secrets since there is no distinction to how each shared secret is treated.  [(Comb-AltInput)](#comb-altinput) would have to make some distinct, since the 2 component shared secrets are used in different ways; for example, the first shared secret is used as the "IKM" input in the 2nd `HKDF-Extract` call, and all subsequent shared secrets are concatenated to be used as the "IKM" input in the 3rd `HKDF-Extract` call.

### Open questions {#comb-open-questions}

At this point, it is unclear which, if any, of the above methods preserve FIPS compliance: i.e., if one shared secret is from a FIPS-compliant method (e.g., ECDH), and another shared secret is from a non-approved method (e.g., post-quantum), is the result still considered FIPS compliant?  Guidance from NIST on this question would be helpful.  Specifically, are any of these approaches acceptable under either {{NIST-SP-800-56C}} or {{NIST-SP-800-135}}?

# Candidate instantiations {#candidate}

In this section, we describe two candidate instantiations of hybrid key exchange in TLS 1.3, based on the design considerations framework above.  It is not our intention that both of these instantations be standardized; we are providing two for discussion and for comparing and contrasting the two approaches.

## Candidate Instantiation 1 {#candidate-1}

Candidate Instantiation 1 allows for two or more component algorithms to be combined [(Num-2+)](#num-2-plus), and negotiates the combination using markers in the `NamedGroup` list as pointers to an extension listing the algorithms comprising each possible combination [(Neg-Comb-2)](#neg-comb-2) following the approach of {{WHYTE13}}.  The client conveys its multiple key shares individually in the `client_shares` vector of the `ClientHello` `key_share` extension [(Shares-Multiple)](#shares-multiple).  The server conveys its multiple key shares concatenated together in its `KeyShareServerHello` struct [(Shares-Concat)](#shares-concat).  The shared secrets are combined by concatenating them then feeding them through a KDF, then feeding the result into the TLS 1.3 key schedule [(Comb-KDF-2)](#comb-kdf-2).

### ClientHello extension supported_groups

Following {{WHYTE13}} section 3.1, the `NamedGroup` enum used by the client to populate the `supported_groups` extension is extended to include new code points representing markers for hybrid combinations:

     enum {
         /* existing named groups */
         secp256r1 (23), 
         ...,
         
         /* new code points eventually defined for post-quantum algorithms */
         ...,

         /* new code points reserved for hybrid markers */
         hybrid_marker00 (0xFD00),
         hybrid_marker01 (0xFD01),
         ...
         hybrid_markerFF (0xFDFF),

         /* existing reserved code points */
         ffdhe_private_use (0x01FC..0x01FF),
         ecdhe_private_use (0xFE00..0xFEFF),
         (0xFFFF)
      } NamedGroup;

`hybrid_marker` code points do not a priori represent any fixed combination.  Instead, during each session establishment, the client defines what it wants each `hybrid_marker` code point to represent using the following extension.

### ClientHello extension hybrid_extension

Following {{WHYTE13}} section 3.2.4, a new `ClientHello` `hybrid_extension` extension is defined.  It is defined as follows:

    struct {
        NamedGroup hybrid_marker;
        NamedGroup components<2..10>;
    } HybridMapping;
    
    struct {
        HybridMapping map<0..255>;
    } HybridExtension;

The `HybridExtension` contains 0 or more `HybridMapping`s.  Each `HybridMapping` corresponds to one of the `hybrid_marker` included in the `supported_groups` extension, and lists the component algorithms that are meant to comprise the this hybrid combination, which can be any of the existing named groups (elliptic curve or finite field), new code points eventually defined for post-quantum algorithms, or reserved code points for private use.

### ClientHello extension key_share

No syntactical modifications are made to the `KeyShareEntry` or `KeyShareClientHello` data structures.  

Semantically, the client does not send a `KeyShareEntry` corresponding to any of the `hybrid_marker` code points.  Instead, the client sends `KeyShareEntry` for each of the component algorithms listed in the `HybridMapping`s.

For example, if the list of `supported_groups` is `secp256r1`, `x25519`, `hybrid_marker00`, and `hybrid_marker01`, where `hybrid_marker00` comprises `secp256r1` with a fictional post-quantum algorithm `PQ1`, and `hybrid_marker01` comprises `x25519` with `PQ1`, then the client could send three `KeyShareEntry` components: one for `secp256r1`, one for `x25519`, and one for `PQ1`.

### ServerHello extension KeyShareServerHello

The server responds with a `KeyShareServerHello` struct containing a single `KeyShareEntry`, which contains a single `NamedGroup` value and an opaque `key_exchange` string.  

To complete the negotiation of a hybrid algorithm, the server responds with the `NamedGroup` value being the `hybrid_marker` code point correspond to the combination that the server was willing to agree to.

The `key_exchange` string is the octet representation of the following struct:

    struct {
        KeyShareEntry key_share<2..10>;
    } HybridKeyShare;

where there is one `key_share` entry for each of the components of this hybrid combination.

Note that the `key_exchange` string has a maximum length of 2^16-1 octets, which may be insufficient for some post-quantum algorithms or for some hybridizations of multiple post-quantum algorithms.  It remains an open question as to whether this length can be increased without breaking existing TLS 1.3 implementations.

### Key schedule

The component algorithm shared secrets are combined by concatenating them, then applying a key derivation function, the output of which is then used in the TLS 1.3 key schedule in place of the (EC)DHE shared secret.  The component shared secrets are concatenated in the order that they appear in the `components` vector of the `HybridMapping` extension above.

We provide two options for concatenating the shared secrets, and would like feedback from the working group in which to proceed with.

Each component algorithm's `shared_secret` is defined by the algorithm itself, for example the DHE or ECDHE shared secrets as defined in Section 7.4 of {{ TLS13 }}, or as defined by post-quantum methods once standardized in their own documents.

**Option 1: Using data structures.**  Option 1 uses a full-fledged TLS 1.3 data structure to represent the list of component shared secrets.  As a result, lengths of each shared secret are unambiguously encoded.

    struct SharedSecret {
        opaque shared_secret<0..2^16-1>;
    }

    struct {
        SharedSecret component<2..10>;
    } HybridSharedSecret;

The `concatenated_shared_secret` is then the octet representation of the `HybridSharedSecret ` struct.

**Option 2: Direct concatenation.**  Option 2 directly concatenates the shared secrets.  Option 2 should only be considered if the shared secret for each algorithm is guarantees to be of a fixed length, which would imply that, once the component algorithms are fixed, concatenation is bijective.

    concatenated_shared_secret = shared_secret0 | shared_secret1 | ...

In either option, the `concatenated_shared_secret` octet string is used as the IKM argument of HKDF-Extract, with the zero-length string as the salt argument.  THe output of HKDF-Extract is used as the IKM argument for HKDF-Extract's calculation of the handshake secret, as shown below.

~~~~
                                    0
                                    |
                                    v
                      PSK ->  HKDF-Extract = Early Secret
                                    |
                                    +-----> Derive-Secret(...)
                                    +-----> Derive-Secret(...)
                                    +-----> Derive-Secret(...)
                                    |
                                   v
  concatenated     0
  shared           |
  secret  -> HKDF-Extract     Derive-Secret(., "derived", "")
  ^^^^^^           |                |
                   v                v
                output -----> HKDF-Extract = Handshake Secret
                ^^^^^^              |
                                    +-----> Derive-Secret(...)
                                    +-----> Derive-Secret(...)
                                    |
                                    v
                              Derive-Secret(., "derived", "")
                                    |
                                    v
                         0 -> HKDF-Extract = Master Secret
                                    |
                                    +-----> Derive-Secret(...)
                                    +-----> Derive-Secret(...)
                                    +-----> Derive-Secret(...)
                                    +-----> Derive-Secret(...)
~~~~

## Candidate Instantiation 2 {#candidate-2}

Candidate Instantiation 2 allows for exactly two component algorithms to be combined [(Num-2)](#num-2), and uses code points standardized for each permissible combination.  The client concatenates its multiple key shares together as a distinct entry in the `client_shares` vector of the `ClientHello` `key_share` extension [(Shares-Concat)](#shares-concat).  The server does the same.  The shared secrets are combined by concatenating them then feeding them through a KDF, then feeding the result into the TLS 1.3 key schedule [(Comb-KDF-2)](#comb-kdf-2).

### ClientHello extension supported_groups

The `NamedGroup` enum used by the client to populate the `supported_groups` extension is extended to include new code points representing each desired combination.

For example,

     enum {
         /* existing named groups */
         secp256r1 (23), 
         x25519 (0x001D),
         ...,
         
         /* new code points eventually defined for post-quantum algorithms */
         PQ1 (0x????),
         PQ2 (0x????),
         ...,

         /* new code points defined for hybrid combinations */
         secp256r1_PQ1 (0x????),
         secp256r1_PQ2 (0x????),
         x25519_PQ1 (0x????),
         x25519_PQ2 (0x????),

         /* existing reserved code points */
         ffdhe_private_use (0x01FC..0x01FF),
         ecdhe_private_use (0xFE00..0xFEFF),
         (0xFFFF)
      } NamedGroup;

### ClientHello extension KeyShareClientHello

The client sends a `KeyShareClientHello` struct containing multiple `KeyShareEntry` values, some of which may correspond to some of the hybrid combination code points it listed in the `supported_groups` extension above.  

The `KeyShareEntry` for a hybrid combination code point contains an opaque  `key_exchange` string which is the octet representation of the following struct:

    struct {
        KeyShareEntry key_share<2..10>;
    } HybridKeyShare;

where there is one `key_share` entry for each of the components of this hybrid combination.

Note that this approach may result in duplication of key shares being sent; for example, a client wanting to support either the combination `secp256r1_PQ1` or `x25519_PQ1` would send two `PQ1` key shares.

### ServerHello extension KeyShareServerHello

The server responds with a `KeyShareServerHello` struct containing a single `KeyShareEntry`, which contains a single `NamedGroup` value and an opaque `key_exchange` string.  The `key_exchange` string is the octet representation of the `HybridKeyShare` struct defined above.

### Key schedule

The key schedule is computed as in Candidate Instantiation 1 above.

## Comparing Candidate Instantiation 1 and 2

CI2 requires much less change to negotiation routines -- each hybrid combination is just a new key exchange method, and the concatenation of key shares and shared secrets can be handled internally to that method.  This comes at the cost, however, of combinatorial explosion of code points: one code point needs to be standardized for each desired combination.  We have also limited the number of hybrid algorithms to 2 in CI2 to somewhat limit the explosion of code points needing to be defined.  Concatenating client key shares also risks sending duplicate key shares, increasing communication sizes.

CI1 requires more change to negotiation routines, since it introduces new data structures and has an indirect mapping between hybrid combinations and key shares.  Benefits from this approach include avoiding sending duplicate key shares and not needing to standardize every possible supported combination.  Implementers, however, must do the work of deciding which combinations of algorithms are meaningful / tolerable / desirable from a security perspective, potentially complicating interoperability.

# IANA Considerations

If Candidate Instantiation 1 is selected, the TLS Supported Groups registry will have to be updated to include code points for hybrid markers.

# Security Considerations

The majority of this document is about security considerations.  As noted especially in {{comb}}, the shared secrets computed in the hybrid key exchange should be computed in a way that achieves the "hybrid" property: the resulting secret is secure as long as at least one of the component key exchange algorithms is unbroken.  While many natural approaches seem to achieve this, there can be subtleties (see for example the introduction of {{GIACON}}).

The rest of this section highlights a few unresolved questions related to security.

## Active security

One security consideration that is not yet resolved is whether key encapsulation mechanisms used in TLS 1.3 must be secure against active attacks (IND-CCA), or whether security against passive attacks (IND-CPA) suffices.  Existing security proofs of TLS 1.3 (such as {{DFGS15}}, {{DOWLING}}) are formulated specifically around Diffie--Hellman and use an "actively secure" Diffie--Hellman assumption (PRF Oracle Diffie--Hellman (PRF-ODH)) rather than a "passively secure" DH assumption (e.g. decisional Diffie--Hellman (DDH)), but do not claim that the actively secure notion is required.  In the context of TLS 1.2, {{KPW13}} show that, at least in one formalization, a passively secure assumption like DDH is insufficient (even when signatures are used for mutual authentication).  Resolving this issue for TLS 1.3 is an open question.

## Resumption

TLS 1.3 allows for session resumption via a pre-shared key.  When a pre-shared key is used during session establishment, an ephemeral key exchange can also be used to enhance forward secrecy.  If the original key exchange was hybrid, should an ephemeral key exchange in a resumption of that original key exchange be required to use the same hybrid algorithms?

## Failures

Some post-quantum key exchange algorithms have non-trivial failure rates: two honest parties may fail to agree on the same shared secret with non-negligible probability.  Does a non-negligible failure rate affect the security of TLS?  How should such a failure be treated operationally?  What is an acceptable failure rate?

# Acknowledgements

These ideas have grown from discussions with many colleagues, including Christopher Wood, Matt Campagna, and authors of the various hybrid Internet-Drafts and implementations cited in this document.  The immediate impetus for this document came from discussions with attendees at the Workshop on Post-Quantum Software in Mountain View, California, in January 2019.

Martin Thomson suggested the [(Comb-KDF-1)](#comb-kdf-1) approach.

--- back
