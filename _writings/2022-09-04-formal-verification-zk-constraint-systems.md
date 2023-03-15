---
layout: post
author: Morgan Thomas and Ventali Tan
title: Formal Verification of ZK Constraint Systems
excerpt: "How do we formally verify that an arithmetic circuit, as used by a zero knowledge proof system, has the desired characteristics, such as soundness, completeness and zero knowledge? Soundness of a proving system means that it does not prove statements that are false. Similarly, soundness of the circuit used by a proof system means that the circuits are unsatisfiable when applied to a statement that is not true."
---


*Thanks to Eric McCarthy from Kestrel Institute for helping with some sections of this document, and to Grigore Rosu (UIUC), Shashank Agrawal (Delendum), Daniel Lubarov (Delendum), Tim Carstens (Risc Zero), Bolton Bailey (UIUC) and Alessandro Coglio (Kestrel) for many helpful suggestions.*

### Table of Content

* [Leading Problem](#leading-problem)
* [Techniques](#techniques)
* [Formal Verification for ZK Circuits](#formal-verification-for-zk-circuits)
* [Synthesizing Formally Verified Programs](#synthesizing-formally-verified-programs)
* [The State of Current Progress](#the-state-of-current-progress)
* [Future Research and Development Directions](#future-research-and-development-directions)

## **Leading Problem**


How do we **_formally verify_** that an arithmetic circuit, as used by a **_zero knowledge proof system,_** has the desired characteristics, such as soundness, completeness and zero knowledge[^1]?

Soundness of a proving system means that it does not prove statements that are false. Similarly, soundness of the circuit used by a proof system means that the circuits are unsatisfiable when applied to a statement that is not true.

Completeness of a proving system means that it is able to prove all statements that are true. Similarly, completeness of the circuit used by a proof system means that the circuit is satisfiable when applied to a statement that is true.

Soundness and completeness of a circuit, taken together, are equivalent to the statement that the relation denoted by the circuit is the intended relation.

A proof system is zero knowledge if and only if a proof does not reveal any additional information beyond the statement being proven (i.e., the public inputs to the proving process). Any facts straightforwardly implied by the statement being proven are necessarily also implied by the proof, but additional facts not implied by the statement should not be revealed. Additional “non-public” witness data, which may be considered as secret inputs, must not be revealed. 

A sound and complete circuit can only be satisfied by the statements which are true in the intended semantics. If the underlying proving system is sound and complete, and if the circuit is correct, then the overall system provides a sound and complete means of proving the intended statements. We can separate the leading problem into two subproblems: verifying the desired characteristics of the underlying proving system, and verifying the soundness and completeness of the circuit. In other words, the circuit denotes the intended relation and the proving system will behave as intended when applied to the circuit.

Our main focus here is on the second subproblem, that of verifying the circuits of a system while assuming the soundness and completeness of the underlying proving system[^2].

Zero knowledge proof systems often use a mathematical constraint system such as R1CS or AIR to encode a computation. The zero knowledge proof is a probabilistic cryptographic proof that the computation was done correctly.  Formal verification of a circuit used in a zero knowledge proof requires (1) a formal specification of the relation the circuit is intended to encode, (2) a formal model of the semantics of the circuit, (3) the circuit itself, (4) the theorem prover, and (5) the mechanized proof of a theorem relating (1), (2) and (3). 


## **Techniques**



* To prove a general statement with certainty, it must be a purely mathematical statement which can be known to be true by reasoning alone, without needing to perform any tests. The reasoning must start from consistent assumptions (axioms) and proceed by truth-preserving inferences.
    * Example: given that a bachelor is defined as an unmarried man, and a married man by definition has a spouse, it can be proven with certainty that no bachelor has a spouse.
* A proof theory (sometimes also called a logic or metalogic) codifies a consistent set of rules for performing truth-preserving inference. Proof theories can be used to construct proofs manually, with pencil and paper.
    * Example: [sequent calculus for classical logic](http://logitext.mit.edu/logitext.fcgi/tutorial)
* Computer programs called proof assistants reduce labor and mitigate the risk of human error by helping the user to construct machine-checkable proofs.
    * Examples: [Coq](https://coq.inria.fr/), [Agda](https://agda.readthedocs.io/en/v2.6.0.1/getting-started/what-is-agda.html), [Isabelle](https://isabelle.in.tum.de/), [ACL2](https://www.cs.utexas.edu/users/moore/acl2/manuals/latest/?topic=ACL2____TOP), [PVS](https://pvs.csl.sri.com/), [Imandra](https://docs.imandra.ai/imandra-docs/), and [Lean](https://leanprover.github.io/)


### **Using a proof assistant**



* To use a proof assistant to prove a statement about a program, there are two main approaches:
    1. Write the program in the proof assistant language and apply the proving facilities to the program directly.
    2. Use a transpiler[^3] to turn a program written in another language into an object which the proof assistant can reason about. This is called post hoc verification.


* Of these approaches, (1) seems preferable for the greater confidence provided by the proof being about exactly the (source) program being executed, as opposed to output of a transpiler which is assumed to have the same meaning as the source program.
* What motivates approach (2) is when (for whatever reason) the proof assistant language is not suitable as a language for developing the application in[^4].

### **Without using a proof assistant**



* There are also ways to prove a statement about a program without (directly) using a proof assistant:
    1. Use a verified compiler (e.g. [CompCert](https://compcert.org/)), which turns a source program into an object program which provably has certain properties by virtue of (proven) facts about the verified compiler.
        * Note that there is a distinction between a **verified** compiler and a verifying compiler.  CompCert itself is proved to generate binary that will always be semantically equivalent to the C input.  A **verifying** compiler generates a binary along with a proof of correctness that the binary is semantically equivalent to the source.
        * When a program is compiled by a verifying or verified compiler, we say that the compiler output is correct by construction (provided that the input is correct).
        * Verified compilers do not detect or fix bugs in the input source code; however, they also do not introduce any new bugs during compilation.
    2. Use an automatic proof search algorithm, which takes as input statements to be proven and outputs proofs of those statements if those statements are true and the proof search algorithm finds proofs.
    3. Use a static analyzer, which takes as input a program and automatically checks for various kinds of issues using predetermined algorithms.


* All of these approaches have limitations:
    1. A verified compiler is limited in what statements it can prove about the resulting program: typically, just that the resulting program has the same meaning or behavior as the source program.
    2. An automatic proof search algorithm is limited in what statements it can prove by the sophistication of the algorithm and the computational power applied to it. Also, due to the undecidability of first-order logic[^5], there cannot exist a proof search algorithm which would find a proof of any given true statement.
    3. A static analyzer is generally not capable of reasoning about the meaning of a program to see if it’s correct; it is only able to recognize patterns which always or often indicate some kind of issue.


## **Formal Verification for ZK Circuits**



* Formal verification for probabilistic proof systems, inclusive of ZK proof systems, encompasses two main problem spaces:
    1. Proving the intended properties of a general-purpose proving system, such as soundness, completeness, and zero knowledge.
        * E.g., Bulletproofs, Halo 2, etc.
    2. Proving the intended properties of a circuit, namely that it correctly encodes the intended relation.
* Problem (1) is generally considered a very difficult problem and has not been done for any significant proving system.
* Problem (2) can be done with a formal specification of the relation and a model of the circuit semantics.  Usually it requires proving functional correctness of functions defined within the relation as well as the existence of witness variables for every argument to the function.

Note: There are different ways to axiomatize a problem to prove it. Some categories are denotational semantics, axiomatic semantics, and operational semantics.   \
 \
Operational semantics is particularly useful for proving things about programs. For example, in the ACL2 theorem prover, a common approach to formalizing programming language semantics is an _interpretive operational semantics_, i.e. a high-level interpreter, written in the logic of the theorem prover, that formalizes the possible forms of the states of computation and describes how the execution of the constructs of the programming language (e.g. expressions and statements) manipulate the computation states.


### **Denotational design**



Denotational design provides a helpful way of thinking about both problem spaces (general and application-specific).

Denotational design also provides a methodology for defining the requirements of a system, such as a zero-knowledge proving system, in such a way that the requirements can be expressed and proven in a formal system.

* A circuit denotes a set: namely, the set of public inputs (i.e., statements) for which the circuit is satisfiable (i.e., the statement is true)
    1. For example, consider a hash verification circuit
        * A public input is a pair (x, h) where x is some data and h is a hash
        * A public input (x, h) expresses the statement that h is the hash of x
        * The circuit is (or should be) satisfiable exactly when h is the hash of x
* The goal of circuit verification is to prove that the circuit denotes the intended relation.
* The goal of general purpose proving system verification is to prove that it has the intended properties with respect to the denotational semantics of circuits:
    2. Soundness means that if the verifier accepts a proof, then with high probability, the public input used to generate the proof (i.e., the statement being proven) is in the set denoted by the circuit (i.e., the statement is true)
    3. Completeness means that if a public input (i.e., a statement) is in the set denoted by the circuit (i.e., the statement is true), then the proving algorithm successfully outputs a proof which the verifier accepts
    4. Example: consider a ZK proving system which one would use to show that one has a solution to a Sudoku puzzle, without revealing the solution
    5. The statements to be proven are of the form "X (a Sudoku puzzle) is solvable"
    6. The relation we intend is the set of solvable Sudoku puzzles
    7. Soundness means that if the verifier accepts a proof that X is solvable, then with high probability, X is solvable
    8. Completeness means that if X is solvable, then the prover creates a proof that X is solvable which the verifier accepts
    9. Zero knowledge means that having a proof that X is solvable does not reduce the computational difficulty of finding a solution to X
    10. To see this example worked out more formally, see [the OSL whitepaper](https://eprint.iacr.org/2022/1003)

<img src="https://github.com/iyusufali/delendum-xyz-posts-assets/blob/main/2022-09-04-formal-verification-zk-constraint-systems/figure1.png?raw=true" style="display: block;margin-left: auto;margin-right: auto;width:60%">
<p style="text-align:center; font-style: italic;"> Figure 1: denotational design</p>


*A formal version of this diagram is in Sigma^1_1 arithmetization [paper](https://eprint.iacr.org/2022/777.pdf) ((288) on page 47). This is a commutative diagram. Soundness and completeness of a circuit compiler are both expressed by the statement that this diagram commutes, meaning that all paths through it between the same objects are equal, or in other words, the denotation of the compiled circuit is always equal to the denotation of the spec.



* If you know that your circuit denotes the relation you intend, and you know that your general purpose proof system is sound and complete in the above senses, then you know that your application-specific proving system (i.e., the circuit plus the general proof system) has the intended soundness and completeness properties for that application.
* This suggests that, given a formally verified general-purpose proving system, and a verified compiler from statements to circuits, one can solve the problem of proving correctness of application-specific proving systems without application-specific correctness proofs.
* Suppose that 
    11. one can write the statement to be expressed by a circuit in a machine-readable and human-readable notation, where it is self-evident that the statement being written has the intended meaning or denotation
    12. one has a verified compiler which turns that statement into a circuit which provably has the same denotation as the source statement
    13. circuit can be executed on a formally verified general-purpose probabilistic proving system
* Then one can generate formally verified application-specific probabilistic proving systems without any additional proof writing for an additional application. This seems like a promising way forward towards a sustainable and cost effective approach to formal verification for ZK circuits.


## **Synthesizing formally verified programs**

It is often a challenge to bridge the gap between the theorem we can prove and the code we can execute. For example, we may be able to prove a theorem about a spec, and code that spec, but then not able to prove that the code implements the spec. Or, we may be able to prove a theorem about some code but not able to compile that code to efficient machine code. Or, we may be able to do that, but unable to prove that the machine code has the semantics as the source code.

Here is a summary of some of the ways in which the ecosystem supports correct-by-construction synthesis of efficient code:



* The proof assistants [Coq](https://coq.inria.fr/) and [Agda](https://github.com/agda/agda/) provide for extraction to other languages to be compiled to machine code; this may leave something to be desired in terms of the efficiency of the resulting machine code.
* The language [ATS](http://www.ats-lang.org/) provides proof facilities and purports to allow for programming with the efficiency of C and C++.
* There are various means for transpiling code written in a mainstream language such as C or Haskell into a proof assistant, which allows for theorems to be proven about the extracted model of the source program.
* You can synthesize an efficient binary program using Coq (e.g., using [Fiat](https://github.com/mit-plv/fiat-crypto) or [CertiCoq](https://certicoq.org/)).
* The proof assistant ACL2 defines a subset of Common Lisp with a full formal logic[^6].  When a definition is executable, it can be compiled into efficient code, and because the language is a formal logic, you can define and prove theorems about the code.
* There is a verifying compiler project, [ATC](https://kestrel.edu/research/atc), from ACL2 to C. 
* Imandra defines a subset of OCaml with a full formal logic and a theorem prover.


### **Current limitations of formal methods on modern hardware**



* In the context of modern computing, most computationally intensive tasks deal with vector math and other embarrassingly parallel problems which are done most efficiently on specialized hardware such as GPUs, FPGAs, and ASICs.
* This is generally true of the problem of constructing proofs in probabilistic proof systems. Provers for these proof systems would be most efficient if implemented on specialized hardware, but in practice, they are usually implemented on CPUs, due to the greater ease of programming on CPUs and the greater availability of those skill sets in the labor market.
* For creating a formally verified implementation of a probabilistic proof system which executes efficiently, it seems that the right goal is not to optimize for speed of execution on a CPU, but to target specialized hardware such as FPGAs, GPUs, or ASICs.
* Unfortunately, tools for compiling formally verified programs to run on FPGAs, GPUs, or ASICs are more or less nonexistent as far as we know.


## **The State of Current Progress**



* Decades of research exists on formal verification strategies for arithmetic circuits in the context of hardware verification.
    * See, e.g., [Drechsler et al (2022)](https://link.springer.com/chapter/10.1007/978-981-16-7182-1_36) and [Yu et al (2016)](https://ieeexplore.ieee.org/document/7442835)
    * This work has limited industrial applications, e.g., the AAMP5 (see [Kern and Greenstreet (1997), page 43](https://cse.usf.edu/%7Ehaozheng/lib/verification/general/survey-FV.pdf)).
    * This line of research is not directly applicable to formal verification of arithmetic circuits for zk-SNARKs, because arithmetic circuits in hardware and arithmetic circuits in zk-SNARKs are not quite the same things.
* Decades of research exists on proving soundness, completeness, and zero knowledge for probabilistic proof systems.
    * This work has generally been carried out informally, but provides a starting point for work on proving the same factors formally.
    * Examples:
        * [Groth16](https://eprint.iacr.org/2016/260.pdf) 
        * [Bulletproofs](https://eprint.iacr.org/2017/1066.pdf) 
        * [PLONK](https://eprint.iacr.org/2019/953.pdf) 
* Orbis Labs is working on:
    * A [verifying Halo 2 circuit compiler](https://github.com/Orbis-Tertius/coq-arithmetization) for [Σ¹₁ formulas](https://eprint.iacr.org/2022/777).
    * [Orbis Specification Language (OSL)](https://eprint.iacr.org/2022/1003/), which provides a high level spec language which we can compile to Σ¹₁ formulas.
    * A toolchain (Miya) for developing formally verified, hardware accelerated probabilistic proof systems.
        * A theory of interaction combinator arithmetization, towards compiling formally verified code into circuits.
        * Definitions of basic concepts such as probabilistic proof system, soundness of probabilistic proof system, and the zero knowledge property.
            * Goal is to find definitions which are maximally parsimonious and amenable to formal proving.
* [Kestrel Institute](https://www.kestrel.edu) is a research lab that has proved functional correctness of a number of R1CS gadgets using the ACL2 theorem prover.  (Kestrel also does a lot of other FV work, including on Yul, Solidity, Ethereum, and Bitcoin.  Kestrel Institute has tools for both correct-by-construction synthesis of code from a formal specification using the APT toolkit  as well as post-hoc verification using the Axe toolkit to lift binaries, bytecode, and R1CS into logic for verification against a formal specification.)
    * They [formalized and proved](https://www.cs.utexas.edu/users/moore/acl2/manuals/latest/index.html?topic=ZKSEMAPHORE____SEMAPHORE) the functional correctness of parts of the Ethereum Semaphore R1CS.
    * They [formalized and proved](https://www.cs.utexas.edu/users/moore/acl2/manuals/latest/index.html?topic=ZCASH____ZCASH) the functional correctness of parts of the Zcash Sapling protocol.  An overview of the process:
        * Used the [ACL2](https://www.cs.utexas.edu/users/moore/acl2/) proof assistant to formalize specs of parts of the Zcash Sapling protocol.
        * Formalized rank 1 constraint systems [(R1CS) in ACL2](https://www.cs.utexas.edu/users/moore/acl2/manuals/latest/index.html?topic=R1CS____R1CS).
        * Used an extraction tool to fetch the R1CS gadgets from the Bellman implementation, and then used the [Axe](https://www.kestrel.edu/research/axe/) toolkit to lift the R1CS into logic.
        * Proved in ACL2 that those R1CS gadgets are semantically equivalent to their specs, implying soundness and completeness.
* [Aleo](https://www.aleo.org/) is developing programming languages such as [Leo](https://leo-lang.org/)<span style="text-decoration:underline;"> </span>that compile to constraint systems such as R1CS.
    * Aleo aims to create a verifying compiler for Leo, with theorems of correct compilation generated and checked using the ACL2 theorem prover.
    * Aleo has also done post-hoc verification of R1CS gadgets using Kestrel Institute's [Axe](https://www.kestrel.edu/research/axe/) toolkit.
* [Nomadic Labs](https://www.nomadic-labs.com/) is a consulting firm that does a lot of work on Tezos and they built the Zcash Sapling protocol for shielded transactions into the Tezos blockchain as of the Edo upgrade.   Kestrel Institute formally verified some of the R1CSes used in that protocol. (Nomadic Labs also does a lot of other FV work.)<span style="text-decoration:underline;"> </span>
* Andrew Miller and Bolton Bailey are working on a [formal verification of a variety of SNARK](https://github.com/BoltonBailey/formal-snarks-project) proof systems, using the Lean Theorem Prover, in the Algebraic Group Model.
* Anoma team is working on the [Juvix language](https://github.com/anoma/juvix) as a first step toward creating more robust and reliable alternatives for formally verified smart contracts than existing languages.
* Veridise is working on:
    * [Medjai](https://github.com/Veridise/Medjai), a symbolic evaluator for Cairo, intended for use in automatic proof search.
    * [Picus](https://github.com/Veridise/Picus), a symbolic VM for R1CS, intended for use in automatic proof search.
    * [V](https://github.com/Veridise/V), a specification language intended for use in expressing statements to be proven by automatic proof search.
    * [Coda](https://github.com/Veridise/Coda), a framework for certifying ZK circuits using the Coq proof assistant. It uses a high-level, functional ZK DSL to summarize circuit behavior and simplify manual proving.
* [Ecne](https://0xparc.org/blog/ecne) is a special-purpose automatic proof search tool which can prove that an R1CS constraint system defines a function (total or partial).
    * In other words, it proves that for any input values on which the system is satisfiable, there is a unique combination of output values on which the system is satisfied.
    * This approach has been proven to be useful in flushing out bugs in circuits.
* [Starkware](https://starkware.co/) is writing [Lean](https://leanprover.github.io/) [proofs](https://github.com/starkware-libs/formal-proofs) to check that circuits expressed as Cairo programs conform to their specs. 
* [Alex Ozdemir](https://cs.stanford.edu/~aozdemir/research/) from Stanford is working on adding a finite field solver to the [cvc5](https://github.com/alex-ozdemir/CVC4/tree/ff) SMT Solver.
* Lucas Clemente Vella and Leonardo Alt are working on [SMT solver](https://github.com/lvella/polynomial-solver/blob/master/docs/SMT-2022-extended-abstract/full-text.pdf) of polynomial equations over finite fields.


## **How to analyze each research effort / development project**

If you like the structure we frame in “Leading Problem”, you could analyze each research effort / development project in terms of



1. the language used for specification; 
2. the language used to model the circuit; 
3. which theorem prover they are using; and
4. what sorts of theorems are they proving, like soundness, completeness, functional completeness, and zero knowledge.

Other interesting attributes of project concerning formal verification for circuits are:



* whether the tooling and the formal verification are open source.  It is hard to have confidence in a theorem when the components that went into proving that theorem are not available for inspection
* what is the _trusted core_, i.e., what software in the stack is not formally verified, and what are the possible consequences if it has bugs.   


## **Future Research and Development Directions**

A lot of work needs to be done. There is not enough emphasis placed on formal verification in the security industry.

Based on the observations and arguments presented in this blog post, we think the following will be some interesting directions for future research and development:



* Build foundations for formally verifying zero knowledge proof systems:
    * Generalizable proof techniques for proving the desired properties formally
    * Reusable verified abstractions for proof systems, e.g., a polynomial commitment scheme library
* Improve specification languages and verified translators between specification languages
* Understand how to create formally verified programs to run on vectorized hardware, e.g., FPGAs, GPUs, and/or ASICs
* Can we formally verify systems that are designed to automatically make ZK circuits more efficient? 
    * For example: systems that choose a different circuit so that the setup MPC is more parallelizable or that allow a prover who learns part of the witness in advance to partially evaluate a circuit and use this information to compute proofs faster
* Use K to prove statements about ZK constraint system
    * Define the semantics of circom/cairo in K
    * Use Rust semantics defined in K to prove properties of arkworks-rs programs 

__________________________________

_If you’re interested in further discussions on this topic or working together on this subject, please consider joining our [group chat](https://t.me/+9WAAmCpPRadjOTNh) or reach out to me at ventali@delendum.xyz._

**Terminology:**



1. If you write a specification of a computation in a high-level formal language and compile it to a constraint system using a verified or verifying compiler, that is called **_correct by construction_**.  If you take an existing constraint system and you try to prove properties about it (up to and including soundness and completeness) with respect to a specification in a high-level formal language, that is called **_post-hoc verification_**. 

**Discussion: “Due to Gödel's incompleteness theorem”**



1. Originally, we put “due to Gödel's incompleteness theorem”. Alessandro suggested we change to “the undecidability of first-order logic”, because the non-existence of such a proof search algorithm is due to "less than" Gödel's incompleteness theorem: it is due to the undecidability of first-order logic (which is complete according to Gödel's [completeness theorem](https://en.wikipedia.org/wiki/G%C3%B6del%27s_completeness_theorem)). Then second-order and higher-order logic is incomplete, in addition to being undecidable. 

2. Morgan: 

    The undecidability of first order logic does not imply that there cannot be a proof search algorithm which proves any given true statement of first order arithmetic. First order logic is computably enumerable, so there can exist a proof search algorithm which would find a proof of any given provable statement. It's due to the incompleteness of any computably enumerable set of axioms for first order arithmetic that there cannot exist a complete proof search algorithm.

    Let me elaborate on that, please. If you have a complete theory then there can exist a proof search algorithm which would eventually prove any true statement. In such a case, the theory is decidable, since the proof search algorithm will also find a disproof of any false statement. The catch is that first order arithmetic can't have a complete and true proof theory with a set of axioms that can be written out by an algorithm. That's Gödel's incompleteness theorem. 

    The fact that first order logic is undecidable is the fact that the logical consequence relation of first order logic is undecidable; there is no algorithm that decides if a given set of axioms entails a given theorem or not. That's not the relevant point, though. If first order arithmetic was complete, then its logical consequence relation would be decidable.

3. Alessandro: 

    Yes, technically first-order logic is [semidecidable](https://en.wikipedia.org/wiki/Decidability_(logic)#Semidecidability), meaning that there can be an enumeration procedure that will eventually find if a formula is a theorem (as Morgan says), but if it is not, it will run forever (so it's undecidable). I agree that Morgan's statement is technically correct, but normally in theorem proving we think of undecidability being the main obstacle, rather than incompleteness. (Well, in fact also decidable theories like propositional logic may take impractically long times.) In practical terms, proving things in first-order logic vs. higher-order logic is equally hard, due to undecidability, regardless of first-order being complete and higher-order being incomplete. I think it's fine if you change back the text to mention Gödel's incompleteness, but I think it might be slightly better to change the whole sentence to say something like "Also, provability is undecidable in first-order and higher-order logic." Then it gets at the heart of the problem in theorem proving, which is not only for incomplete logics, but also for complete ones. But I think that either way is fine. 



    An alternative phrasing could be "Also, due to undecidability of first-order and higher-order logic, there cannot exist an algorithm that can establish whether a formula is a theorem or not."


<!-- Footnotes themselves at the bottom. -->
**Footnotes**

[^1]:
     We do not elaborate on the _zero knowledge property_ in the article and will write another article on that. To describe that, you need to get into the concept of private inputs.  (The example of correct computation of a hash from a hidden plaintext is a good one for understanding this.)  The zero knowledge property is how hard it is to determine the private inputs from the public information.  You must encode some hard-to-invert function of a private input in the circuit if you want it to have the zero-knowledge property.  The zero-knowledge property is generally a probabilistic measure.  However, if the computation is too simple and the output is known, it might become possible to figure out the input with certainty.

[^2]:
     It is worth noting that this problem can be solved in a manner that is independent of the ambient ZK system: it is really just a problem about systems of polynomial equations.

[^3]:

     The transpiler idea makes the most sense when there is a formally defined DSL that is close to the new language you want to define, and then you can transpile from your new language to that DSL. However, that transpiler should really be formally verified as well.

[^4]:

     In our experience of formal verification for systems that have some cryptographic component, we have noticed a few different things: 
    1. It's extremely difficult to formally reason about the security properties of the "whole" system. The system design is usually very complex with a lot of dependencies of different types. Even the cryptographic component, which could be pretty small and simple at a high-level (say encryption, MAC, etc), may be implemented in a very complex way, with support for failures, recovery, backup, etc. As a result, even modeling the entirety of just the crypto component in a formal way is quite challenging. So any formal conclusions we draw ends up being for a very small part of the whole system. Our methods have not yet developed to a point where we know how to reason about the whole system in formal proofs.
    2. Proof assistant languages -- at least for crypto protocols -- so far have limited expressivity. There are limited tools that can directly prove theorems about a crypto protocol written in C or even Rust. In general, developers of protocols need to handle multiple threads and timing issues and those are hard to formalize.  E.g., [Gallina](https://coq.github.io/doc/v8.9/refman/language/gallina-specification-language.html) (for Coq) doesn't have such constructs. From that point of view, Gallina is not very developer-friendly and Rust using threads is. Future efforts to reason formally about concurrent software systems may benefit from formalization of existing abstractions for reasoning about such systems, such as process calculi, like for example [the π-calculus](https://www.sciencedirect.com/science/article/pii/0890540192900084).

[^5]:

     Originally, we put “due to Gödel's incompleteness theorem”. Alessandro suggested we change to “the undecidability of first-order logic”, because the non-existence of such a proof search algorithm is due to "less than" Gödel's incompleteness theorem: it is due to the undecidability of first-order logic (which is complete according to Gödel's [completeness theorem](https://en.wikipedia.org/wiki/G%C3%B6del%27s_completeness_theorem)). Then second-order and higher-order logic is incomplete, in addition to being undecidable. 

[^6]:

     Common Lisp can be compiled into quite efficient code, very close to C or Rust. (However, not as efficient as GPUs.  Also, concurrent programming, while available in Common Lisp, is not part of the subset in ACL2 for which a formal logic is defined.) If you write a formal specification that is executable though, then it can usually be compiled into fairly efficient code, all within the language of the theorem prover.
