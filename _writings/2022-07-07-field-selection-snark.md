---
layout: post
author: Gautam Botrel
title: Field Selection for Recursive SNARKs
--- 

*Many thanks to Carol Xie, Daniel Lubarov, Shashank Agrawal, Ventali Tan, Youssef El Housni & Ariel Gabizon for feedback.*


Proof systems like Groth16, PlonK, Halo, Spartan or Marlin “encode” a computation as a set of polynomial equations, defined over a finite field Fr. Fr denotes a finite field of order r, which is generally a prime number. Encoding programs that were not designed to operate on Fr can result in gigantic (thus impractical) circuits, yet, it is key to SNARK programmability and composability.

In this article, we will explore field selection and encoding of “non native” arithmetic operations. Intuitively, it encompasses two problems. The first is that programmers are used to manipulate 32/64bit integers, floats, and booleans. The second is that at the protocol level, it is common to verify cryptographic constructs (hashes, signatures, proofs) that may be defined over an arbitrary field (GF2, Fq, …) inside our SNARK field F_r. 


## Field selection

If we’re using pairing-based SNARKs, we **cannot** use general-purpose elliptic curves[^1] such as secp256k1 or ed25519. We need curves which are pairing-friendly, which limits our options to tailored curves[^2] such as bn254 or bls12-381. You can read more about elliptic curves for proof systems in [A survey of elliptic curves for proof systems](https://eprint.iacr.org/2022/586.pdf). Note that for elliptic curve based constructs, SNARK circuit elements live in the curve’s _scalar field_ (Fr), while the proof is encoded in the _base field_ (Fp).

For most SNARKs (including non pairing-based constructs), it is also important to look at the factors in its multiplicative group. If it contains enough factors of two[^3], we can use FFTs to speed up polynomial computations. This includes evaluation, interpolation, and multiplication, all of which are common in SNARKs. Without FFTs, these operations would need to be done with quadratic algorithms, or special algorithms like [ECFFT](https://arxiv.org/abs/2107.08473)[^4]. You can read more about FFTs in this article by Vitalik: [Fast Fourier Transforms](https://vitalik.ca/general/2019/05/12/fft.html).

Moreover, choosing the right field is a tradeoff between security and performance. Larger fields slow down field arithmetic but can increase the security of some SNARKs. 

Next, why do we want to encode arithmetic statements defined over a field Fq into equations defined over Fr[^5]? 

This operation enables **proof recursion** such as a SNARK verifying SNARK[s], and more generally, **composing statements** defined over different fields. 

For example, Bitcoin and Ethereum use the ECDSA signature scheme over the secp256k1 elliptic curve. The curve is not pairing friendly and the base field is not highly 2-adic; in other words, we can’t instantiate Groth16 or PlonK zkSNARKs on top of it. Yet, an Ethereum Rollup or Bridge operator may want to encode such signatures in a SNARK, and verify them on the Mainnet using the available instructions ([BN254 precompiled](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-197.md) contract).

This generalizes to other primitives; a lattice-based construct may use a small field while RSA-based cryptography will use much larger fields, and one may want to encode these inside a SNARK.


---

We will explore 3 options to work with non-native field arithmetic operations. 

**Brute force**

If we want to emulate Fq arithmetic in Fr, we can decompose the Fq elements in limbs and operate on those limbs. We may reduce and recompose the element only when necessary. The approach is computationally quite expensive, since it involves bit decompositions and range checks. It may represent up to 1000X the original constraints in the limbs and may blow up the circuit. 

[xJsnark: A Framework for Efficient Verifiable Computation](https://akosba.github.io/papers/xjsnark.pdf) details this approach. [gnark](https://github.com/ConsenSys/gnark/blob/3d3672148b38c548b6527c5f2cfe1af5ae61a11f/std/math/nonnative/doc.go) has a package for operations over any modulus with the split-limb structure.

**2-chains and cycles**

Another approach [for pairing based schemes] is to find a curve C1 whose base field matches a curve C2 scalar field. This targets pairing based schemes which have constant time pairing verification. Since the fields are the same, it is efficient to implement a “C1-SNARK verifier” inside a “C2 SNARK”. These are called “[2-chains of elliptic curves](https://eprint.iacr.org/2021/1359.pdf)”. Projects using these include Aleo and Celo. 

Non-pairing based schemes can use non-pairing friendly cycles or hybrid cycles at a cost. 

The Polynomial Commitment Scheme (PCS) part in non-pairing based schemes has a linear verification time, which will proportionally increase proof verifications cost and circuit size. The additional time complexity may be amortized with accumulation techniques. Examples of non-pairing friendly or hybrid cycles include ZCash’s[ Pasta Curves](https://electriccoin.co/blog/the-pasta-curves-for-halo-2-and-beyond/) (also used in Mina). In a nutshell, the idea is to enable proof recursion (proof verifying proofs) and to delay the linear part of the computation to the last layer of recursion.

It is non-trivial to find curves satisfying all sought after properties to enable recursive SNARKs. Yet this section is particularly relevant to our opening problem. In some cases, 1 layer of recursion is all that is required by the protocol; being able to _avoid_ field emulation through recursion is a powerful concept. Recall also the issue with brute force; the size of the circuit grows, potentially making proving time (and sometimes proof size) less practical. 

Recursive SNARKs tackle that; the final proof may compress thousands of other proofs, which may be computed in parallel.

**Lower the requirements on the field**

The key building block of some recent proof systems is the Polynomial Commitment Scheme[^6] (PCS). Its choice directly impacts the properties needed in the field. For example, a FRI-based PCS does not involve elliptic curves at all. We still need the 2-adicity for FFTs, so we can’t use this technique with the secp256k1 field. In theory, novel PCS constructs, for example based on sumchecks or ECFRI, could enable efficient dedicated proof systems to be instantiated on arbitrary fields. This is work in progress and it may result in a complexity (prover/verifier time, proof size) tradeoff.


---

These options are not mutually exclusive. They have various properties in prover time, proof size, security assumptions, and more, and can co-exist in a larger design. Moreover, virtual machines like AIR, Cairo and zkEVM propose different solutions to efficiently compute in different arithmetic fields. 

New proof systems have been introduced by researchers and developers at a rapid pace. Some systems use elliptic curves while others might not. The proof systems decide the efficiency of the building blocks such as the choice of the hash functions, signature schemes, etc. The performance outcome of the building blocks feeds back to the choice of the system design and has repercussions at the protocol level. 

Bridges and cross-chain applications between L1 and L2 can involve different arithmetic fields for the proof systems. The ability to implement cryptographic constructs with different fields will be crucial for the evolution of proof systems. This is an interesting and useful ongoing research domain, and we are looking forward to progress in field interoperability and practical implementations in different ecosystems. 

__________________________________

_If you are a zero-knowledge proof or cryptography expert interested in further discussions or working together on this subject, please reach out to me at gautam@delendum.xyz._


<!-- Footnotes themselves at the bottom. -->
**Footnotes**

[^1]:
     Generic-purpose elliptic curves refer to any curves that are secure with respect to the Discrete Logarithm Problem (DLP), with no known weaknesses.

[^2]:
     Other factors include the embedding degree ratio against the subgroup order, the seed, the trace, which will impact security, field size, and performance overall of the pairing. 

[^3]:
     In principle, other factors can also be used in a mixed-radix FFT. However, small radices are generally the most efficient, and many FFT implementations only support factors of 2.

[^4]:
     This is a recent research paper; efficient implementations are not available yet 

[^5]:
     Fr denotes the arithmetic field for the polynomial equations and Fq denotes the field for the proof. r may or may not be a prime. 

[^6]:
     The wikipedia page points to various commitment schemes: https://en.wikipedia.org/wiki/Commitment_scheme
