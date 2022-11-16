---
layout: post
author: Ole Spjeldnæs
title: "zk-SNARK Aggregation"
excerpt: "Popular zk-SNARK aggregation constructions and some potentially interesting approaches"
---

By Delendum Research and ETH Zurich

### Table of Content

- [Introduction](#introduction)
- [Popular zk-SNARK aggregation constructions](#popular-zk-snark-aggregation-constructions)
  * [Halo](#halo)
  * [Plonky2](#plonky2)
  * [Nova](#nova)
  * [SnarkPack](#snarkpack)
- [Some potentially interesting approaches](#some-potentially-interesting-approaches)
  * [PlonkPack](#plonkpack)
  * [Multilinear Polynomials](#multilinear-polynomials)

## Introduction

When comparing zk-SNARKs, we are usually primarily interested in three
different metrics: prover time, verifier time, and proof length -
some may also want to add quantum security and setup trust assumptions
to that list.

Within the context of blockchains, there is one further criterion which
I would argue to be as important as those three, and we'll give it the
clunky name *aggregation-friendliness*.

A zk-SNARK with universal setup can be said to be aggregation-friendly
if (informally) there exists some m so that it is easy to merge m proofs
$$P_{1},...,P_{m}$$ into one proof $$P_{1,...,m}$$ so that $$P_{1,...,m}$$ is
not much bigger than either $$P_{i}$$, and certainly much smaller than the
two of them together. Specifically there should exist a deterministic
polynomial-time algorithm $$\mathfrak{A}$$ (which we call the aggregator)
and $$\mathfrak{V}_{\mathfrak{A}}$$ (which we call the aggregation
verifier) such that for any $$P_{1},...,P_{m}$$:

-   The aggregation time $$|\mathfrak{A}(P_{1},...,P_{m})|$$ is
    (quasi-)linear in m

-   The proof size $$|P_{1,...,m}|$$ is sublinear in m and quasilinear in
    $$\max_{i}(\|P_{i}\|)$$

-   $$|\mathfrak{V}_{\mathfrak{A}}(P_{1,...,m})|$$ is quasilinear in
    $$|\mathfrak{V}({argmax}_{i}(|P_{i}|))|$$, where
    $$\mathfrak{V}$$ is the verifier of the given zk-SNARK

So why is aggregation-friendliness so important to us? In blockchains,
especially for scalability purposes, SNARKs are used to prove the
validity of a bunch of statements at once. Now, if we want to prove a
block of n transactions of, say for simplicity, the same size m, with a
single zk-SNARK, the prover time will typically be O($$nm\log nm$$). Often
both proof length and verifier time are also non-constant in n. Bigger n
therefore leads to significantly bigger costs, and furthermore, this
doesn't really scale with hardware parallelization or multiple provers.

On the other hand, imagine if the same proof system were
aggregation-friendly, and let us for now ignore network limitations. You
could have n different provers, each proving one transaction in time
O($$m\log m$$), and then aggregating them in time sublinear in n,
therefore generating a block proof much more quickly. Hopefully the
resulting proof would also not be much bigger than the original one or
much more difficult to verify.

To hammer the point home, assume we have two schemes,
$$S_{1}:=(\mathcal{P}_{1}, \mathcal{V}_{1}), S_{1}:=(\mathcal{P}_{2}, \mathcal{V}_{2})$$.
$$\mathcal{P}_{1}$$ is more efficient that $$\mathcal{P}_{2}$$, with the
respective proving times for a statement of length m being
$$t_{1} = 5m\log_{2}m, t_{2} = 10m\log^{2}_{2}m$$. However, while $$S_{2}$$
is aggregation-friendly, $$S_{1}$$ is not. There exists an aggregator
$$\mathcal{A}$$ for $$S_{2}$$ which can aggregate n proofs of a length-m
statement in time $$4\log^{2}_{2}n$$. In total, the proving time for
$$P_{1}$$ will be $$5nm\log_{2}nm$$. On the other hand, we can run $$P_{2}$$
in parallel for each of the statements, and then run $$\mathcal{A}$$ to
aggregate them into a single proof, for a total time of

$$10m\log^{2}_{2}m\cdot 4\log^{2}_{2}n = 40m\log^{2}_{2}m\log^{2}_{2}n$$

Let's look at how $$t_{1}$$ and $$t_{2}$$ compare for some different values
of n and m:

-   For n = 2, m = 2, we have $$t_{1}=40, t_{2}=80$$

-   For n = 8, m = 4, we have $$t_{1} = 800, t_{2} = 5760$$

-   For n = 4096, m = 16, we have
    $$t_{1} = 5242880, t_{2} = 1474560 \simeq 0.28\cdot t_{1}$$

Here we see that, although $$\mathcal{P}_{1}$$ performs significantly
better than $$\mathcal{P}_{2}+\mathcal{A}$$ for small n, it's a very
different story for big n, and should make it clear why
aggregation-friendliness is important.

This post will focus on some different approaches to zk-SNARK proof
aggregation. At a high level, there are three axes which divide them
into categories:

-   **Recursive/Batching**: Recursive proofs are proofs of proofs. We
    refer to non-recursive aggregation methods as \"batching\"

-   **Sequential/Parallel**: Can the scheme be executed in parallel?

-   **Algebraic/Arithmetization-based**: This distinction will become
    clear once we look at some examples.

## Popular zk-SNARK Aggregation Constructions

### Halo

-   Recursive

-   Sequential

-   Algebraic

As so much of cryptography, zero-knowledge and otherwise, Halo is based
on elliptic curves. Specifically, it uses a polynomial commitment scheme
(PCS), which was previously used in another well-known zk-SNARK scheme,
namely *Bulletproofs*: Given an elliptic curve E (where we write
$$E(\mathbb{F})$$ for the group defined by the points on E in the field
$$\mathbb{F}$$ over which we work) and random points
$$P_{i}\in E(\mathbb{F})$$, one can commit to a polynomial p(x) =
$$\sum_{i}a_{i}x^{i}$$ with the element $$\sum_{i}a_{i}\cdot P_{i}$$, where
$$\cdot$$ denotes scalar multiplication in the additive group
$$E(\mathbb{F})$$. This commitment is also often written as
$$<\textbf{a},\textbf{P}>$$ as if it were an inner product.

The \"inner product argument\" (IPA) is a protocol between the prover
and the verifier where the prover tries to convince the verifier that he
knows the polynomial corresponding to his commitment. Roughly speaking,
he does this by iteratively chopping his tuples **a** and **P** in two
equally long tuples and then putting them together so as to get two new
tuples of half the length. Eventually, after a logarithmic number of
rounds, the prover is left with a single equation, where, almost if and
only if he has acted honestly throughout will he be able to satisfy the
verifier's final challenge.

The problem is that, although the number of rounds is nice and small,
the verifier still needs to compute two \"inner products\" of
\"vectors\" of the original length. One may think that this ruins the
protocol: the verifier's work is now linear rather than logarithmic.
However, this is where the recursive proof composition (aggregation)
comes in:

The aggregation technique largely relies on an ad hoc polynomial
g$$_{u_{1},...,u_{k}}$$(x), which is defined in terms of the verifier's
challenges $$u_{i}$$ to the prover in the IPA. As it turns out,
g$$_{u_{1},...,u_{k}}$$(x) can be used to compute both of the \"inner
products\", where we will focus on one of them: At the end of the IPA,
the prover has a single elliptic curve point P. As it turns out, by the
construction of g$$_{u_{1},...,u_{k}}$$(x), the commitment to it defined
by the above scheme yields exactly P! The other \"inner product\" can
also be defined as an evaluation of g$$_{u_{1},...,u_{k}}$$(x). This does
not help us directly, as evaluating the polynomial still takes linear
time, however, the prover can now create such polynomials for many
statements, commit to them, and then at the end prove that a linear
combination of them corresponds to a desired value - this uses a
property of the PCS which we have not mentioned thus far, namely that
it's additively homomorphic (specifically, it's based on Pedersen
commitments).

To recap, the verifier previously had to do a linear-time operation for
each statement. By invoking a certain polynomial with some nice
properties, the prover can defer the final opening proof until he has
computed and committed to arbitrarily many such polynomials. Halo people
often talk about the \"inner\" and the \"outer\" circuit to reflect the
distinction between the point of view of the prover (who finishes most
of each proof before computing an aggregated proof for all together) and
the verifier (for whom the proof really boils down to the linear-time
computation she has to perform).

For specifics, we refer to [the original paper](https://eprint.iacr.org/2019/1021). The Halo approach is clearly algebraic in its nature: It depends on the nice homomorphic
properties of Pedersen commitments, as well as a particular polynomial.
Moreover, it is sequential: The prover iteratively adds commitments
$$G_{1}, G_{2},...$$ to his inner proof, before opening a linear
combination of them at the end of the protocol. Finally, it is
recursion-based: The prover proves the validity of the previous inner
proof to the current one.

Halo 2 is used by several protocols, and its main selling point is the
aggregation property. Notably, Mina protocol uses Halo to create a
constant-sized blockchain, in the sense that the validity of the current
state is proven simply by verifying the validity of the proof of the
previous state together with the transactions in the last block.
Finally, a proof is computed, representing everything that has happened
up to that state.

The Electric Coin Company, the developers of the scheme, recently
incorporated Halo 2 as part of their upgrade to the Orchard shielded
pools.

Halo 2 is also used by the zkEVM rollup project Scroll, in order to
aggregate multiple blocks into one final proof and thereby amortize the
L1 verification costs between them.

## Plonky2

-   Recursive

-   Batching

-   Arithmetization-based

Plonky2 emphasizes the modular nature of zk-SNARKs:
We need some arithmetization scheme, a PCS, a zero test, and a
consistency check between the arithmetization polynomials and the
constraint polynomial.

The goal of the scheme is to make recursive proving extremely fast, i.e.
for two (or more) Plonky2 proofs, proving the validity of them with a
single proof should be a small computation.

As an aside, why not simply use PLONK? While an elliptic curve E defined
over some finite field $$\mathbb{F}_{q}$$ can have q points over
$$\mathbb{F}_{q}$$, those curve/field combinations are not appropriate
within the setting of cryptography. Instead, say E($$\mathbb{F}_{q})$$ has
r elements for some prime r. Then the arithmetic of the group is defined
over $$\mathbb{F}_{r}$$, a different field altogether. This means that for
a proving scheme for $$\mathbb{F}_{q}$$-arithmetic satisfiability, the
verifier has to perform arithmetic over $$\mathbb{F}_{r}$$. This means
that the recursive proof must use a circuit defined over
$$\mathbb{F}_{r}$$, the recursive proof of that a different field, and so
on. Halo in fact heavily relies on the existence of 2-cycles of curves,
namely tuples (E,q), (E',r) such that q and r are primes and
$$\#E(\mathbb{F}_{q})=r, \#E'(\mathbb{F}_{r})=q$$.

Back to Plonky2. Because of the aforementioned issues with elliptic
curves, the scheme essentially takes advantage of the flexibility of the
so-called Plonkish arithmetization, while using a PCS which doesn't rely
on elliptic curves. Specifically, Plonky2 replaces the KZG-based proof
with FRI. We will only briefly describe FRI here:

For any univariate polynomial f(x) $$\in \mathbb{F}[x]$$, we can consider
the polynomials $$f_{even}$$(x) and $$f_{odd}(x)$$ defined by the
coefficients of the even and odd powers of x in f(x), respectively. Note
also that the degree of both of those polynomials is at most half the
degree of f and that $$f(x) = f_{even}(x^{2}) + x\cdot f_{odd}(x^{2})$$

Roughly speaking, we can iteratively commit to polynomials\
$$f(x) = f^{(0)}(x),...,f^{(\log deg f)}(x)$$ such that deg $$f^{(i)} \leq$$
deg $$f^{(i+1)}/2$$ by evaluating them over certain appropriate domains
and Merkle-committing to the evaluations.

For the verifier, and hence the construction of recursive proofs, the
main computational task lies in computing hashes to verify Merkle paths.
The high-level idea is that while for vanilla PLONK complex arithmetic
expressions (such as hash functions) lead to deep circuits, it's
possible to implement custom arithmetic gates which themselves compute
more complex expressions than addition and multiplication.

Plonky2 implements custom gates which ensure that the verifier circuit
is shallow, as well as some other engineering optimizations, in order to
make recursive proving as fast as possible. Plonky2 was created by the formerly Mir Protocol, now Polygon Zero team.

## Nova

-   Recursive

-   Sequential

-   Arithmetization-based + Algebraic

Unlike the two preceding schemes, Nova is not universal; it can only be
used to incrementally verify computations $$y^{(i)}=f^{(i)}(x)$$. The
authors themselves consider it as a \"Halo taken to the extreme\".
Unlike Halo, each step of Nova does not output a zk-SNARK, but a more
minimal construction, and the authors have dubbed this technique
incrementally verifiable computation (IVC). Nova is indeed faster than
Halo, but this also has the consequence that not any arbitrary output is
suitable within every context (as is the case for Halo), and the idea of
IVC is really, as the name suggests, to prove the validity of n
repetitions of the same computation on the same input and not care about
any individual step i $$<$$ n.

Nova uses a modified R1CS arithmetization which allows the prover to
\"fold\" two instances (one representing the recursive \"proof\", one
representing the current step of the computation) into one. The prover
ultimately proves the validity of the final folded R1CS instance with a
zk-SNARK, which, thanks to the properties of the IVC scheme, proves the
validity of all n invocations of f.

Although not appropriate for most applications we're interested in
within a blockchain context, we have included Nova because the idea of
IVC is nevertheless interesting: Take some half-baked proofs, which by
themselves are neither zero-knowledge nor succinct, then aggregate and
construct a single proof at the end.

## SnarkPack

-   Batching

-   Parallel

-   Algebraic

The schemes we have seen up to this point have been zk-SNARKs optimized
for aggregation, i.e. the aggregation techniques are essentially baked
into the respective protocols. SnarkPack takes a different approach: It is
constructed as an addition to an already existing zk-SNARK, namely
Groth16. In this sense, SnarkPack is the first (and only) true
*aggregation scheme* covered here.

Groth16 uses the popular KZG polynomial commitment scheme, and the
verification of the proof includes checking an equation of the form
$$e(A,B) = Y + e(C,D)$$ for an elliptic curve pairing
$$e: E_{1}(\mathbb{F})\times E_{2}(\mathbb{F})\rightarrow E_{pairing}(\mathbb{F})$$
and points\
$$A,C\in E_{1}(\mathbb{F}), B,D \in E_{2}(\mathbb{F})$$, and
$$Y\in E_{pairing}(\mathbb{F})$$ ($$E(\mathbb{F})$$ here denotes the group
of points (x,y) on the elliptic curve E with $$x,y\in \mathbb{F}$$). An
elliptic curve is a bilinear non-degenerate map, where the bilinearity
is most important to us. To see why, imagine for a moment Y=0,
$$A_{1}, A_{2}, C_{1}, C_{2}\in E_{1}(\mathbb{F})$$, and we want to check
the validity of the equation $$e(A_{i},B) = e(C_{i},D), i=1,2$$ 

One way of verifying this equation is to pick a random r $$\in\mathbb{F}$$ and
check 

$$\label{eq:ec_1}
    e(A_{1},B) + re(A_{2},B) = e(C_{1},D) + re(C_{2},D)$$ 
    
Now, bilinearity of e implies that the left side of the equation can be
rewritten as $$e(A_{1}+rA_{2},B)$$ and likewise for the right side, so
that we have reduced to $$e(A_{1}+rA_{2},B) = e(C_{1}+rC_{2},D)$$
Importantly, we only have to compute two elliptic curve pairings rather
than four (pairings are expensive).

For m proofs we can imagine the same approach yielding even greater
benefits. Now, the above toy example does not fully apply to Groth16; in
particular Y cannot be assumed to be 0, and B does not remain constant
for different i.

For m Groth16 proofs $$\pi_{i}=(A_{i},B_{i},C_{i})$$, the SnarkPack
aggregation scheme computes $$Y_{i}=Y(\pi_{i})$$ as according to Groth16,
and samples the random element r as in our toy example. 

Denote by

$$C^{(m)} := \sum_{i=0}^{m-1}r^{i}C_{i}{\;and\;} Y^{(m)} := \sum_{i=0}^{m-1}r^{i}Y_{i}.$$

The aggregated equation to be checked is of the form

$$\sum_{i=0}^{m-1}r^{i}e(A_{i},B_{i}) = Y^{(m)}+C^{(m)}$$ 

This looks very linear, and therefore not all that nice, however, the prover can
use the IPA technique also used in Halo, in order to ensure that the
proof size as well as the verifier time are logarithmic in m.

## Some Potentially Interesting Approaches

### PlonkPack

-   Batching

-   Parallel

-   Algebraic

**Since writing this section, We have come across the brand new paper
aPlonk, which is essentially SnarkPack for PLONK. Nevertheless, the idea here is slightly different, and so it should still be worth exploring.**

SnarkPack is Groth16-based, and therefore not relevant for applications
relying on universal SNARKs, such as blockchain execution layers. There
are some problems with adopting that kind of approach to KZG-based
universal zk-SNARKs: For one, most of the ones we have are IOPs,
in practice meaning that they rely on random challenges from the
verifier. Each proof will have different challenges contained within it,
and this is a problem for aggregation schemes relying on structural
properties of e.g. the elliptic curve pairing in KZG.

That said, we can think of aggregation within different contexts:

1.  In SnarkPack, one can simply take n proofs and aggregate them.

    Note: for SnarkPack, specifically, the circuit has to be the same
    for all because Groth16 is non-universal. We are focused on schemes
    for aggregating proofs of arbitrary circuits into one.

2.  One might imagine a synchronous scheme where n provers with n
    statements to prove, build an aggregated proof together. After each
    prover has completed a given round, the verifier sends the same
    random challenge to all of them

(1) is the SnarkPack approach, and it is ideal in the sense that a
scheme like this offers the most flexibility and universality.

(2) will generally entail significant communication costs, however we can
imagine each prover simply as a core on a GPU. In this case, the
approach may well make sense (this is the approach taken by the
aPlonk authors as well). This is also very much relevant to today's
problems, as all current zk-rollups have one prover per block, i.e.
there's not a decentralized network of provers each proving the validity
of individual transactions, and there likely won't be for a long time.

For an approach based on homomorphism similar to SnarkPack, then, it's
natural to look at PLONK, which within our framework can be seen as a
universal counterpart to Groth16 - the main innovation of PLONK
arguably lays in its arithmetization, which we consider to be irrelevant
for this kind of aggregation scheme.

As we have seen, in PLONK, the goal is to prove the validity of some
equation of the form: $$P(x) \equiv 0 { mod } Z_{H}(x)$$ For
m such polynomials $$P_{1}(x),...,P_{m}(x)$$. For linearly independent
elements $$\alpha_{i}$$, roughly speaking, for each $$P_{i}(x)$$ the
corresponding such equation holds if and only if:
$$\sum_{i}\alpha_{i}P_{i}(x)\equiv 0 { mod } Z_{H}(x)$$ In
practice it's not that easy, as we do not get these nice, linear
relations. Specifically, for vanilla PLONK, the equation to be proved is
of the form: 

$$\label{eq:plonk}
    \omega_{L}(x)q_{L}(x) + \omega_{R}(x)q_{R}(x) \\ + \omega_{L}(x)\omega_{R}(x)q_{M}(x) + \omega_{O}(x)q_{O}(x) + q_{C}(x)\equiv 0 {\,mod\,} Z_{H}(x)$$

where the $$q_{I}$$(x) define the circuit and the $$\omega_{J}$$(x)
constitute the witness. Assuming, then, that all those polynomials are
defined such that we have $$\alpha_{i}q_{I,i}(x)$$ and
$$\alpha_{i}\omega_{J,i}(x)$$, the aggregated polynomial is no longer
equal to 0 modulo the vanishing polynomial. Specifically, the left side
of the previous equation now looks like 

$$\begin{split}
    \alpha_{i}\omega_{L,i}(x)\alpha_{i}q_{L,i}(x) + \alpha_{i}\omega_{R,i}(x)\alpha_{i}q_{R,i}(x) \\ +\, \alpha_{i}\omega_{L,i}(x)\alpha_{i}\omega_{R,i}(x)\alpha_{i}q_{M,i}(x) + \alpha_{i}\omega_{O,i}(x)\alpha_{i}q_{O,i}(x) + \alpha_{i}q_{C,i}(x) = \\
    \alpha_{i}^{2}\omega_{L,i}(x)q_{L,i}(x) + \alpha_{i}^{2}\omega_{R,i}(x)q_{R,i}(x) \\ +\, \alpha_{i}^{3}\omega_{L,i}(x)\omega_{R,i}(x)q_{M,i}(x) +
    \alpha_{i}^{2}\omega_{O,i}(x)q_{O,i}(x) + \alpha_{i}q_{C,i}(x)
    \end{split}$$ 
    
where the problem is that we have different powers of
$$\alpha_{i}$$ for the different terms. This is likely not an unsolvable
problem; for example, the \"circuit aggregator\" could simply multiply
the various $$q_{I,i}(x)$$ with powers of $$\alpha_{i}$$, so that this
ultimately ends up as an equation of the form

$$\sum_{i}\alpha_{i}^{k}P_{i}(x)\equiv 0\, { mod }\, Z_{H}(x)$$
for some k $$>$$ 1.

This does raise another issue, namely that the set $$\{a_{i}^{k}\}_{i,k}$$
cannot be assumed to be linearly independent. Setting
$$\alpha_{i}:=L_{i}(y)$$ for the Lagrange polynomials $$L_{i}(y)$$ defined
on some set of size m easily solves this, as the set
$$\{L_{i}^{k}(y)\}_{i,k}$$ is linearly independent. It might also provide
some geometric intuition.

We are fairly certain that this approach will work with some tweaking. Of course, PLONK is not as simple as proving this one equation directly, and one does encounter
some issues down the road.

### Multilinear Polynomials

-   Recursive

-   Parallel

-   Arithmetization-based

We observe that multilinear polynomials will become increasingly
popular for arithmetization, in particular with the recent release of
HyperPlonk, which has some exciting properties such as the possibility for high-degree custom gates. The main benefit of multilinear polynomials is the O(n) interpolation, as
opposed to O(n$$\log$$n) for univariate polynomials. We interpolate over
the Boolean hypercube $$\{0,1\}^{t}$$, and for a function
$$f:\{0,1\}^{t}\rightarrow \mathbb{F}$$, its multilinear extension is
defined as:


$${f}(\textbf{x}) = \sum_{\textbf{w}\in\{0,1\}^{t}}f(\textbf{w})\cdot {Eq}(\textbf{x}, \textbf{w})$$


where for two length-t vectors **x**,
**y**:

$${Eq}(\textbf{x},\textbf{y}) = \prod_{i=1}^{t}(x_{i}y_{i}-(1-x_{i})(1-y_{i}))$$

We see that if x and y are binary, Eq(x,y) = 1 iff x
= y, 0 otherwise, hence the name.

There is an isomorphism between the additive group of all univariate
polynomials of max degree $$2^{t}-1$$ with coefficients in $$\mathbb{F}$$
and the group of all multilinear polynomials with t variables. We call
this the *power-to-subscript* map, and it sends a power $$x^{k}$$ to the
monomial $$x_{0}^{i_{0}}...x_{t-1}^{i_{t-1}}$$, where $$i_{0}...i_{t-1}$$ is
the binary decomposition of k. The inverse is

$$x_{0}^{i_{0}}...x_{t-1}^{i_{t-1}}\mapsto x^{i_{0}\cdot 2^{0}}...x^{i_{t-1}\cdot2^{t-1}}$$.

**Multilinear to univariate interpolation** 

This raises one (not directly related to aggregation) question: Is it possible to do O(n)
univariate interpolation by first doing multilinear interpolation over
the Boolean hypercube and then choosing a fitting map to turn into a
univariate polynomial? At this point, we may think that the answer is
obviously a positive one, since we have the power-to-subscript map and
its inverse, however that does not work; evaluations on the hypercube
are not mapped to the corresponding evaluations in $$\mathbb{F}$$ (Thanks
to Benedikt Bünz for pointing this out).

Sadly, there is reason to think this doesn't exist. First of all,
someone would have probably already thought of this, since O(n)
univariate interpolation would be a massive deal.

More concretely, the missing piece to our puzzle is the so-called
pullback 

$$\varphi^{*}$$ for $$\varphi: \mathbb{F}\rightarrow\{0,1\}^{t}, n = \sum_{j} i_{j}2^{j}\mapsto (i_{0},...,i_{t-1})$$

For $$f(x_{0},...,x_{t-1})\in\mathbb{F}[x_{0},...,x_{t-1}]$$,
$$\varphi^{*}f(x)$$ is defined as f($$\varphi(x))$$, so $$\varphi^{*}f$$ is
exactly the univariate polynomial we'd want to map f to.

The problem, then, is to find a closed form of the map $$\varphi$$. If we
can do this, we have solved the problem. More likely it does not exist.

**Better recursive proofs** 

In HyperPlonk, the authors use KZG commitments. For a Plonky-like approach, we would like something hash-based such as FRI. There does in fact exist a FRI analogue for
multivariate polynomials. The problem occurs where we need to check the consistency of the witness with the final prover message of the sumcheck protocol.

Specifically, the prover uses sumcheck in order to check that an
equation of the kind $$\sum_{\textbf{x}\in\{0,1\}^{t}}P(\textbf{x}) = 0$$,
where $$P(\textbf{x})=C(\omega(\textbf{x}))$$ for the constraint
polynomial C and the witness $$\omega$$ (we'll treat $$\omega$$ as a single
polynomial for simplicity). At the end of the protocol he sends a field
element $$m_{final}$$ which should correspond to P($$r_{1},...,r_{t})$$ for
random verifier challenges $$r_{i}$$.

The issue here is that a Merkle commitment corresponding to the values
of $$\omega$$ evaluated over, say, $$\{0,1\}^{t}$$, cannot simply be opened
at $$(r_{1},...,r_{t})$$. Actually, this is not quite true; one of the
nice facts about multilinear polynomials is that if we have the
evaluations of $$\omega$$ over $$\{0,1\}^{t}$$ we can use this to compute
$$\omega(r_{1},...,r_{t})$$ for any random such point. The drawback is
that this takes $$O(2^{t})$$ time, so it's linear in the witness size. It
also likely requires access to nearly all the leaves of the tree, which
kind of defeats the point.

This leaves us with a few open questions:

1.  Is there an efficient way for the prover to prove that he has
    computed $$\omega(r_{1},...,r_{t})$$ correctly, for example using the
    formula mentioned? (see for details)

2.  If not, is there some other recursion-friendly PCS which could be
    used?

As it turns out, we have likely found a solution to 1 (see [this
video](https://www.youtube.com/watch?v=tv5-gFgQWr0)).

This seems to be a promising direction, due to the linear proving time
and the increased flexibility with respect to custom gates. It also
ought to be mentioned that the use of the sumcheck protocol adds a
logarithmic term to the proof, which is more annoying for KZG-based than
FRI-based proofs, as the latter are already polylogarithmic (bigger than
the sumcheck part) whereas the former are constant-sized (much smaller)
by default.

__________________________________

_If you’re interested in further discussions on this topic or working together on this subject, please consider joining our [group chat](https://t.me/+9WAAmCpPRadjOTNh) or reach out to us at ole@delendum.xyz or research@delendum.xyz._