---
layout: post
author: Delendum Research
title: "ZK System Benchmarking"
excerpt: "comparing the performance of different zero-knowledge proof libraries"
image: "/assets/posts/2023-01-11-zk-system-benchmarking/preview.png"
---

Delendum Research, Miden, Risc Zero

_Many thanks to Bobbin Threadbare, Dominik Schmid, Brian Retford, Tim Cartens, Daniel Lubarov, Andrei Nagornyi and Ventali Tan for feedback._

### Table of Content

- [Introduction](#introduction)
- [Principles](#principles)
- [What are we testing?](#what-are-we-testing)
- [Roadmap](#roadmap)
- [Sample results](#sample-results)
- [Call for collaboration](#call-for-collaboration)


## Introduction

The purpose of this benchmarking project is to produce a collection of standard benchmarks for comparing the performance of different zero-knowledge proof libraries, in an easy-to-read format, including graphs and tables. The first question we ask ourselves is - who is the target audience? 

Is it a) developers, who are not necessarily familiar with the internals of ZK systems, but are looking to make an informed decision based on the tradeoffs involved with using a given proving system for a concrete, real-world use case, or b) individuals who understand the broad underlying primitives and are interested in the performance of key cryptographic building blocks? 

Our goal is to cover both[^1].


## Principles

### Relevant

Our benchmarks aim to measure the performance of realistic, relevant tasks and use cases. This allows third-party engineers to estimate how each proof system would perform in their application.

### Neutral

We do not favor or disfavor any particular proof system. We seek to allow each proof system to show its best features in a variety of realistic test cases.

### Idiomatic

We allow each proof system to interpret each benchmark in whatever way makes the most sense for that system. This allows each proof system to showcase their performance when being used idiomatically.

### Reproducible

Our measurements are automated and reproducible.


## What are we testing?

### Performance

We measure proving time[^2] in various standard hardware environments, thereby allowing each proof system to showcase its real-world behavior when running on commonly available systems.

For each benchmark, we measure the performance of these tasks:

* Proof generating
* Verifying a valid proof
* Rejecting an invalid proof

We start with smaller computations such as iterated hashing, merkle inclusion proof, and recursion etc and will eventually move on to larger end-to-end scenarios.

The following graph explains our approach:

![graph](/assets/posts/2023-01-11-zk-system-benchmarking/benchmarks.png)

### Definitions

Here are the definitions of the three tests mentioned above:

#### Iterated hashing

Iterated hashing is an essential building block for Merkle tree structures and whenever one needs to succinctly commit larger amounts of data.

Compute H(H(H(...H(x)))), where H() is a cryptographic hash function, for some input x. As input x we chose a 32-bytes input [0_u8; 32] and job_size defines the number of iterations.

#### merkle inclusion proof

A merkle inclusion proof proves that a certain leaf is part of a merkle tree. The VM is given the leaf and the merkle path and needs to prove inclusion by iterated hashing. We define the benchmark to pick a Merkle path of depth 32 and then the job_size is the number of Merkle paths we verify. 

[Benchmarks will be presented soon]

#### recursion

[Benchmarks will be presented soon]

### Other factors

The following factors are also very important to consider and we will try to describe them in future: 

#### Security

* What is the security model?
* How many "bits of security" does the system offer?
* Is it post-quantum?
* What hash functions does it support?

#### Ease of building new apps

* How hard is it to write new apps for the platform?
* Does it require custom circuits?
* Does it support custom circuits?
* Are there libraries and developer tools? How mature are they?

#### Upgradability

* Is the VM tightly coupled to its cryptographic core? Or is there an abstraction layer between them?
* If a new breakthrough in ZKPs took place tomorrow, would the VM be able to incorporate the new advances without breaking existing apps?

## Sample results

For iterated hashing, here are sample results on a 64 core Graviton 3 and an Apple M2 machine:

<table>
    <thead>
        <tr>
            <th rowspan=2 colspan=2>Proving time in s</th>
            <th colspan=3>SHA256</th>
            <th colspan=3>BLAKE3</th>
            <th colspan=4>RP64_256</th>
        </tr>
        <tr>
            <th>1</th>
            <th>10</th>
            <th>100</th>
            <th>1</th>
            <th>10</th>
            <th>100</th>
            <th>1</th>
            <th>10</th>
            <th>100</th>
            <th>1000</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td colspan=12>Apple M2 8GB RAM </td>
        </tr>
        <tr>
            <td> </td>
            <td style="text-align:left">Miden</td>
            <td>0.27</td>
            <td>1.91</td>
            <td>40.39</td>
            <td>0.15</td>
            <td>0.96</td>
            <td>9.87</td>
            <td>0.04</td>
            <td>0.03</td>
            <td>0.05</td>
            <td>0.28</td>
        </tr>
        <tr>
            <td> </td>
            <td style="text-align:left">RISC Zero</td>
            <td>0.63</td>
            <td>1.29</td>
            <td>5.48</td>
            <td> </td>
            <td> </td>
            <td> </td>
            <td> </td>
            <td> </td>
            <td> </td>
            <td> </td>
        </tr>
        <tr>
            <td colspan=12>AWS Graviton 3</td>
        </tr>
        <tr>
            <td> </td>
            <td style="text-align:left">Miden</td>
            <td>0.12</td>
            <td>0.49</td>
            <td>3.99</td>
            <td>0.10</td>
            <td>0.33</td>
            <td>2.06</td>
            <td>0.04</td>
            <td>0.04</td>
            <td>0.05</td>
            <td>0.13</td>
        </tr>
        <tr>
            <td> </td>
            <td style="text-align:left">RISC Zero</td>
            <td>0.18</td>
            <td>0.40</td>
            <td>1.59</td>
            <td> </td>
            <td> </td>
            <td> </td>
            <td> </td>
            <td> </td>
            <td> </td>
            <td> </td>
        </tr>
    </tbody>
</table>

* Note that for Miden 100 SHA2 results is 2x longer than what it should be (should actually be closer to 16 seconds). This likely happened because M2 system we use has only 8GB of RAM and it probably started swapping to disk for this benchmark.

___

<table>
    <thead>
        <tr>
            <th rowspan=2 colspan=2>Verification time in ms</th>
            <th colspan=3>SHA256</th>
            <th colspan=3>BLAKE3</th>
            <th colspan=4>RP64_256</th>
        </tr>
        <tr>
            <th>1</th>
            <th>10</th>
            <th>100</th>
            <th>1</th>
            <th>10</th>
            <th>100</th>
            <th>1</th>
            <th>10</th>
            <th>100</th>
            <th>1000</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td colspan=12>Apple M2 8GB RAM </td>
        </tr>
        <tr>
            <td> </td>
            <td style="text-align:left">Miden</td>
            <td>2.26</td>
            <td>2.42</td>
            <td>3.73</td>
            <td>2.68</td>
            <td>2.56</td>
            <td>2.52</td>
            <td>2.44</td>
            <td>2.24</td>
            <td>2.28</td>
            <td>2.42</td>
        </tr>
        <tr>
            <td> </td>
            <td style="text-align:left">RISC Zero</td>
            <td>1.81</td>
            <td>1.92</td>
            <td>2.44</td>
            <td> </td>
            <td> </td>
            <td> </td>
            <td> </td>
            <td> </td>
            <td> </td>
            <td> </td>
        </tr>
        <tr>
            <td colspan=12>AWS Graviton 3</td>
        </tr>
        <tr>
            <td> </td>
            <td style="text-align:left">Miden</td>
            <td>3.05</td>
            <td>3.26</td>
            <td>3.54</td>
            <td>3.27</td>
            <td>3.24</td>
            <td>3.47</td>
            <td>2.79</td>
            <td>2.77</td>
            <td>2.81</td>
            <td>3.04</td>
        </tr>
        <tr>
            <td> </td>
            <td style="text-align:left">RISC Zero</td>
            <td>2.81</td>
            <td>3.03</td>
            <td>4.05</td>
            <td> </td>
            <td> </td>
            <td> </td>
            <td> </td>
            <td> </td>
            <td> </td>
            <td> </td>
        </tr>
    </tbody>
</table>

___
<table>
    <thead>
        <tr>
            <th rowspan=2 colspan=2>Proof sizes in bytes</th>
            <th colspan=3>SHA256</th>
            <th colspan=3>BLAKE3</th>
            <th colspan=4>RP64_256</th>
        </tr>
        <tr>
            <th>1</th>
            <th>10</th>
            <th>100</th>
            <th>1</th>
            <th>10</th>
            <th>100</th>
            <th>1</th>
            <th>10</th>
            <th>100</th>
            <th>1000</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td> </td>
            <td style="text-align:left">Miden</td>
            <td>72356</td>
            <td>89776</td>
            <td>107560</td>
            <td>67699</td>
            <td>83300</td>
            <td>100752</td>
            <td>53203</td>
            <td>53284</td>
            <td>57529</td>
            <td>72742</td>
        </tr>
        <tr>
            <td> </td>
            <td style="text-align:left">RISC Zero</td>
            <td>177684</td>
            <td>187796</td>
            <td>210068</td>
            <td> </td>
            <td> </td>
            <td> </td>
            <td> </td>
            <td> </td>
            <td> </td>
            <td> </td>
        </tr>
       
</table>


## Roadmap

1. Identify key performance metrics for evaluating the efficiency of zero-knowledge proof libraries (completed)
1. Develop a set of test cases that exercise a range of different functionality in the libraries, including creating and verifying proofs for various types of statements (completed)
2. Implement automation scripts for running the test cases and collecting performance data (completed)
3. Set up continuous integration to regularly run the benchmarking suite and track performance over time (ongoing)
4. Collaborate with other organizations to add more libraries, incorporate their test cases and use cases into the benchmarking suite (TODO)
5. Publish the results of the benchmarking efforts in a public repository, along with documentation and scripts for reproducing the benchmarks (completed)
6. Continuously update and improve the benchmarking suite as new features and functionality are added to the libraries (ongoing)


## Call for collaboration

We are seeking collaborators to help us expand and improve our benchmarking efforts. If you are interested in joining us in this effort, there are several ways you can contribute:

* Help with adding benchmarks for additional proving systems.
* Suggest new benchmarks or test cases for us to include in our repository.
* Help us verify and validate the accuracy of the benchmarks and test cases that are already included in the repository.
* Contribute code or scripts to automate the running and reporting of the benchmarks.
* Help us improve the documentation and usability of the repository.

To get involved, please reach out to us via the repository's issue tracker or email research@delendum.xyz. We welcome contributions from individuals and organizations of all experience levels and backgrounds. Together, we can create a comprehensive and reliable benchmarking resource for the ZK community.


<!-- Footnotes themselves at the bottom. -->
## Notes

[^1]:
     You may find arm benchmarking whitepaper ([here](https://documentation-service.arm.com/static/6194d6b0f45f0b1fbf3a8977?token=)) and programming language benchmarks game ([here](https://benchmarksgame-team.pages.debian.net/benchmarksgame/)) interesting

[^2]:
     We don’t measure memory usage right now, but it would be interesting/valuable to add it in future.














