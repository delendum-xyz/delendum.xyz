---
layout: post
author: Christopher Goes
title: A Brief Taxonomy of Circuit Compilation Strategies
excerpt: "If most general-purpose applications of ZKPs will require general-purpose programmable privacy, what compiler stack can allow developers to safely write programs and also produce efficient outputs?"
---


*Thanks to Bobbin Threadbare, Alex Ozdemir, Daniel Lubarov, and Ventali Tan for helpful suggestions. This article is based on the talk given by Christopher Goes at Delendum ZKP Panel Series ([direct link](https://www.youtube.com/watch?v=SxI8uNBp05k&t=4s)).* 

### Table of Content

* [Motivation Question](#motivation-question)
* [General Constraints](#general-constraints)
	* [DSL Approach](#dsl-approach)
	* [ISA/VM Approach](##isa-vm-approach)
	* [Direct Approach](#direct-approach)
* [Takeaways](#takeaways)

## **Motivation Question**


The motivating question in defensive ZKP compiler pipeline design is:

*If most general-purpose applications of ZKPs will require general-purpose programmable privacy, what compiler stack can allow developers to **safely** write programs and also produce **efficient** outputs?*

This is the motivating question simply because the things that are many are the programs. Eventually, we’ll have a few proof systems and maybe a few different compiler stacks and those might have some bugs and we’ll need to iron out those bugs and formally verify those parts - but the proof systems and compiler stacks will be relatively small in number compared to the actual programs. 

Consider comparable examples, such as the Ethereum Virtual Machine (EVM) and the Solidity compiler. Early on in the history of the EVM and Solidity, the EVM and the Solidity compiler both had a lot of bugs, and often, bugs in programs were if not equally likely then at least non-trivially likely to be due to a bug in the compiler or in the EVM as opposed to a bug in the program caused by a developer not thinking about their model or writing the Solidity code correctly. But, over time, as the EVM and Solidity “solidified” and became better understood and more formally modeled in different ways, the frequency of those bugs has declined, in the case of the EVM perhaps to zero and in the case of Solidity to a very low number and to mostly minor ones. Now most of the problems, if you go on [Rekt](https://rekt.news/), most bugs nowadays are not bugs in the Solidity compiler or bugs in the EVM but rather bugs in code written by developers - sometimes mistakes caused by the model of the program itself being unclear or wrong (with respect to what the developer probably intended), sometimes mistakes in the use of the Solidity language to express that model, but in both cases the result of an incongruity at the developer level. If you’re interested in preventing bugs, you have to focus on this layer because most of the bugs are written by developers.

This holds doubly so for programs involving zero-knowledge proof systems (ZKPs) and privacy, because most developers aren’t going to be intricately familiar with the implementation details of zero-knowledge proof systems, and if the compilation stack requires that they understand these details in order to write programs safely, their programs are going to have a lot of bugs. It’s really hard to understand cryptography, especially if you don’t have a deep background in it and especially if you’re writing a complex application at the same time. Of course, if you have a very safe compiler stack but it’s not fast enough and the performance costs are too high for actual applications, no one will be any safer because no one will use it. So, the question can be rephrased a wee bit: what compiler stacks can provide the most protection to developers, while also being efficient enough that they will actually be used?


## **General Constraints**


There are some different constraints in circuit / ZKP compiler design than in the design of ordinary compilers. For one, there are some very specific “threshold latency constraints”, where execution times under a few seconds will be fine but execution times over that will render many applications impossible. In most cases, at least if you want privacy, users have to be creating the proofs, and when users are creating the proofs there’s a really big user experience difference between 3 seconds latency and 9 seconds latency, even though this is only a constant factor of 3. 

With normal compilers, you might be able to tolerate such small factor differences in the interest of making it easier to write programs (see: Javascript) - if you’re running a spreadsheet tax calculation program, for example, a difference of a factor of 3 may not actually matter that much - the person would just do something else while the program is running as long as they aren’t running it too frequently. With normal compilers you also tend to care about user interaction latency, which has deep implications in compiler design. For example, it means that languages with automatic memory management tend to do incremental garbage collection, which limits garbage collector latency, as opposed to simpler garbage collection algorithms which can cause unpredictable latency spikes. This doesn’t apply when writing circuits to be used in ZKP systems, because all of the interactions are non-interactive with respect to one individual program - the compiler should instead optimize for making the total prover execution time as low as possible.

Another class of parties who could potentially write bugs are the circuit compiler developers - us! - so it behooves the ecosystem to come up with at least some reusable complements of the compiler stack that can be shared by multiple parties and jointly verified. To that end, the subsequent parts of this post craft a high-level taxonomy of current approaches to circuit compilation and describe their trade-offs.

This taxonomy categorizes compilation approaches in three broad classes, which are defined and analyzed in order:

1. DSL approach
2. ISA/VM approach
3. Direct approach


### **DSL Approach**



The first approach and the approach that quite logically the ecosystem started with is herein termed the “DSL approach”. DSL stands for “domain-specific language”, and the DSL approach is to take something like [R1CS](https://learn.0xparc.org/materials/additional-learning-resources/r1cs%20explainer/) or [PLONK](https://eprint.iacr.org/2019/953.pdf) or [AIR](https://medium.com/starkware/arithmetization-i-15c046390862) and build a language on top of it, crafting abstractions iteratively in order to simplify the job of the programmer. DSL instructions in this sort of a system typically act like constraint-writing macros - some variables can be tracked, there’s some sort of simplified compilation going on, but there’s no intermediate instruction set or abstract machine, and instructions in the DSL translate pretty directly to constraints. Examples of this approach include:

* [Zokrates](https://zokrates.github.io/introduction.html)
* [Circom](https://docs.circom.io/)
* [Snarky](https://github.com/o1-labs/snarky)
* [Leo](https://leo-lang.org/)
* [Noir](https://noir-lang.github.io/book/index.html)
* [Juvix Circuits](https://github.com/anoma/juvix-circuits)
* probably many others (delendum has a more detailed table [here](https://kb.delendum.xyz/zk-knowledge#programming-languages))

While these languages are quite different, written with different syntaxes and targeting different proof systems, they are all following the same kind of general approach of starting with a low-level constraint system and iteratively adding abstractions on top. 

Advantages of the DSL approach:

* It’s relatively easy to write a DSL because you can iteratively build abstractions on top of constraints - you don’t need to have a whole separately-defined instruction set or architecture
* Because you have this low-level control, you can use a DSL as an easier way of writing hand-rolled circuit optimisations - it’s easy to reason about performance when the developer has a good understanding of exactly what constraints each instruction is translated into

Disadvantages of the DSL approach:

* DSLs aren’t as capable of high-level abstraction, at least not without more complex compiler passes and semantics more removed from the underlying constraint systems
* If there are a lot of different DSLs for different proof systems, developers are always going to need the semantics of some proof-system-specific language in order to write circuits, and the semantics of these DSLs are quite different than the semantics of the languages most developers already know
* If developers are writing programs that include some circuit components and some non-circuit components, those are two different semantics and languages, and the conversions in-between are a likely location for the introduction of bugs
* Most DSLs don’t built in much of the way of a type system for reasoning about the behavior about programs, so developers who wish to verify that their programs satisfy certain correctness criteria will need to use external tools in order to accomplish this

One challenge of the DSL approach is that each DSL needs a compiler. Such compilers are hard to build. The burden of building a new compiler from scratch just to test out a new idea for a DSL makes it infeasible to explore a large number of DSL designs.

One idea to decrease the cost of building such compilers is to design compiler infrastructure for compiling to circuits. The infrastructure can implement common optimizations, transformation algorithms, and abstractions. These are provided as a library that makes it easy to build new compilers. [CirC](https://eprint.iacr.org/2020/1586) is a compiler infrastructure for existentially quantified circuits, which capture the computation model of ZKP/MPC/FHE/SMT/ILP.


### **ISA/VM Approach**


The next compilation approach is termed ISA (instruction set architecture) / VM (virtual machine) compilation. This approach is more involved. ISA/VM compilation requires clearly defining an intermediate virtual machine and instruction set architecture which then intermediates in the compilation pipeline between the high-level languages in which developers are writing programs and the low-level zero-knowledge proof systems with which proofs are being made. Although it is possible to write VM instructions directly, in this approach it is generally assumed that developers won’t (and these VMs are not optimized for developer ergonomics).

This can be done with an existing instruction set. For example, the Risc0 team has done this with the RISC-V microarchitecture - originally designed for CPUs with no thought given to ZKP systems - but Risc0 has written a [STARK prover and verifier for RISC-V](https://github.com/risc0/risc0) which is compatible with the existing specification of the RISC-V ISA.

This can also be done with a custom-built VM and instruction set designed specifically to be efficient for a particular proof system, and potentially for recursive verification in the VM or some other specific application you might want to support efficiently. For example, the [Cairo VM](https://eprint.iacr.org/2021/1063) and the [Miden VM](https://github.com/maticnetwork/miden) are designed to be more efficient to prove and verify in a STARK than an architecture such as RISC-V (which was not designed for this) might be, and the [Triton VM](https://github.com/TritonVM/triton-vm) has been designed specifically to make recursive proof verification in the VM relatively inexpensive.

All of these approaches use one circuit for all programs, and support many higher-level languages - the higher-level languages just need to compile to the specified instruction set - and most of these examples are [von Neumann architecture](https://en.wikipedia.org/wiki/Von_Neumann_architecture)-style machines - so there’s a stack, operations on the stack, some form of memory, storage, I/O, and maybe some co-processors or built-in operations. Note that RISC-V, Cairo are not stack machines (RISK-V is a register machine and Cairo is a memory machine), and Miden VM is not von Neumann architecture.

Advantages of the ISA/VM approach:

* Developers can use higher-level languages which they are either already familiar with or are easier to learn than low-level circuit DSLs
* In the case of using an existing microarchitecture, such as RISC-V, you can reuse tons of existing tooling - anything which compiles to RISC-V, which is anything which compiles to [LLVM](https://llvm.org/), which is almost everything you would care about - can be used with the Risc0 circuit, prover, and verifier
* Even in the case of a new instruction set architecture, such as with Miden VM and Triton VM, compilation techniques to these kinds of stack-based VMs for both imperative and functional languages are pretty well-known already, and compiler authors can reuse a lot of existing work that is not specific to ZKPs or circuits

Disadvantages of the ISA/VM approach:

* Although it’s unclear exactly what this cost is, there is probably going to be some cost of abstraction when compiling high-level programs to some intermediate instruction set - e.g. you’re doing stack operations, which can probably be checked more directly in a purpose-built circuit for any given program. There’s reason to expect that this will be especially true and that there will be a higher performance penalty if the instruction set architecture wasn’t built specifically for ZKPs (such as RISC-V).
* Arguably, this approach offers more power than applications actually need, and is going to be correspondingly harder to verify correctness of because it’s more powerful. There’s a lot of noise about Turing-completeness, but no one really wants to be able to run ZKP computations that don’t terminate because it would be impossible to create a proof in practice. This is different than in the interactive world - most interactive programs, say a spreadsheet, never terminate because they take in user input in a loop and take action according to the users’ input, then wait for more input until the user (or OS) tells the program to exit. Circuit execution, on the other hand, always terminates.
* If you want to verify the correctness of the compiler in the ISA/VM approach, you have to verify both the correctness of the translation from the higher-level language semantics into the intermediate instructions, and the correctness of the circuit which implements the intermediate ISA VM - although if you’re using existing or more powerful higher-level languages, there’s probably a lot of existing work that can be reused here.

### **Direct Approach**

The third approach, for lack of a better word, is just referred to here as direct. The basic idea here is to somehow directly compile higher-level programs to circuits. These higher-level programs could be functional, based on the lambda calculus, or some kind of imperative language, but the direct approach involves transforming this high-level language directly to a circuit without an intermediate instruction set. 

The best candidate for this sort of translation, at least for functional languages, is based on a paper called [“Compiling to Categories”](http://conal.net/papers/compiling-to-categories/) by Conal Elliot, which works by taking the kind of simple lambda calculus core that you can translate a functional language to, translating the semantics of lambda calculus operations to the semantics of a category, translating the categorical semantics into the category of polynomial operations, then translating the polynomial operations into a specific circuit. You can find a more detailed description in the GEB project repository [here](https://github.com/anoma/geb/tree/main/geb). 

Similar to the DSL approach, with the direct approach each program generates a unique circuit. Other kinds of direct compilation may be possible - arguably, there’s a sort of continuum of DSLs where they approach higher and higher-level semantics and get more and more like this kind of direct compilation.

Advantages of direct compilation:

* Developers can use a very high-level language and at least in principle you can get very high performance because you don’t lose any information when you compile this way (e.g. you don’t have to represent things in terms of intermediate stack operations)
* This approach is similar to some existing work in hardware circuit synthesis done outside of the context of ZKP systems but in the context of the end of Moore’s law and compiler pipelines that want to produce physical circuits and extremely parallel programs
* At least with “Compiling to Categories”, compilation has a very clear mathematical model that is easy to reason about and verify

Disadvantages of direct compilation:

* More complexity in the compiler pipeline, at least compared to the simple DSL approach
* Most existing tooling cannot be reused, since the core and compilation techniques are new and non-standard
* As in general with any approach that involves a higher degree of abstraction, it may be more difficult for developers to reason about the performance of their programs


## **Takeaways**

So many compiler options! It’s all a little exhausting - and prompts the question: what might be expected to happen, and where, accordingly, can efforts best be spent right now? 

The three approaches outlined here have different trade-offs, particularly in terms of potential performance, low-level control, degree of abstraction, and difficulty of creating appropriate tooling. Due to these trade-offs, we’re likely to see a hybridization of approaches, where critical functions and gadgets (e.g. hash functions, Merkle proof verification, signature checks, recursive verification, other cryptography) are written directly in lower-level DSLs (1) or even in raw constraints, but these will be provided in standard libraries and then tied together with “business logic” written by developers in a higher-level language compiled using either the ISA/VM (2) or direct compilation (3) approaches. 

Hybridization is likely because it gives you the best of both worlds - you can spend your “optimisation time” (the time of people writing hand-optimized circuits) on the critical “hot spots” of your code which really need to be optimized and whose performance mostly determines the overall performance of your program, and use a less efficient but more general compilation pipeline for custom per-program “business logic” written by regular developers.

This is also similar to the structure of cryptographic systems today - if you look at how people implement cryptographic protocols, there’s usually hardware support (after standardization) for specific optimized primitives (hash functions, encryption algorithms), which are tied together in protocols themselves written in higher-level languages which are easier to develop in and reason about.

Finally, a minor exhortation: agreement on and convergence to specific standard primitives and intermediate interfaces can help facilitate deduplication of work across the ecosystem and an easier learning environment for developers. A good example of a moderately successful standardization effort in the space already is the [IBC protocol](https://ibcprotocol.org/), which is now being adopted by many different chains and ecosystems. 

For zero-knowledge related standards effort, [zkInterface](https://github.com/QED-it/zkinterface) is a standard tool for zero-knowledge interoperability between different ZK DSLs, gadget libraries, and proving systems. Anoma is trying to help a little bit in this direction with [VampIR](https://github.com/anoma/vamp-ir) proof-system-agnostic intermediate polynomial representation, Delendum is also exploring practical approaches for composition and conversion across different proving systems — and we’d be very happy to collaborate or learn about other standards efforts. 


__________________________________

_If you’re interested in further discussions on this topic or working together on this subject, please consider joining our [group chat](https://t.me/+9WAAmCpPRadjOTNh) or reach out to us at research@delendum.xyz._

