---
layout: post
author: Lita Foundation
title: "Plonky3 / Valida Review Notes"
excerpt: "Review criteria for Plonky3 and Valida MVP focus on functional completeness, computational robustness, algorithmic efficiency, and compatibility across systems"
image: ""
---

By Morgan Thomas at Lita Foundation

**Objective**

The underlying objective of this article is to provide our developer community with a transparent update on our progress regarding Plonky3 and Valida. We understand and appreciate the enthusiasm and eagerness with which many are anticipating the release of these libraries for their projects. As we navigate through the complexities of development, your understanding and patience mean everything to us. In addition to the specifics covered in this review criteria, we will also be separately publishing detailed notes on the LLVM Valida compiler. This separate publication will ensure a focused and comprehensive discussion on that particular aspect. Our commitment remains steadfast in delivering a robust MVP that meets the standards and expectations of our community.

*Currently, based on our development trajectory, we project a combined total of 68 days (36 for Plonky3 and 32 for Valida) to complete the MVP. *

### Table of Content

- [Review criteria](#review-criteria)
   * [Functional completeness & happy path correctness](#functional-completeness-happy-path-correctness)
   * [Partiality](#partiality)
   * [Exception handling](#exception-handling)
   * [Floating point in verifiers](#floating-point-in-verifiers)
   * [Asymptotic complexity](#asymptotic-complexity)
   * [System support](#system-support)
- [Plonky3 review](#plonky3-review)
   * [Functional completeness & happy path correctness](#functional-completeness-happy-path-correctness-1)
   * [Partiality](#partiality-1)
   * [Exception handling](#exception-handling-1)
   * [Floating point in verifiers](#floating-point-in-verifiers-1)
   * [Asymptotic complexity](#asymptotic-complexity-1)
- [Valida review](#valida-review)
   * [Functional completeness & happy path correctness](#functional-completeness-happy-path-correctness-2)
   * [Partiality](#partiality-2)
   * [Exception handling](#exception-handling-2)
   * [Floating point in verifiers](#floating-point-in-verifiers-2)
   * [Asymptotic complexity](#asymptotic-complexity-2)

# Review criteria

This section outlines the criteria with respect to which Plonky3 and Valida will be reviewed in order to scope out the remaining work on them for the MVP. 


## Functional completeness & happy path correctness

This review will try to verify that Plonky3 and Valida are functionally complete for the purposes of the MVP, and that the “happy path” (non-exceptional) execution patterns are correct.


## Partiality

Partiality refers to when a partial computable function is not total computable: i.e., when the function may fail, for some inputs, to terminate with a result of its specified output type. There are two ways for a partial computable function to exhibit partiality on some inputs: either it fails to terminate on those inputs, or it exhibits an abnormal termination condition (such as a segmentation fault in C++ or a panic in Rust) on those inputs.

For our purposes, Plonky3 and Valida ought to consist of total functions. For an MVP, there should be no known partiality in Plonky3 and Valida. All functions exposed by their APIs should exhibit a normal termination condition on all inputs. This is important for ensuring the robustness of services built on these APIs.

This review will try to flag all potential causes of partiality in Valida and Plonky3.


## Exception handling

This review will try to assess what edge cases and exceptional conditions may occur in the anticipated Plonky3 / Valida execution paths. The review will try to flag any edge cases or exceptional conditions that may have an undesired result, such as:



1. unintended or inexpedient suppression of an error condition,
2. throwing an exception when a graceful failure would be preferable,
3. a graceful failure without adequate logging of the error condition,
4. an uninformative error message, or
5. an abnormal termination.


## Floating point in verifiers

The Plonky3 and Valida verifier algorithms must not make use of floating point primitives, because these will not be assumed to be present in the context of verification inside a Valida STARK, as in recursive proofs. This review will try to verify that floating point primitives are not being used in the Plonky3 and Valida verifiers, or flag any usage thereof.


## Asymptotic complexity

This review will try to identify any algorithms in Plonky3 and Valida which are in an asymptotic complexity class that is non-optimal for a solution to that problem (i.e., asymptotically slow solutions). Any asymptotically slow solutions in Plonky3 and Valida may likely present issues for scalability.


## System support

Valida and Plonky3 must be buildable _on _the following systems:



* x86_64-linux

Valida and Plonky3 must be buildable _targeting_ the following systems:



* x86_64-linux
* ARM64-android

The Valida and Plonky3 _verifiers _must be buildable _targeting_ the following systems:



* x86_64-linux
* ARM64-android
* Valida

The x86_64-linux and ARM-64-android targets are supported by `rustc` (which is based on LLVM), and so building Valida to run on these targets should not be a big problem. To build the verifiers to run on Valida, we’ll need to get the Valida LLVM backend to a sufficient level of completion.


# Plonky3 review

This review applies to commit `000681e190eb0f7fc8523f47b8bd06de075460d8` of Plonky3.

Plonky3 consists of the following functional components, listed in a linearization of their dependency ordering:



1. A collection of simple utility functions. This is located in the `util` subdirectory.
2. A feature-gated wrapper around <code>[rayon](https://docs.rs/rayon/latest/rayon/)</code>. This is located in the <code>maybe-rayon</code> subdirectory.
3. A field abstraction with various implementations and associated algorithms. This is located in the <code>field</code> subdirectory.
4. A collection of utilities for testing field implementations. This is located in the <code>field-testing</code> subdirectory.
5. A matrix abstraction with various implementations and associated algorithms. This is located in the <code>matrix</code> subdirectory.
6. A collection of AIR-related traits and implementations, including a two-row matrix view and affine functions over columns in a pair (“virtual columns”). This is located in the <code>air</code> subdirectory.
7. A framework for symmetric crypto primitives, including compression function related traits and implementations, a padding-free, overwrite-mode sponge function, a hasher trait, a serializing hasher, and cryptographic permutation related traits. This is located in the <code>symmetric</code> subdirectory.
8. An implementation of the Baby bear field and its quartic and quintic extension fields. This is located in the <code>baby-bear</code> subdirectory.
9. An implementation of the Goldilocks field and its quadratic extension field. This is located in the <code>goldilocks</code> subdirectory.
10. An implementation of the Mersenne-31 field and some of its field extensions, as well as DFT and radix 2 DIT implementations for Mersenne-31. This is located in the <code>mersenne-31</code> subdirectory.
11. A Keccak permutation and hash function implementation. This is located in the <code>keccak</code> subdirectory.
12. A Blake3 hash function implementation. This is located in the <code>blake3</code> subdirectory.
13. A library of DFT-related traits and implementations. This is located in the <code>dft</code> subdirectory.
14. A collection of coding theoretic traits. This is located in the <code>code</code> subdirectory.
15. A collection of LDE-related traits and implementations, used for testing only. This is located in the <code>lde</code> subdirectory.
16. An implementation of Reed-Solomon codes. This is located in the <code>reed-solomon</code> subdirectory.
17. A collection of traits and implementations for generating Fiat-Shamir challenges based on an IOP’s transcript. This is located in the <code>challenger</code> subdirectory.
18. A collection of commitment scheme traits. This is located in the <code>commit</code> subdirectory.
19. A collection of MDS permutation related traits and implementations. This is located in the <code>mds</code> subdirectory.
20. A Poseidon permutation implementation. This is located in the <code>poseidon</code> subdirectory.
21. A Poseidon2 permutation implementation and related traits and implementations. This is located in the <code>poseidon2</code> subdirectory.
22. A Rescue-XLIX permutation implementation. This is located in the <code>rescue</code> subdirectory.
23. A binary Merkle tree implementation and a vector commitment scheme backed by binary Merkle trees. This is located in the <code>merkle-tree</code> subdirectory.
24. An implementation of the Spielman-based code described in the Brakedown paper. This is located in the <code>brakedown</code> subdirectory.
25. A couple of functions to help with Lagrange interpolation. This is located in the <code>interpolation</code> subdirectory.
26. A collection of traits and implementations related to low degree tests, including a low degree test based PCS. This is located in the <code>ldt</code> subdirectory.
27. A PCS using degree 2 tensor codes, based on [BCG20](https://eprint.iacr.org/2020/1426). This is located in the <code>tensor-pcs</code> subdirectory.
28. An implementation of the Monolith-31 permutation. This is located in the <code>monolith</code> subdirectory.
29. An implementation of FRI. This is located in the <code>fri</code> subdirectory.
30. A minimal univariate STARK framework. This is located in the <code>uni-stark</code> subdirectory.
31. A minimal multivariate STARK framework. This is located in the <code>multi-stark</code> subdirectory.
32. An AIR for the Keccak-f permutation. This is located in the <code>keccak-air</code> subdirectory. I believe this is for illustration purposes only.


## Functional completeness & happy path correctness

The FRI verifier is not yet implemented; see `fri/src/verifier.rs`. The univariate STARK verifier has one of its checks commented out, with a comment that says `TODO: Re-enable when it's passing.`; see `uni-stark/src/verifier.rs`. There is no verifier module in the multivariate STARK framework. The insufficiency of functioning verifiers suggests that there may be undiscovered bugs in proving which would be flushed out in testing against a verifier.

The main correctness properties we are looking for out of Plonky3 are soundness and completeness. A proof protocol is _sound_ if and only if the verifier will not accept anything as proof of a false statement. A proof protocol is _complete_ if and only if the prover always successfully outputs a proof when given true inputs, and the verifier accepts all proofs which the prover successfully outputs.

Plonky3 consists of a collection of proof protocols and supporting libraries. The proof protocols specifically included in Plonky3 are FRI and Keccak AIR. Keccak AIR seems to serve as an example, demonstrating how one can use the libraries in Plonky3 to create new AIR-based proof protocols verified using a FRI polynomial commitment scheme (PCS). The main correctness properties we are looking for out of Plonky3 are that FRI is sound and complete, and that the proof protocols we build with Plonky3 are sound and complete when they are correctly built. These correctness properties as stated are rather vague and ambiguous; in particular, they leave undefined what it means for a protocol to be correctly built using Plonky3. The process of verifying Plonky3 should involve defining, as formally as is reasonably possible, some sufficient conditions for protocols built using Plonky3 to be sound.

Soundness cannot be tested very effectively. Tests of soundness could include providing random statements and random proofs and testing that the verifier rejects them (which it should do with overwhelming probability). Tests of soundness could also include fuzz testing, where one takes a true statement and a valid proof of it and makes random changes to the statement or the proof, checking that the verifier rejects the result (which it should do, with overwhelming probability). However, these tests are really not enough to verify soundness, nor are any tests. Soundness has to be proven.

This review did not include a concerted effort to establish that the implementations of Plonky3 are correct in their happy paths. Because of the incompleteness of the verifier implementations, there is a high likelihood that the prover implementations are wrong in some aspects, since without a complete verifier implementation, the prover implementations cannot be tested for completeness or soundness.


## Partiality

There is one instance of `todo!()` in the non-test code. It is stubbing out the implementation of `to_row_major_matrix` in the trait definition of `MatrixRows&lt;T>` in `matrix/src/lib.rs`.

There are 10 instances of `panic!()` in non-test code:



1. In the `MatrixRows&lt;T>::row` implementation for `TwoRowMatrixView&lt;'_, T>`, in `air/src/two_matrix.rs`.
2. In the `MatrixRowSlices&lt;T>::row_slice` implementation for `TwoRowMatrixView&lt;'_, T>`, in `air/src/two_matrix.rs`.
3. In the unsafe implementation of `PackedField::interleave` for `PackedBabyBearNeon` in `baby-bear/src/aarch64_neon.rs`.
4.  In the implementation of `PackedField::interleave` for `PackedMersenne31Neon` in `mersenne-31/src/aarch64_neon.rs`.
5. In the implementation of `get_max_height` in the trait definition of `Mmcs&lt;T>`, in `commit/src/mmcs.rs`.
6. In the implementation of `PackedField::interleave` for `F: Field` in `field/src/packed.rs`.
7. In the implementation of `prove` in `fri/src/prover.rs`.
8. In the implementation of `AirBuilder::is_transition_window` for `ConstraintFolder&lt;'a, F, Challenge, PackedChallenge>` in `multi-stark/src/folder.rs`.
9. In the implementation of `AirBuilder::is_transition_window` for `ProverConstraintFolder&lt;'a, SC>` in `uni-stark/src/folder.rs`.
10. In the implementation of `AirBuilder::is_transition_window` for `VerifierConstraintFolder&lt;'a, Challenge>` in `uni-stark/src/folder.rs`.

There are 9 instances of `.expect()` in non-test code:



1. In the implementation of `CanSample&lt;F>::sample` for `DuplexChallenger&lt;F, P, WIDTH>` in `challenger/src/duplex_challenger.rs`.
2. In the implementation of `CanSample&lt;F>::sample` for `HashChallenger&lt;F, H, OUT_LEN>` in `challenger/src/hash_challenger.rs`.
3. In the implementation of `AbstractExtensionField&lt;F>::from_base_slice` for `BinomialExtensionField&lt;F, D>` in `fields/src/extension/binomial_extension.rs`.
4. In the default implementation of `Field::inverse` in `field/src/field.rs`.
5. In the implementation of `sum_vecs` in `field/src/helpers.rs`.
6. In the implementation of `get_repeated` in `ldt/src/quotient.rs`.
7. In the implementation of `AbstractField::from_canonical_u64` for `Mersenne31` in `mersenne-31/src/lib.rs`.
8. In the implementation of `AbstractField::from_canonical_usize` for `Mersenne31` in `mersenne-31/src/lib.rs`.
9. In the implementation of `get_inverse` in `rescue/src/util.rs`.

There are 20 instances of `.unwrap()` in non-test code:



1. In the implementation of `batch_multiplicative_inverse` in `field/src/batch_inverse.rs`.
2. In the implementation of `Permutation::permute` for `KeccakF` in `keccak/src/lib.rs`.
3. In the implementation of `Mccs&lt;EF>::open_batch` for `QuotientMmcs&lt;F, EF, Inner>` in `ldt/src/quotient.rs`.
4. In the implementation of `Mccs&lt;EF>::verify_batch` for `QuotientMmcs&lt;F, EF, Inner>` in `ldt/src/quotient.rs`.
5. In the implementation of `Default::default` for `CosetMds&lt;F, N>` in `mds/src/coset_mds.rs`.
6. In the implementation of `apply_circulant_fft` in `mds/src/util.rs`.
7. In the implementation of `FieldMerkleTree&lt;F, DIGEST_ELEMS>::new`, in `merkle-tree/src/merkle-tree.rs`.
8. In the implementation of `FieldMerkleTree&lt;F, DIGEST_ELEMS>::root`, in `merkle-tree/src/merkle-tree.rs`.
9. In the implementation of `Mmcs&lt;P::Scalar>::verify_batch` for `FieldMerkleTreeMmcs&lt;P, H, C, DIGEST_ELEMS>`, in `merkle-tree/src/mmcs.rs`.
10. In the implementation of `Permutation::permute` for `MonolithMdsMatrixMersenne31&lt;NUM_ROUNDS>` in `monolith/src/monolith.rs`.
11. In the implementation of `Rescue&lt;F, Mds, Sbox, WIDTH>::num_rounds` in `rescue/src/rescue.rs`.
12. In the implementation of `get_alpha` in `rescue/src/util.rs`.
13. In the implementation of `get_inverse` in `rescue/src/util.rs`.
14. In the implementation of `PseudoCompressionFunction&lt;[T; CHUNK], N>::compress` for `TruncatedPermutation&lt;InnerP, N, CHUNK, WIDTH>` in `symmetric/src/compression.rs`.
15. In the implementation of `CryptographicHasher&lt;F, [F; 4]>::hash_iter` for `SerializingHasher&lt;F, Inner>` in `symmetric/serializing_hasher.rs`.
16. In the implementation of `CryptographicHasher&lt;F, [F; 8]>::hash_iter` for `SerializingHasher&lt;F, Inner>` in `symmetric/serializing_hasher.rs`.
17. In the implementation of `CryptographicHasher&lt;T, [T; OUT]>::hash_iter` for `PaddingFreeSponge&lt;P, WIDTH, RATE, OUT>` in `symmetric/src/sponge.rs`.
18. In the implementation of `optimal_wraps` in `tensor_pcs/src/reshape.rs`.
19. In the implementation of `Iterator::next` for `WrappedMatrixRow&lt;'a, T, M>` in `tensor-pcs/src/wrapped_matrix.rs`.
20. In the implementation of `prove` in `uni-stark/src/prover.rs`.

The following files contain instances of `assert!()` and/or `assert_eq!()` in non-test code:



1. `baby-bear/src/aarch64_neon.rs`
    1. In `PackedField::from_slice` for `PackedBabyBearNeon`, line 572
    2. In `PackedField::from_slice_mut` for `PackedBabyBearNeon`, line 582
2. `field/src/field.rs`
    3. In `AbstractExtensionField&lt;AF>::from_base_slice` for `AF : AbstractField`, line 309
3. `field/src/helpers.rs`
    4. In `add_vecs`, line 36
4. `keccak-air/src/columns.rs`
    5. In `Borrow&lt;KeccakCols&lt;T>>` for `[T]`, line 126
    6. In `Borrow&lt;KeccakCols&lt;T>>` for `[T]`, line 127
    7. In `Borrow&lt;KeccakCols&lt;T>>` for `[T]`, line 128
    8. In `BorrowMut&lt;KeccakCols&lt;T>>` for `[T]`, line 137
    9. In `BorrowMut&lt;KeccakCols&lt;T>>` for `[T]`, line 138
    10. In `BorrowMut&lt;KeccakCols&lt;T>>` for `[T]`, line 139
5. `matrix/src/mul.rs`
    11. In `mul_csr_dense`, line 19
6. `matrix/src/stack.rs`
    12. In `VerticalPair&lt;T, First, Second>::new`, line 14
7. `mersenne-31/src/aarch64_neon.rs`
    13. In `PackedField::from_slice` for `PackedMersenne31Neon`, line 522
    14. In `PackedField::from_slice_mut` for `PackedMersenne31Neon`, line 533
8. `mersenne-31/src/complex.rs`
    15. In `TwoAdicField::two_adic_generator` for `Mersenne31Complex&lt;Mersenne31>`, line 275
    16. In `AbstractExtensionField&lt;AF>::from_base_slice` for `Mersenne31Complex&lt;AF>` where `AF: AbstractField&lt;F = Mersenne31>`
9. `monolith/src/monolith.rs`
    17. In `MonolithMersenne31&lt;Mds, WIDTH, NUM_FULL_ROUNDS>::new` where `Mds: MdsPermutation&lt;Mersenne31, WIDTH>`, line 38
    18. In `MonolithMersenne31&lt;Mds, WIDTH, NUM_FULL_ROUNDS>::new` where `Mds: MdsPermutation&lt;Mersenne31, WIDTH>`, line 39
    19. In `MonolithMersenne31&lt;Mds, WIDTH, NUM_FULL_ROUNDS>::new` where `Mds: MdsPermutation&lt;Mersenne31, WIDTH>`, line 40
10. `poseidon/src/lib.rs`
    20. In `Poseidon&lt;F, Mds, WIDTH, ALPHA>::new`, line 40
11. `tensor-pcs/src/wrapped_matrix.rs`
    21. In `WrappedMatrix::new`, line 16
12. `uni-stark/src/prover.rs`
    22. In `prove`, line 45
13. `util/src/lib.rs`
    23. In `log2_strict_usize`, line 32

The following files contain `unsafe` blocks, which should be assumed pending further review to potentially result in partiality:



1. `baby-bear/src/aarch64_neon.rs`
2. `dft/src/butterflies.rs`
3. `dft/src/util.rs`
4. `field/src/packed.rs`
5. `goldilocks/src/lib.rs`
6. `keccak-air/src/columns.rs`
7. `matrix/src/dense.rs`
8. `mersenne-31/src/aarch64_neon.rs`
9. `monolith/src/monolith.rs`
10. `util/src/lib.rs`

The following arithmetic operations may potentially result in a division by zero, which would cause a panic:



1. In the implementation of `Matrix&lt;T>::height` for `RowMajorMatrix&lt;T>`, in `matrix/src/dense.rs`, line 179.
2. In the implementation of `Matrix&lt;T>::height` for `RowMajorMatrixViewM&lt;'_, T>` in `matrix/src/dense.rs`, line 256.
3. In the implementation of `Matrix&lt;T>::height` for `VerticallyStridedMatrixView&lt;Inner>` in `matrix/src/dense.rs`, line 16.
4. In the implementation of `Mmcs&lt;EF>::open_batch` for `QuotientMmcs&lt;F, EF, Inner>` in `ldt/src/quotient.rs`, on line 71.
5.  In the implementation of `Matrix&lt;T>::height` for `WrappedMatrix&lt;T, M>` in `tensor-pcs/src/wrapped_matrix.rs`, line 34.

The following instances of unchecked array indexing may potentially result in an index out of range error and a panic:



1. In `FieldMerkleTree&lt;F, DIGEST_ELEMS>::root`, in `merkle_tree/src/merkle_tree.rs`, line 53.
2. In `first_digest_layer`, in `merkle_tree/src/merkle_tree.rs`, line 107.
3. In `compress_and_inject`, in `merkle_tree/src/merkle_tree.rs`, lines 160, 171, 172, 188, 189, 198, and 199.
4. In `compress`, in `merkle_tree/src/merkle_tree.rs`, lines 230, 231, 241, 242.
5. In `prove`, in `uni-stark/src/prover.rs`, lines 75, 76, and 77.


## Exception handling

The Goldilocks field implementation allows for non-canonical forms, which means that field elements can be represented by 64-bit integers larger than the largest field element, with wrap-around semantics. This can lead to unexpected behaviors; for example, two instances of the same field element may have different hashes, because one is the canonical form and one is the non-canonical form.

The following arithmetic operations may potentially result in unchecked arithmetic overflow or underflow:


1. The multiplication in `PrimeField64::linear_combination_u64` for `Mersenne31`, on line 257 of `mersenne-31/src/lib.rs`.
2. The multiplication in `PrimeField64::linear_combination_u64` for `Mersenne31`, on line 259 of `mersenne-31/src/lib.rs`.
3. In the implementation of `dit_butterfly_inner` in `mersenne/src/radix_2_dit.rs`, lines 133-148.
4. In the default implementation of `TwoAdicSubgroupDft::coset_lde_batch`, on line 93 of `dft/src/traits.rs`.
5. In the default implementation of `SystematicCode::parity_len`, on line 15 of `code/src/systematic.rs`.
6. In the implementation of `interpolate_coset`, on line 55 of `interpolation/src/lib.rs`.


## Floating point in verifiers

Plonky3 uses floating point to compute `Rescue&lt;F, Mds, Sbox, WIDTH>::num_rounds`, in `rescue/src/rescue.rs`, on line 40. This function plausibly would be involved in a verifier algorithm for Plonky3 using Rescue-XLIX as its hash function.


## Asymptotic complexity

No asymptotically slow algorithms were identified in this review.


# Valida review

This review applies to commit `eddd2b031a13278bc4855dea802fbc045f1378d8` of Valida (available in the `lita-xyz` fork). This review does not include the assembler in the `assembler` subdirectory. The following Valida functional components are in scope (listed in a linearization of their dependency ordering):



1. Some macros for deriving trait implementations of `Machine`, `Borrow`, and `BorrowMut`. This is located in the `derive` subdirectory.
2. Some utility functions, located in the `util` subdirectory.
3. A dictionary of opcodes, located in the `opcodes` subdirectory.
4. A collection of traits, types, and implementations for building Valida chip AIRs, Valida VMs, and Valida zk-VM STARKs.  This is located in the `machine` subdirectory.
5. A collection of bus-related traits. This is located in the `bus` subdirectory.
6. A RAM chip implementation with an associated AIR implementation. This is located in the `memory` subdirectory.
7. A program ROM chip implementation with an associated AIR implementation. This is located in the `program` subdirectory.
8. A range checker chip implementation with an associated AIR implementation. This is located in the `range` subdirectory.
9. A CPU chip implementation with an associated AIR implementation. This is located in the `cpu` subdirectory.
10. A 32-bit ALU chip implementation with an associated AIR implementation. This is located in the `alu_u32` subdirectory.
11. A native field chip implementation with an associated AIR implementation. This is located in the `native_field` subdirectory.
12. An output chip implementation with an associated AIR implementation. This is located in the `output` subdirectory.
13. A basic Valida zk-VM implementation and a wrapper to run its prover. This is located in the `basic` subdirectory.
14. A verifier stub. This is located in the `verifier` subdirectory.


## Functional completeness & happy path correctness

The verifier is not implemented; see `verifier/src/lib.rs`.

The DIV32 AIR for the ALU_U32 AIR is not implemented; see `alu_u32/src/div/stark.rs`.

The implementation of `Chip&lt;M>::localSends` for `Mul32Chip` is stubbed out, in `alu_u32/src/mul/mod.rs`, line 84.

The constraints for immediate values are missing a range check on `read_value_2`, in the implementation of `Air&lt;AB>::eval` in `cpu/src/stark.rs`; see line 41.

Not all parts of the transcript are observed by the challenger. This can result in soundness bugs. See `derive/src/lib.rs`, line 253 and line 277.

The implementation of `PermutationAirBuilder::permutation_randomness` for `ConstraintFolder&lt;'a, F, EF, M>` is stubbed out; see `machine/src/__internal/folding_builder.rs`, line 32.

The implementation of `AirBuilder::assert_zero` for `ConstraintFolder&lt;'a, F, EF, M>` is stubbed out; see `machine/src/__internal/folding_builder.rs`, line 88.

The implementation of `prove` in `machine/src/__internal/prove.rs` is stubbed out.

The range checker chip STARK is not implemented; see `range/src/stark.rs`.

A comment in `memory/src/stark.rs` says `FIXME: Degree constraint 4, need to remove`. It’s not clear to me why a degree 4 constraint needs to be removed. Is this an optimization or a requirement for functional completeness and correctness?

The default extension field for `valida-derive` is the Baby bear field, which is also the default field. The comment on it says `FIXME: Replace`. See `machine/src/__internal/mod.rs`, line 4. This may not matter, though, if the defaults are never used.

There are concerns around the soundness of the memory argument; see [Valida issue #40](https://github.com/valida-xyz/valida/issues/40).

The main correctness properties we are looking for out of Valida are its soundness and completeness. The Valida verifier should not accept anything as proof of a false claim. Also, for all true statements of the form “Valida program _p_ has output _x _on some input” which are verified by an execution trace within the size limitations of the SNARK, the prover can construct a Valida SNARK of that statement, which the Valida verifier accepts.

As with Plonky3, this review made no concerted effort to check that Valida is correct in its happy path executions. Testing soundness and completeness would require a verifier. Testing soundness is not really achievable, and at the end of the day soundness needs to be proven.


## Partiality

There are three instances of `panic!()` , excluding macro code which is only executed at compile-time, and code which is only used for debugging:



1. In the quoted code for `run` in the implementation of the `run_method` macro in `derive/src/lib.rs`.
2. In the implementation of `AirBuilder::is_transition_window` for `ConstraintFolder&lt;'a, F, EF, M>` in `machine/src/__internal/folding_builder.rs`.
3. In the implementation of `MemoryChip::read`, in `memory/src/lib.rs`.

There are no instances of `.expect()` in non-test code, except for some instances in macro code which runs exclusively at compile time. It is okay for code which runs exclusively at compile time to be partial, as long as it terminates.

There are 16 instances of `.unwrap()` in non-test-code, excluding macro code which runs exclusively at compile time:



1. In the implementation of `main` in `basic/src/bin/valida.rs`, on the call to `load_program_rom`.
2. In the implementation of `main` in `basic/src/bin/valida.rs`, on the call to `std::io::stdin().read_to_end`.
3. In the implementation of `main` in `basic/src/bin/valida.rs`, on the call to `std::io::stdout().write_all`.
4. In the implementation of `check_constraints` in `machine/src/__internal/check_constraints.rs`, on line 30.
5. In the implementation of `check_constraints` in `machine/src/__internal/check_constraints.rs`, on line 39.
6. In the implementation of `check_constraints` in `machine/src/__internal/check_constraints.rs`, on line 44.
7. In the implementation of `check_cumulative_sums` in `machine/src/__internal/check_constraints.rs`, on line 90.
8. In the implementation of `generate_permutation_trace` in `machine/src/chip.rs`, on line 40.
9. In the implementation of `generate_permutation_trace` in `machine/src/chip.rs`, on line 169.
10. In the implementation of `generate_permutation_trace` in `machine/src/chip.rs`, on line 189.
11. In the implementation of `eval_permutation_constraints` in `machine/src/chip.rs`, on line 268.
12. In the implementation of `eval_permutation_constraints` in `machine/src/chip.rs`, on line 270.
13. In the implementation of `MemoryChip::insert_dummy_reads` in `memory/src/lib.rs`, on line 257.
14. In the implementation of `MemoryChip::insert_dummy_reads` in `memory/src/lib.rs`, on line 258.
15. In the implementation of `MemoryChip::insert_dummy_reads` in `memory/src/lib.rs`, on line 259.
16. In the implementation of `Instruction&lt;M>::execute` for `WriteInstruction` in `output/src/lib.rs`, on line 154.

There are 4 instances of `assert_eq!()` in non-test code which is run after compile time:



1. In the implementation of `check_constraints` in `machine/src/__internal/check_constraints.rs`.
2. In the implementation of `check_cumulative_sums` in `machine/src/__internal/check_constraints.rs`.
3. In the implementation of `Instruction&lt;M>::execute` for `WriteInstruction` in `output/src/lib.rs`, on lines 162 and 163.

There is one instance of `assert!()` in non-test code which runs after compile time:



1. In the default implementation of `read_word` in the `MachineWithProgramChip` trait definition, in `program/src/lib.rs`.

The following files contain `unsafe` blocks, which should be assumed pending further review to potentially result in partiality:



1. `alu_u32/src/add/columns.rs`
2. `alu_u32/src/add/mod.rs`
3. `alu_u32/src/bitwise/columns.rs`
4. `alu_u32/src/bitwise/mod.rs`
5. `alu_u32/src/div/columns.rs`
6. `alu_u32/src/lt/columns.rs`
7. `alu_u32/src/lt/mod.rs`
8. `alu_u32/src/mul/columns.rs`
9. `alu_u32/src/shift/columns.rs`
10. `alu_u32/src/shift/mod.rs`
11. `alu_u32/src/sub/columns.rs`
12. `alu_u32/src/sub/mod.rs`
13. `cpu/src/columns.rs`
14. `cpu/src/lib.rs`
15. `derive/src/lib.rs`
16. `memory/src/columns.rs`
17. `memory/src/lib.rs`
18. `native_field/src/columns.rs`
19. `native_field/src/lib.rs`
20. `output/src/columns.rs`
21. `output/src/lib.rs`
22. `program/src/columns.rs`
23. `range/src/columns.rs`
24. `range/src/lib.rs`

The following arithmetic operations may result in division by zero and a panic:



1. In the implementation of `Div::div` for `Word&lt;u8>`, in `machine/src/core.rs`, line 87.
2. In the implementation of `MemoryChip::insert_dummy_reads` in `memory/src/lib.rs`, line 222.
3. In the implementation of `Instruction&lt;M>::execute` for `Div32Instruction`, in `alu_u32/src/div/mod.rs`, line 89.

The following unchecked array indexes may result in an out of bounds error and a panic:



1. In the implementation of `generate_permutation_trace` in `machine/src/chip.rs`, lines 120, 166, 179, 182, and 189.
2. In the implementation of `generate_rlc_elements` in `machine/src/chip.rs`, lines 285 and 300.
3. In the implementation of `CpuChip::set_instruction_values` in `cpu/src/lib.rs`, line 248.
4. In the implementation of `CpuChip::pad_to_power_of_two` in `cpu/src/lib.rs`, lines 328, 329, 330, 346, 347, 348, 351, 352, 355, 356, and 357.

The zk-VM `run` method is capable of non-terminating; see `derive/src/lib.rs`, lines 158-176. This is to be expected, because Valida is Turing complete. Perhaps, however, we should code in a maximum number of steps, in order to avoid partiality. We can even pass in the maximum number of steps as an input to the method.


## Exception handling

The following arithmetic operations may result in unchecked overflow, resulting in different behavior in debug vs release compilation mode. For each of these, the desired resolution is likely (a) the overflow should wrap around, and this should be made explicit and made to happen in both debug and release mode, or (b) overflow is deemed to be impossible for all inputs, and should result in a panic with an informative error message. However, other resolutions are possible and these must be considered on a case by case basis.



1. In `machine/src/core.rs`:
    1. In the implementation of `Add:add` for `Word&lt;u8>`, on line 57
    2. In the implementation of `Sub::sub` for `Word&lt;u8>`, on line 67
2. In `alu_u32/src/add/mod.rs`:
    3. In the implementation of `Add32Chip::op_to_row`:
        1. The addition on line 108
        2. The first addition on line 109
        3. The second addition on line 109
        4. The first addition on line 113
        5. The second addition on line 113
    4. In the implementation of `Instruction&lt;M>::execute` for `Add32Instruction`:
        6. The addition on line 141 (computing `read_addr_1`)
        7. The addition on line 142 (computing `write_addr_1`)
        8. The addition on line 149 (computing `read_addr_2`)
        9. The addition on line 153 (computing `a`)
3. In `cpu/src/lib.rs`:
    5. In `Instruction&lt;M>::execute` for `ReadAdviceInstruction`:
        10. The two additions on line 410
        11. The multiplication on line 410
    6. In `Instruction&lt;M>::execute` for `WriteAdviceInstruction`:
        12. The first and second additions on line 436 (i.e., `fp + mem_addr`)
        13. The third addition on line 436 (i.e., `(fp + mem_addr) + mem_buf_len`)
    7. In `Instruction&lt;M>::execute` for `Load32Instruction`:
        14. The addition on line 470 (computing `read_addr_1`)
        15. The addition on line 472 (computing `write_addr`)
    8. In `Instruction&lt;M>::execute` for `Store32Instruction`:
        16. The addition on line 491 (computing `read_addr`)
        17. The addition on line 492 (computing `write_addr`)
    9. In `Instruction&lt;M>::execute` for `JalInstruction`:
        18. The addition on line 512 (computing `write_addr`)
        19. The addition on line 513 (computing `next_pc`)
        20. The addition on line 518
    10. In `Instruction&lt;M>::execute` for `JalvInstruction`:
        21. The addition on line 536 (computing `write_addr`)
        22. The addition on line 537 (computing `next_pc`)
        23. The addition on line 540 (computing `read_addr`)
        24. The addition on line 543 (computing `read_addr`)
        25. The additive assignment on line 545
    11. In `Instruction&lt;M>::execute` for `BeqInstruction`:
        26. The addition on line 562 (computing `read_addr_1`)
        27. The addition on line 570 (computing `read_addr_2`)
        28. The addition on line 576
    12. In `Instruction&lt;M>::execute` for `BneInstruction`:
        29. The addition on line 594 (computing `read_addr_1`)
        30. The addition on line 602 (computing `read_addr_2`)
        31. The addition on line 608
    13. In `Instruction&lt;M>::execute` for `Imm32Instruction`:
        32. The addition on line 624 (computing `write_addr`)
    14. Every instance of `state.cpu_mut().pc += 1;`
    15. In `impl CpuChip`:
        33. On line 655 and 660, `self.pc += 1;`
        34. On line 668, `self.clock += 1;`
4. In `alu_u32/src/bitwise/mod.rs`:
    16. In `Instruction&lt;M>::execute` for `Xor32Instruction`:
        35. The addition on line 145 (computing `read_addr_1`)
        36. The addition on line 146 (computing `write_addr`)
        37. The addition on line 153 (computing `read_addr_2`)
    17. In `Instruction&lt;M>::execute` for `And32Instruction`:
        38. The addition on line 181 (computing `read_addr_1`)
        39. The addition on line 182 (computing `write_addr`)
        40. The addition on line 190 (computing `read_addr_2`)
    18. In `Instruction&lt;M>::execute` for `Or32Instruction`:
        41. The addition on line 217 (computing `read_addr_1`)
        42. The addition on line 218 (computing `write_addr`)
        43. The addition on line 225 (computing `read_addr_2`)
5. In `alu_u32/src/div/mod.rs`, in `Instruction&lt;M>::execute` for `Div32Instruction`:
    19. The addition on line 77 (computing `read_addr_1`)
    20. The addition on line 78 (computing `write_addr`)
    21. The addition on line 85 (computing `read_addr_2`)
6. In `alu_u32/src/lt/mod.rs`, in `Instruction&lt;M>::execute` for `Lt32Instruction`:
    22. The addition on line 128 (computing `read_addr_1`)
    23. The addition on line 129 (computing `write_addr`)
    24. The addition on line 136 (computing `read_addr_2`)
7. In `alu_u32/src/mul/mod.rs`, in `Instruction::execute` for `Mul32Instruction`:
    25. The addition on line 123 (computing `read_addr_1`)
    26. The addition on line 124 (computing `write_addr`)
    27. The addition on line 131 (computing `read_addr_2`)
    28. The multiplication on line 135 (computing `a`)
8. In `alu_u32/src/shift/mod.rs`:
    29. In `Instruction&lt;M>::execute` for `Shl32Instruction`:
        44. The addition on line 172 (computing `read_addr_1`)
        45. The addition on line 172 (computing `write_addr`)
        46. The addition on line 180 (computing `read_addr_2`)
    30. In `Instruction&lt;M>::execute` for `Shr32Instruction`:
        47. The addition on line 216 (computing `read_addr_1`)
        48. The addition on line 217 (computing `write_addr`)
        49. The addition on line 224 (computing `read_addr_2`)
9. In `alu_u32/src/sub/mod.rs`, in `Instruction&lt;M>::execute` for `Sub32Instruction`:
    31. The addition on line 137 (computing `read_addr_1`)
    32. The addition on line 138 (computing `write_addr`)
    33. The addition on line 145 (computing `read_addr_2`)
    34. The subtraction on line 149 (computing `a`)
10. In `native_field/src/lib.rs`:
    35. In `Instruction&lt;M>::execute` for `AddInstruction`:
        50. The addition on line 157 (computing `read_addr_1`)
        51. The addition on line 158 (computing `write_addr`)
        52. The addition on line 165 (computing `read_addr_2`)
    36. In `Instruction&lt;M>::execute` for `SubInstruction`:
        53. The addition on line 197 (computing `read_addr_1`)
        54. The addition on line 198 (computing `write_addr`)
        55. The addition on line 205 (computing `read_addr_2`)
    37. In `Instruction&lt;M>::execute` for `MulInstruction`:
        56. The addition on line 237 (computing `read_addr_1`)
        57. The addition on line 238 (computing `write_addr`)
        58. The addition on line 245 (computing `read_addr_2`)
11. In `output/src/lib.rs`:
    38. In `Chip&lt;M>::generate_trace` for `OutputChip`:
        59. The subtraction on line 66 (computing `cols.diff`)
    39. In `Instruction&lt;M>::execute` for `WriteInstruction`:
        60. The addition on line 149 (computing `read_addr_1`)


## Floating point in verifiers

This review identified no potential issues with floating point in verifiers in the Valida code base.


## Asymptotic complexity

The following algorithms are asymptotically slow:



1. The implementation of `Chip&lt;M>::generate_trace` for `RangeCheckerChip&lt;MAX>`, in `range/src/lib.rs`, is `O(MAX * log(|self.count|))` but can be done in `O(|self.count| * log(|self.count|))`. In this context, `self.count` is a `BTreeMap&lt;u32, u32>` whose key space is `[0, MAX)`. This means that `|self.count|`, the cardinality or number of keys of `self.count`, is less than or equal to `MAX`, and thus the algorithm is asymptotically slow, as noted by the comment.


__________________________________

_If you’re interested in further discussions on this topic or working together on this subject, please reach out to us at research@delendum.xyz._
