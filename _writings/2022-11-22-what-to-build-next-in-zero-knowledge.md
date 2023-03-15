---
layout: post
author: Delendum Research
title: "Part I: What to build next in Zero Knowledge?"
excerpt: "What are the problems that haven’t been solved in blockchain and how can we leverage zero-knowledge proof as a tool to solve these problems?"
image: "/assets/posts/2022-11-14-what-to-build-next-in-zero-knowledge/preview.jpg"
redirect_from:
  - /2022/11/14/what-to-build-next-in-zero-knowledge
  - /what-to-build-next-in-zero-knowledge
--- 

By Daniel Lubarov, Aaron Li, Andrei Nagornyi, James Stearn, Ole Spjeldnæs, Guiltygyoza, Ventali Tan

*Thanks to Dmitry Khovratovich, Alan Szepieniec, Bobbin Threadbare, Tim Cartens and Thor Kamphefner for helpful suggestions.*

### Table of Content

- [Introduction](#introduction)
- [Blockchain setting](#blockchain-setting)
    * [Scalable zk-rollup](#scalable-zk-rollup)
    * [Faster hash function](#faster-hash-function)
    * [Cross chain: trust, data and privacy](#cross-chain-trust-data-and-privacy)
    * [Universal layer for proof aggregation](#universal-layer-for-proof-aggregation-and-composition)
    * [Verifiable gaming and open world](#verifiable-computation-gaming-and-open-world)
    * [Formal verification of the ZK system](#formal-verification-of-the-zero-knowledge-tech-stack)
- [Introducing our fellowship program](#introducing-our-fellowship-program)

## Introduction

What are the problems that haven’t been solved in blockchain and how can we leverage zero-knowledge proof as a tool to solve these problems? We welcome everyone to contribute to this list, improve on and build upon the existing ideas, and propose potential solutions to some of the open problems.

This is an on-going list of development and research ideas. 

<!--1. Blockchain setting
    1. Scalable zk-rollup
    2. Cross chain: trust, data and privacy
    3. Universal layer for proof aggregation
    4. Verifiable gaming and open worlds
    5. Formal verification
    6. MEV
    7. State growth problem
    8. Private computation blockchain
    9. Faster hashing function (resolved)
2. Non-blockchain setting
1. Accelerating the progression of academic ideas
2. Collaborative dataset curation
3. Verifiable ML inference pipeline
4. Software supply chain security-->


## Blockchain Setting


### Scalable zk-rollup

Generating ZKPs can take considerable time, particularly when we’re dealing with conventional execution environments like Ethereum’s. Among ZK-rollup (ZKR) projects, there is currently a lot of focus on the efficiency of generating proofs. Some projects, like StarkNet, have even introduced new SNARK-friendly languages in the interest of prover efficiency.

However, we would argue that prover efficiency isn’t essential. There are two potential concerns here: latency and compute costs.

Latency isn’t much of an issue because proving is highly parallelizable. The main bottlenecks in most proof systems, such as building Merkle trees (in FRI-based systems), scale naturally to many CPUs or GPUs. Nowadays, we have a variety of practical proof aggregation schemes: Plonky2, Halo, SnarkPack, and so forth. This lets us generate each transaction proof on a separate machine in parallel, then aggregate them later.

What about compute costs? Suppose we have a very inefficient CPU-based prover, which takes a whole CPU-hour to create a typical transaction proof. I can rent a VM from CoreWeave for as little as $0.0125 per vCPU-hour. This is simply negligible compared to typical transaction fees, which are driven by native execution bottlenecks.

So, if proving is easy to scale, does that mean ZKRs have solved the scalability problem? Not so much! Existing ZKR designs make no attempt to scale sequencing, i.e. native execution. Unlike proving, sequencing is not trivially parallelizable.

There are several approaches to enabling parallelism. One is to have transactions declare what state they will interact with, so that nodes can easily parallelize based on the transactions’ dependency graph. Solana is an example of this model. If we want to support more dynamic transactions, with interactions that won’t be known until they’re executed, we can use an optimistic parallelism, as in the Block-STM model.

With those techniques, we’re still limited to the power of one machine, and storage remains a bottleneck. To scale further, we must turn to sharding. Ideally, a sharded system would still allow transactions to atomically interact with state on multiple shards. To support this we must lock state before touching it, as transactional databases do.

As an alternative to locking, Vitalik suggested the idea of “[yanking](https://ethresear.ch/t/cross-shard-contract-yanking/1450).” In a way this makes atomicity the application developer’s responsibility, which isn’t ideal. Another interesting alternative is to use sharding within a node, as in the [Ostraka paper](https://arxiv.org/abs/1907.03331). This makes sharding simpler but makes it harder to run a node, which doesn’t seem like a major issue since light clients enjoy the same security in a succinct blockchain.

### Faster hash function

For recursive FRI-based proofs, we need hashes which are efficient both on CPUs and in arithmetic circuits. Some recent algebraic hashes were designed with arithmetic circuits in mind, but they are more expensive than we'd like. Particularly on CPUs, they are expensive compared to conventional hashes like BLAKE3. (For example, algebraic hashes are frequently 100x - 1000x more efficient to verify inside a proving system, but on the CPUs they are 10x - 100x less efficient than something like BLAKE3.)

It seems that any algebraic hash can be attacked by applying a root-finding algorithm to a low-degree CICO problem. See, for example, Algebraic Attacks against Some Arithmetization-Oriented Primitives. While we can mitigate such attacks with many rounds of arithmetic, making our overall permutation very high-degree, this results in rather expensive hashes.

[Reinforced Concrete](https://eprint.iacr.org/2021/1038.pdf) poses an interesting alternative - it adds a single non-algebraic layer, which helps mitigate such algebraic attacks, to an otherwise algebraic permutation (using lookup tables is a very robust way to thwart algebraic attacks). However, RC only supports a permutation width of 3, which is rather limiting; it cannot be used with small fields like Goldilocks. Can we find a more general hybrid solution?


To solve this problem, Ethereum foundation has been working on a Reinforced Concrete version for this field (Goldilocks here is referred to as 2^64-2^31+1). You may see it at the end of this [talk](https://youtu.be/SXnb7T9YATs).

One feature that stands out to us in Reinforced Concrete is that there is only one round with the lookup table; all the other rounds are arithmetization-friendly. There are a bunch of attack techniques that allow you to skip one round at the beginning or at the end. To our knowledge, none of them have been extended to skipping rounds in the middle, but it wouldn't be surprising if it were developed. It would be much more reassuring if a hash function used a lookup table in many rounds (i.e., every round or every other round). 

Lookup tables are kind of expensive because you need to 

- decompose your field element and 
- recompose the looked up elements, such that the entire procedure is a permutation over the field, not counting the lookup table itself. The correct decomposition and recomposition is arithmetically expensive.


You can use the structure of the prime 2^64 - 2^32 + 1 to decompose a field element into two u32s with provable correctness (and recompose it afterwards), but then a u32 is still too large for a lookup table – and even that only works if you find a u32 lookup table that sends pairs of u32's that represent valid field elements to pairs of u32's that represent valid field elements. Can we come up with something that is on par with [Rescue-Prime](https://eprint.iacr.org/2020/1143.pdf) in terms of arithmetic complexity? For generic primes, we think it's completely hopeless.

### Cross chain: trust, data and privacy

ZKP could potentially “fix the bridge problem” - i.e. the transfer of assets from one chain to another without going through any centralized exchange[^1], the single use case that suffered [$2 billion](https://blog.chainalysis.com/reports/cross-chain-bridge-hacks-2022/) loss in 2021 and 2022 from [hacks](https://messari.io/report/a-year-of-bridge-exploits). Among these incidents, the top root causes are leaked private keys from negligent operators, and bugs in cross-chain event verification code

Bridges generally function by locking assets on the source chain while simultaneously releasing some equivalent assets on the target chain. At the core of the bridge problem is how a bridge smart contract on the target chain can reliably verify assets are indeed locked in the source chain. When such proofs can be forged (either through operator’s private key or by exploiting bugs), the bridge gets hacked and all assets locked under the bridge may be stolen. Therefore, it makes sense to create mechanisms that minimize the trust given to entities that produce proofs that authorize fund movement, and multiple layers of verifications against those proofs before releasing any fund. A new generation of bridge projects are already doing this. Their approaches fall into the following categories[^2]:



* relying trusted watchers to monitor transactions on hold, and report fraudulent submissions before finalizing them and releasing the funds (Nomad),
* processing sensitive operations (verifying transactions, signing off the release of funds) inside trusted SGX enclaves and verify attestations of execution before finalizing the transactions  (Avalanche)
* using an independent network of validators to trace events and synchronize communications across chains, where trust over the validators is established by economic mechanisms, such as deposits and bonds (Axelar, Map Protocol[^3]) 


* requiring trusted multi-party consensus (LayerZero, Multichain). 

However, these solutions primarily focus on how and where trust is placed and partitioned.  Somewhere in each approach, there are some trusted third-parties which the users may have no idea about, yet their security and good-faith performance of their role are critical for the security of the bridge. If they get hacked, users’ funds locked under the bridge could be completely stolen. For example, watchers can be bribed or made offline temporarily (plus users have to wait for a long time for fraud challenge periods). Tens of different hacks on SGX enclaves are published every year, which can be used to execute malicious transactions. Validators from an independent network usually only post million dollars in bonds, compared to billions of dollars locked under the bridge. The validator network could go down or malfunction, and the validator themselves may be bribed.

Ideally, we should be “trustless” - i.e. in the bridging flow, eliminate the need to trust any third-parties doing their jobs correctly or not getting hacked. Third parties are the entities that are external to the source and target blockchains (and their validators) where the user already trusts implicitly[^4]. This is achievable with ZKP, since we can prove an event (such as an asset-locking transaction) has occurred with ZKP and have the proof verified by smart contract. Here, the challenges are how ZKP can be used efficiently, affordable, and made universally accessible to all users on a variety of blockchain networks, for example: 



* different blockchain networks use different consensus mechanisms, signatures, curves, and each combination may require construction of new circuits;
* With existing tools, generating proofs for very complex circuits could consume hundreds to thousands of gigabytes in memory and hours of computations; 
* ZKP verification and recording events on-chain frequently can be very expensive

The concept of creating a “trustless” bridge is not new. Multiple projects have proposed customized variations[^5]. Some of them have already achieved some success. The NEAR bridge[^6]  is based on a trustless architecture with the exception that it relies on economic incentives and watchers to secure bridging assets from NEAR to Ethereum. The smart contract light client on Ethereum requires anyone who submits block headers (summarizing events from NEAR) on the light client (on Ethereum) to first post a bond in Ether, and allows watchers (which can be anyone) to challenge the submissions, show proof that some events are fraudulent, and receive rewards taken from submitter’s bonds upon a successful challenge. The drawback created by this exception is similar to the approach by Nomad as discussed previously. The transactions must be held back for 16 hours before they can be finalized, to create enough economic incentives and give watchers enough time to challenge the submitted block headers. If watchers took a long time to identify frauds, all transactions in-between must be rolled back. It also creates a trust assumption that some watchers will always be online and faithfully monitor all submitted events to catch frauds. If all watchers are attacked, bribed, or made offline, the mechanism may collapse.

Instead of creating an exception, we could verify the submitted block headers using ZKP. NEAR validators sign the block headers using Ed25519. A zero knowledge proof of a single Ed25519 signature can be generated offline within a few seconds, and verified on-chain using a small amount of gas (close to minting 3-5 NFTs). There are still challenges remaining regarding how all (100+) validators’ signatures can be efficiently verified and aggregated, and how multiple block headers can be submitted, verified, and committed in an expensive way. 

Finally, we should also keep in mind that since only the occurrence of events need to be proved by ZKP, the functionality of “bridging” can be extended to cross-chain communication, such as allowing apps to send messages from one chain to another[^7], thereby support use cases other than mere transferring of assets[^8]. In some use cases, the communication needs to be private or semi-private[^9]. Nonetheless, to make these happen in reality, we need to build the tools and infrastructure, to name a few: ZKP circuits for signature verification, cross-chain communication data formats, standardized proof formats and verification flow, data serialization packages, efficient cross-chain communication and handshake protocol, cross-chain message encryption packages, and many others. For developers, each of these areas could present interesting challenges and enormous opportunities.


### Universal layer for proof aggregation and composition

Proof composition is a powerful technique that allows generating proofs of proofs. At a high-level, composition allows developers to combine proofs in a way that optimally balances the final proof size and verifier runtime. For example, a simple composition may involve combining a proof from a system with a fast prover and large proof size with a proving system with a slow prover and a small proof size, yielding a final proof that is small and fast to verify. Composition may be realized through several approaches, some of which include recursion, aggregation, and accumulation. Proof composition may happen within the same proving system, or across different proving systems.

Some use cases for proof composition in the blockchain context include:


* Succinct blockchains
* Proof compression before publishing a proof to an L1 blockchain. This is a common technique for L2 rollups to save on L1 gas costs
* Enabling users to make transactions with private data, as done in Aztec's [zk-rollup](https://medium.com/aztec-protocol/proof-compression-a318f478d575)
* Fractal scaling as originally [proposed](https://medium.com/starkware/fractal-scaling-from-l2-to-l3-7fe238ecfb4f) by StarkWare

While current proof composition deployments are usually part of the same project, over time we will see composed proofs combining proofs that originate from different projects.

Open problems:


* We need developer tooling for seamless proof composition and conversion across different proving systems. Today, this is usually a manual process
* When composing proving systems, it’s important to understand the security of the overall system. We need a formal framework for understanding the security of such systems
* Benchmarking and profiling specific implementations of proving systems, as well as their compositions


Delendum is exploring this area and is open to collaboration on this problem. We are also working on building a consolidated, neutral, and community-driven place for benchmarking performance of zero knowledge proving systems with Risc Zero and Miden. We start with smaller computations (e.g. hash functions, signatures, chains of 100 to 1000 hashes, merkle inclusion proofs of varying tree depths, and recursion) and will eventually move on to larger end-to-end scenarios (e.g., integrity of modified images). If you are interested in contributing to this [benchmark](https://github.com/delendum-xyz/zk-benchmarking), we are collecting the desired matrix from the community and would love to have your input.


### Verifiable computation, gaming and open world

Over the long term, verifiable computation will transform the nature of video games completely. Video games have gone through many rounds of evolution, from the early days of Atari, arcade gaming, home gaming, to multiplayer gaming via LAN and the Internet, and mobile gaming. These evolutions shaped the various natures of video games we now take for granted - rich interactivity, social connectivity, competitive mechanics and matchmaking, immersion, on-demand access to dopamine-driven game loops coupled with audio-visual effects - all against the backdrop of exponential growth in memory capacity, compute power, audio and visual fidelity, and network speed. Yet it is obvious that blockchain will impact video games in ways that are very distinct from their past evolution.

We believe that what distinguishes real world experience from existing gaming experience is the openness of the world we live in, despite that there is a group of industry leaders who think the key to revolutionizing gaming is to break its physical limitations. We do not believe that a closed system, even with the most superior sensual effects and self-adaptive artificial intelligence, will and should dominate the next era of this industry. 

A prediction is thus: blockchain will bring about game 2.0. Enforcing ownership on strongly-immutable execution platforms whose compute power scales with ongoing innovation in verifiable computation (proof system, proof recursion and aggregation, rollup layer architecture, hardware acceleration etc) and whose programming paradigm strives for permissionless composability and interoperability, we are heading toward a future where large swaths of populations borderlessly and directly participate in the creation and evolution of core game systems - not just cosmetic items, game modes, maps, but the underlying character design, AI systems, and physics systems - in modular fashion (composability) and owning their creation across contexts and ecosystems (interoperability).

Following this train of logic, we can briefly derive some areas of opportunities:



- Creator-friendly interfaces: This includes both high level editors and high level programming languages. As an example for editors, Chris Hecker proposed the idea of [Photoshop of AI](https://www.chrishecker.com/Structure_vs_Style) more than a decade ago. As for programming languages, even the most modern programming languages such as Rust are considered low level in the sense that they require intimate understanding of low level systems (memory, thread etc) to be approachable. It will be immensely useful to converge toward a multi-level compiling scheme, exposing expressive creator-centric programming interfaces at the highest level, sparing creators the trouble of “learning how to code” and thinking in machine terms.

- Native implementation of core engine systems: as zkVMs and rollup architectures continue to evolve, it is important for core game engine systems to be built natively with best patterns invented on the fly. This means we need native smart contract patterns for describing physics systems, AI systems, and resource/entity management systems in ways that are modular, extensible, and sufficiently robust and powerful to become standards eventually - only with robust standards will these game 2.0 worlds be interoperable.

Historically, gaming demand played a critical role in driving hardware and computer graphics technologies. We believe verifiable gaming has similar potential with respect to proof systems, proof recursion and aggregation, and hardware acceleration.

The relationship between computing and gaming had been long intertwined. Gaming was an unintended application of computer graphics in the 1990’s. GPU was first created with the intent to capture a niche market in industrial 3D rendering, such as in aerospace, mechanical engineering, architectural design, and others. Surprisingly, the market size grew exponentially because people loved games, and they always needed better GPUs to enjoy the games. Game developers recognized the opportunities, continuously pushed the boundaries of GPU and computer graphics algorithms at the time, and developed new games with exhilarating visual experiences despite the hardware and algorithm limitation at the time.

GPU makers subsequently seized the opportunity, made substantial investments on GPU R&D and worked with game developers to roughly double the performance every 1-2 years. Operating system developers joined later, defined new standards, programming interfaces, and libraries for gaming (e.g. DirectX). Resources from the industry and research institutions started to concentrate on computer graphics technologies to meet the new demand, as seen in areas such as computer vision, 3D rendering, physics, simulation, and others. Over the decade of 2010-2020, new use cases were discovered and GPU and computer graphics technologies went through a new era of transformation, because of applications in machine learning, autonomous driving, robotics, virtual reality, and many more. 

If gaming's trajectory and dynamic with the broader the industry is to be somewhat repeated by verifiable gaming and ZKP, we believe the use cases and practical demands could ultimately lead to many more applications of these technologies than what we could imagine today.


### Formal verification of the zero-knowledge tech stack

We wrote an article on formal verification of ZK constraint systems previously. In general, a lot of work needs to be done. There is not enough emphasis placed on formal verification in the security industry. Based on the observations and arguments presented in the [previous article](https://delendum.xyz/2022/09/04/formal-verification-zk-constraint-systems.html), we think the following will be some interesting directions for future research and development:


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


## Introducing our fellowship program

If you are interested in working together with top experts to explore practical use cases of new research ideas, or to work on one of the topics above, please consider applying to our [fellowship program](https://delendum.xyz/fellow). We are also open to development proposals for concrete solutions of other significant problems. 

For a research fellow, you will be collaborating with researchers on original, exploratory topics that lead to the creation of new use cases in practice. For a developer fellow, you will be working with other builders to create new products, with feedback from on product feasibility and implementation.

We will evaluate applications monthly on a rolling basis. Typically, we will get back to you within a week. Since we have very limited capacity (3 fellows per month), the early applicants in that month generally get a better chance. If you have any questions, please do not hesitate to contact us at research@delendum.xyz.

__________________________________

_If you’re interested in further discussions on this topic or working together on this subject, please consider joining our [group chat](https://t.me/+9WAAmCpPRadjOTNh) or reach out to us at research@delendum.xyz._


<!-- Footnotes themselves at the bottom. -->
**Footnotes**

[^1]:
     There are more use cases to bridging than transferring assets. This is discussed later in the article 

[^2]:
     See also “[Security Stack-Up: How Bridges Compare](https://jumpcrypto.com/security-stack-up-how-bridges-compare/)” and “[A classification of various bridging technologies](https://medium.com/harmony-one/harmonys-cross-chain-future-41d02d53b10)” for some different perspectives 

[^3]:

     Also uses ZKP, but creates its own chain to make proof generation and verification substantially easier

[^4]:
     as in, the user is already transacting and placing assets on these two chains

[^5]:
     for example, the Harmony Horizon trustless Ethereum bridge (not launched, still in development) 

[^6]:
     [https://near.org/blog/eth-near-rainbow-bridge/](https://near.org/blog/eth-near-rainbow-bridge/)

[^7]:
     Similar to the apps based on LayerZero

[^8]:
     as also noted by [zkBridge](https://arxiv.org/abs/2210.00264) paper

[^9]:
     For example, a gaming application could use Ethereum for gaming asset trading and storage, and use another faster chain for game state update and interactions between players, which should not be revealed

[^10]:
     See 3.2 section of this paper: [https://hal.archives-ouvertes.fr/hal-03518757/document](https://hal.archives-ouvertes.fr/hal-03518757/document)

[^11]:
     Applications of zero knowledge proofs to portions of ML pipelines is explored by Singh, Dayama, and Pandit in [https://eprint.iacr.org/2021/1633](https://eprint.iacr.org/2021/1633)

[^12]:
     An example from the widely used ML model and code repository service HuggingFace: [https://huggingface.co/docs/transformers/main_classes/pipelines](https://huggingface.co/docs/transformers/main_classes/pipelines)

[^13]:
     The exact means of providing access to a private model is an interesting design space which we don't explore here

[^14]:
     Github has support for viewing the [dependency graph](https://docs.github.com/en/code-security/supply-chain-security/understanding-your-software-supply-chain/about-the-dependency-graph), and automation for [crawling the graph](https://docs.github.com/en/code-security/dependabot/dependabot-alerts/about-dependabot-alerts)

[^15]:
     This is a more involved use case, and overlaps heavily with formal verification. Recursive or aggregated proofs of integrity for formally verified programs is a longer run possibility

