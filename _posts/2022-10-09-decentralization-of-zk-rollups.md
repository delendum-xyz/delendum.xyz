---
layout: post
author: Ibrahim Yusufali
title: Decentralization of ZK Rollups
excerpt: "Zero knowledge rollups were used in the past couple of years to solve the scalability constraints in L1 monolithic chains. A lot of new projects have emerged in the space since then, each of them ambitious and hoping to be the “ultimate” solution. However, as it is with every rapidly growing technology there are certain aspects that may be overlooked, in order to facilitate fast execution and entry to the market. We will be doing a deep dive into what we believe is one of those aspects: “decentralization”."
---

_Many thanks to Preston Evans, Shashank Agrawal, Ventali Tan, Daniel Lubarov, Gautam Botrel and James Stearn for feedback._

## Outline

1. Introduction
2. Upgrade Keys
3. Why Proof of Stake (PoS) Might Not Work
4. Proof of Efficiency
5. Proof Generation Outsourcing + MEV Auction
6. Our Proposals
7. Conclusion


## Introduction

Zero knowledge rollups were used in the past couple of years to solve the scalability constraints in L1 monolithic chains. A lot of new projects have emerged in the space since then, each of them ambitious and hoping to be the “ultimate” solution. However, as it is with every rapidly growing technology there are certain aspects that may be overlooked, in order to facilitate fast execution and entry to the market. We will be doing a deep dive into what we believe is one of those aspects: “decentralization”. 

All current zk rollups are centralized. Some do not even have a projected path to decentralization. This is not a good situation to be in. Some features in a truly decentralized zk-rollup which we hope to achieve:



1. Safety / Soundness. Ability to resist a malicious party from taking control over the network, and dictating the present and future state of the network. 
2. Censorship resistance. Everyone should be able to have access to the network and interact with other participants. The network should not be able to deny this right to any individual. It should also be possible for users to withdraw their funds back to L1 or force execute transactions in L2 by paying gas in L1, even if the network is compromised, or the operators are limiting user activities on the network. 
3. Low node / network participant requirements. Individuals should be able to build and run their own node, prover or sequencer without an unreasonable amount of difficulty. Expecting state of the art or highly specialized equipment and high compute resources should be avoided. 
4. Economically aligned incentives. Participants should be rewarded equally for upkeep of the network, in order to encourage new participants to enter. 
5. Liveness. Honest messages should always be included and available to block producers. Chances of halting or lapses in network availability  should be mitigated. 


## Upgrade Keys

Upgrade keys are public keys that have been whitelisted by a rollup’s L1 smart contract, and they have two main benefits:



1. Can fix buggy code, an eventuality that cannot be escaped, as reducing the chance of this happening is very hard. 
2. Can improve the protocol to update with new technology advancements. This is available on L1s through hard forks.

But, this amount of centralized control over an L2 is not very safe as this entity could change the complete state and protocol. 

How do we mitigate this risk? We have identified some ways to move forward:

Abolish upgrade keys permanently. If there are any changes to be made with the original smart contract, bundle them together and deploy a new smart contract with the changes. This is extremely abrupt and harsh. You will then have to entice the miners and users to organically switch over from the “old” network. You can ease into it by airdropping tokens of the “new” rollup to users of the “old” rollup equivalent to their last checked balance, or by accepting the wrapped L1 tokens from the previous L2 rollup.

Another possible solution is to bake in social consensus to the L1 contract, so that if a smart contract change is to be made, it first needs to be agreed by all the participants of the L2 network. However, this opens up to new problems regarding this type of governance, such as the handling of urgent situations, voter fatigue, and power centralization[^1]<sup>,</sup>[^2]. It would also be advisable to add delays, so if the users don’t like an upgrade, they have time to exit, and move their assets back to the L1. 


## Why Proof of Stake (PoS) Might Not Work

Currently, zk-rollups have a centralization in the block creator. To decentralize the block creator, a question is raised on who we decide as the leader / coordinator for a given block. 

There are multiple answers to this question (some we will discuss further in this article): MEV Auction, Burn Auction, Proof of Efficiency and finally Proof of Stake. Most know Proof of Stake from their implementation in L1 blockchains such as Ethereum. However, they can also be used in zk-rollups for leader election, with some teams actively working on this: Polygon Zero, and Starkware. 

The problem with integrating PoS into a zk rollup is that you are unable to fo a hard fork, which provided most of the security guarantees in L1 implementations. This makes it susceptible to DOS and Fake DOS attacks. 

A DOS attack happens when a sufficiently large group of colluding validators with a lot of stake tries to leverage it by minting empty blocks or being extremely exclusive with the transactions they include. This basically allows them to have full control over the transactions in the rollup. They could:



1. Charge high fees in order to include transactions in the block. 
2. Prevent withdrawals. If they have a short position on the network token of the rollup, this could give them economic rewards. 
3. Lock up supplies of certain tokens. This allows them to reap economic benefits from liquidity pools of said tokens. 

In addition to these DOS attacks, there is also a Slashing attack to consider. This is possible because of the slashing condition that punishes stakers who do not create a block within a specified time X. If a block producer refuses to include transactions from other block producers, even with blocks containing their own transactions, in favor of uncling them the stakers will get slashed. If the block producer conducting this attack has a combined active stake with at least a ⅓ of active voting power, then this could be an effective, with-standing attack. Otherwise, it would be fairly rare for a small staker to get two blocks in a row, so it would not have much effect in the long-run. 


## Proof of Efficiency

Proof of Efficiency is a new consensus mechanism put forward to solve the criticisms of PoS on rollups[^3]. It involves a two-step model that splits activities between a sequencer and an aggregator. 

A sequencer is responsible for pre-processing and posting L2 transactions to L1 by submitting a transaction that has the details of the L2 transactions in the calldata field. The sequencer has to pay the L1 gas fee as well as an additional fee in the network token to discourage bad actors. This usage of calldata has certain storage limitations to be considered as well. Even with the intense amount of optimizations it has been through it can only store a limited number of transactions. The average Ethereum block uses about 80KB of data, and with the new EIP-4844 proposal, the calldata (blob) field will provide on average 1MB of data.[^4] This means the base layer could theoretically support a 10X in throughput from rollups. This is discounting compression techniques that could potentially give space savings of up to 70%. There is also a misalignment with this reward and cost structure of the sequencer, as they get charged for the cost of posting the calldata but get rewarded by the L2 gas their transactions generate, which in turn encourages sequencers to favor gas-heavy transactions over simple transactions. During times of network congestion this means that simple transactions may become deprioritized and postponed for an indefinite time, no matter its importance to the user. A solution to this would be to implement a fee structure similar to EIP-1559[^5]<sup>,</sup>[^6], where there is a base fee[^7] and tip. The tip would represent the importance of the transaction to the user and would be the reward for the sequencer. This allows a prioritization of transactions based on importance to the user, rather than a crude one of gas.

To note is also the fact that any individual can become a sequencer by posting the L1 transaction. This does have benefits in terms of censorship resistance (if another sequencer refuses to process my transaction, I can just submit my own transaction), but it does come with some drawbacks. One is that due to this uncoordinated nature, it may not be obvious as to which sequencer I should submit my transaction to, in order for it to be confirmed quicker, taking into consideration that larger batches are more likely to be confirmed by aggregators quicker, and that some aggregators may wait to pool more L2 transactions before posting to L1 in order to reduce the per-unit gas cost. Additionally, the action of sequencers posting the L2 transaction data on L1s could have opened up the possibility for an inter-sequencer MEV attack. An observer of the L1 mempool can observe L2 transactions as they come in, and if they identify a transaction that is of potential MEV value they could have submitted another transaction with the same transactions but now with the desired order / inclusion of transactions and a higher gas tip. This means that the attacker's transaction would have been accepted before the honest sequencers’, resulting in the sequencer losing out on the reward of L2 gas, and having to pay for L1 gas without any reimbursement. But, the team has implemented a ChainID function that is specific to each sequencer, and is included in the signed transaction, so that a user only ever authorizes one sequencer (and only them) to execute a transaction, thereby mitigating the chance of a sandwich attack by another sequencer. 

Once the sequencer has posted the transaction information on L1, it has now become the new virtual future state. In order for this to be confirmed, a validity proof must be submitted on L1 by an aggregator. This soft-confirmation by the virtual state, allows there to be a higher TPS, and lower block interval time. Without this soft-confirmation block interval time would be lengthened to the time taken to generate a proof, which is around 5 minutes, and coordination between provers would constrain throughput as there is no ordering of transactions that is decided on. This is conducted on a “first come, first accepted” basis, in that the aggregator who submits the proof first will be rewarded by the L2 transaction fees. This race format is concerning in that it encourages centralization around the fastest aggregator, which is the one with the most compute resources. 

<img src="https://raw.githubusercontent.com/iyusufali/delendum-xyz-posts-assets/main/2022-10-09-decentralization-of-zk-rollups/poe%20proof%20submission.png" style="display: block;margin-left: auto;margin-right: auto;width:60%;background:white;padding:15px">
<p style="text-align:center; font-style: italic;">Figure 1: Demonstration of Proof of Efficiency validity proof submission.</p>

### The MEV Inequality Problem

Overall, there is also an MEV inequality problem that appears through this separate tiered approach between the sequencer and aggregator. This problem comes from the idea that block producers have a lot of power over the ordering, inclusion and exclusion of transactions which gives them the opportunity to benefit from arbitrage opportunities that may originate from DeFi, ICO transactions. This revenue gained from what we call MEV (Miner Extractable Value) is seen to rival that of the revenue generated from transaction fees. 

The problem is that this high revenue is solely enjoyed by the sequencer, and the aggregator does not get any information or access to this revenue. In its current implementation, sequencers pay aggregators a proving price to generate a validity proof for a block, and through market dynamics we can ensure this would allow for aggregators to cover costs and then some. But in a more optimal world, what we want since aggregators are the more performant and compute heavy party in the system, is to be rewarded proportionally with sequencers. Funneling more rewards to aggregators may help to reduce finality time, increase proving capacity in the network and encourage more decentralization within the aggregators. 


## Proof Generation Outsourcing + MEV Auction

Another approach that has been published is to combine proof generation outsourcing and MEV auctions into a consensus mechanism.[^8] The focus of proof generation outsourcing is to decentralize proof generation by outsourcing it to “rollers”. Then, after this is successful, the sequencer (block proposer) is decentralized through MEV Auction. 


### 

<img src="https://github.com/iyusufali/delendum-xyz-posts-assets/blob/main/2022-10-09-decentralization-of-zk-rollups/MEVA%20contract%20(2).jpg?raw=true" style="display: block;margin-left: auto;margin-right: auto;width:60%">
<p style="text-align:center; font-style: italic;">Figure 2: Transaction flow from executing on L2 to confirming on L1.</p>



### Proof Generation Outsourcing

In order to participate in this process, a roller needs to first deposit network tokens into a smart contract. A reputation score is then calculated for each roller based off of the deposit amount and performance history. A centralized sequencer receives transactions from the L2 mempool and generates a new L2 block. It will then randomly select multiple rollers for each block depending on the calculated ratio score. If the roller sends an invalid proof it will be fined, or if the roller sends the proof later than a time T, its reputation score will be decreased. However, if the proof is valid and sent in a time less than T, it will be rewarded. The sequencer will then aggregate all the proofs from the different rollers and upload the proofs to layer 1 for verification. 

This approach has a benefit in that it adopts a collaborative format rather than a race format. This allows for reduced wasted compute resources but it also helps for decentralization, as it is moving away from a “fastest prover wins” goal, which causes centralization around the fastest prover. The act of parallelizing proof generation across multiple rollers for one block, helps to lower the barriers of entry to being a prover, making it more accessible for someone to run their own node as they might not need to invest as much into equipment. 

This is a similar motive to a federated prover network[^9] where a large block is separated into smaller blocks, which is then broken down into even smaller blocks until you are left with mini 2x2 blocks. The proofs are then generated for this root node (imagine a tree forming, with different layers), and then recursively proven with its neighboring block. This process is repeated until you have a recursive proof for the entire block. In practice, this breaking down process, and building the proof back up from the root takes an immense amount of communication and coordination between the different provers, so much so that it might be more efficient to have a few centralized provers. 

On the other hand, the trend towards decentralization is contrasted by the calculation of the reputation score, which is predominated by the deposit amount. If a roller deposits a larger amount, he is more likely to get more proofs to generate. If this lasts over an extended period of time, it would cause resources to be sucked away from smaller rollers (lack of rewards), causing a cycle which may result in the network becoming dependent on a select group of rollers. 


### MEV Auction

MEV Auction was created to solve the issue that sequencers solely benefit from the MEV revenue (mentioned in the MEV Inequality Problem above) and allow other participants in the network to benefit from the new revenue source instead of just the block producers[^10]. 

The way it works is that there is a MEVA smart contract that auctions off the right to reorder transactions within an N block window to the highest bidder. In this process, block proposers submit transactions to the MEVA contract and then this contract designates a single “sequencer” at a time to roder the transactions, and publish the block. Block proposers in this model would generally be repurposed L1 miners. 

Staying true to its origins, the main benefits from this approach is that the winning bids deposited into the MEVA contract from the auctions can then be used to reward rollers for the blocks they have proven, or to fund public goods on the network, there are a lot of possibilities. 

<img src="https://github.com/iyusufali/delendum-xyz-posts-assets/blob/main/2022-10-09-decentralization-of-zk-rollups/Users%20submit%20txs%20to%20MEVA%20contract.jpg?raw=true" style="display: block;margin-left: auto;margin-right: auto;width:60%">
<p style="text-align:center; font-style: italic;"> Figure 3: Diagram of MEV auction process.</p>

However, there are certain limitations with this approach that need to be considered. Firstly, is the possibility for collusion between the sequencers during the auction process. This could lead to artificially low bids, reducing revenue to the network. Another concern is how to set up the auction for long-term stability, regarding specifically two parameters: duration of time slot, and penalties to reduce the risk of bad actors. The time slot must be large enough to provide economic benefits to the sequencer but must also be low enough so that the health of the network is not compromised if the sequencer is indeed a bad actor. And the penalty must be a balance with this economic benefit that is provided to the sequencer throughout the time slot in order to discourage bad behavior. 


## Our Proposals

These proposals are not supposed to be final, full protocol architectures but rather a thought experiment to test our understanding of what is possible and encourage discussion. They mainly aim to solve the limitations we have outlined in some of the protocols above. 



1. MEV Auction + Single Block Creator

An aspect of the Proof Generation Outsourcing + MEV Auction that was overlooked in our analysis is the coordination between the sequencer that has won the MEV auction and the decentralized network of provers / rollers. How does the sequencer communicate the block being made to the rollers, and how do we make sure there are no sequencer griefing or data withholding attacks that make the provers create proofs for invalid blocks, wasting compute resources? There is currently no perfect solution to this problem that avoids malicious front-runners and data withholding possibilities. Which is why we propose keeping the sequencer and prover in the same entity and then combining that with a variation of an MEV Auction. The difference between this variant and the traditional MEV auction outlined above is that instead of an MEVA smart contract the sequencer gets picked randomly, and solicits bids from builders. The deal is basically that the sequencer gets to keep some percentage of the bid that he accepts, and the rest goes into a smart contract for distribution to reward participants of the smart contract. This way, the chain isn't forced to always accept the highest bid. During a DOS attack, the sequencer can easily accept a lower bid that actually includes transactions. And since sequencers are chosen randomly, at least some of the time we expect them to do this, since many sequencers will value the liveness of the chain more than a small increase in their share of the revenue.

But, there are some drawbacks to this solution too, since malicious sequencers could in theory withhold blocks (violating liveness for the time period the sequencer is allocated). Also, the time taken to generate the validity proof that is uploaded to L1, would be the block interval time, as there is no soft-commitment of transactions. This proof generation time can be around a couple of minutes, which is the opposite of what is expected from a low-latency, high-throughput zk-rollup. 



2. Optimistic Pipeline

An alternate proposal to avoid the long block interval time is to post a soft-confirmation before processing the validity proof for the block. However, to avoid the limitations of the calldata storage, instead of including the raw transaction details in the calldata field the root commitment and state difference is processed. Afterwards, when the validity proof is generated it is uploaded to L1, finalizing the block. The main drawback is that if the proposer-builder does not follow through with the proof after posting the commitment, this would disrupt proposer-builder’s building future blocks, making their work invalid and halting the network as no one else has the perfect information of what the transactions were. We put forward an open question to the community if there are any safe-guards we can add to the protocol to mitigate this malicious sequencer risk.


## Conclusion

From our discussions above it might seem that zk-rollups face some of the same concerns as L1s, and to some extent that is true. A shift of focus to hardware and consensus is expected as we begin to delegate computation, raising questions that L1s have been plagued with for some time now. It would do us good to take inspiration from them, and improve on our learnings. 

It is also important to note that most proposals and solutions proposed above are extremely theoretical. When network participants interact with these protocols in the future, edge cases and unpredictable behavior should be expected. This will only help us to iterate and create more robust ecosystems. zk-rollups are in a unique position in regards to this. For the first time, we are creating decentralization plans after network adoption. Therefore potential ramifications to existing users from faulty decentralization plans could create a difficult situation. 

In closing, there is still progress to be made within zk-rollups to reach our decentralized, ideal vision put forward in the introduction. Hopefully the urgency to reach this ideal state will become more apparent in the coming years. 

__________________________________

_If you are a zero-knowledge proof or cryptography expert interested in further discussions or working together on this subject, please reach out to me at ibrahim@delendum.xyz._

<!-- Footnotes themselves at the bottom. -->
**Footnotes**

[^1]:
     [Brief by Antler on the Challenges facing DAO governance] [https://www.antler.co/blog/daos-and-web3-governance](https://www.antler.co/blog/daos-and-web3-governance) 

[^2]:
     [Talk on Verifiably Secure Governance at the DeFi Security Summit 2022] [https://www.youtube.com/watch?v=-Zzr87YB6TQ](https://www.youtube.com/watch?v=-Zzr87YB6TQ) 

[^3]:
     [Proposal by Polygon Hermez team of Proof of Efficiency] [https://ethresear.ch/t/proof-of-efficiency-a-new-consensus-mechanism-for-zk-rollups/11988](https://ethresear.ch/t/proof-of-efficiency-a-new-consensus-mechanism-for-zk-rollups/11988) 

[^4]:
     [Notes on EIP-4844] [https://notes.ethereum.org/@vbuterin/proto_danksharding_faq#What-is-Danksharding](https://notes.ethereum.org/@vbuterin/proto_danksharding_faq#What-is-Danksharding) 

[^5]:
     [Notes on EIP-1559] [https://notes.ethereum.org/@vbuterin/eip-1559-faq](https://notes.ethereum.org/@vbuterin/eip-1559-faq) 

[^6]:
     It is also important to note that from the beginning of ETH, users could offer different gas prices based on their transaction’s importance. There was a simple gas_price parameter, and miners received gas_price * gas_consumed. It wasn’t explicitly called a tip but still served a similar function - users would use a higher gas_price for more important transactions, giving miners a greater incentive to include them.

[^7]:
     The base fee is burned and  is calculated by a formula that compares the size of the previous block with the target size.

[^8]:
     [Notes on Scroll’s decentralization plans which includes details on decentralized prover network mentioned] [https://hackmd.io/@yezhang/SkmyXzWMY](https://hackmd.io/@yezhang/SkmyXzWMY) 

[^9]:
     [Aztec’s talk on overall rollup decentralization ideas and federated prover network] [https://www.youtube.com/watch?v=rU7EBP3FbNg](https://www.youtube.com/watch?v=rU7EBP3FbNg)

[^10]:
     [Proposal of MEV Auction] [https://ethresear.ch/t/mev-auction-auctioning-transaction-ordering-rights-as-a-solution-to-miner-extractable-value/6788](https://ethresear.ch/t/mev-auction-auctioning-transaction-ordering-rights-as-a-solution-to-miner-extractable-value/6788) 
