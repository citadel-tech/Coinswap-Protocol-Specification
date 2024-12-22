# Introduction to Coinswap

Coinswap is a protocol in Bitcoin that enables trustless, decentralized atomic swaps between parties without requiring intermediaries. Originally proposed in a [post by Greg Maxwell](https://bitcointalk.org/index.php?topic=321228.0) in 2013 on bitcointalk.org, the protocol has evolved significantly, with Chris Belcher introducing important improvements in 2020 through the bitcoin-dev mailing list.

This documentation covers the technical details of the protocol. This introduction contains an overview of what Coinswap is, a high-level picture of how it works and the potential impact. [Architecture](./1_architecture.md) contains a more detailed view of the protocol's components and participants, while [Protocol Flow](./3_protocol-flow.md) provides a step-by-step guide to the process of a Coinswap transaction. [Fidelity](./4_fidelity.md) explains the concept of fidelity bonds and their role in the protocol, and [Fees](./5_fees.md) outlines the fee structure.[Security](./6_security.md) discusses the security considerations.

## What is Coinswap?

At its core, Coinswap is an intra-chain atomic swap protocol, meaning it facilitates the exchange of bitcoin between different parties within the same blockchain. Unlike cross-chain atomic swaps that enable exchanges between different cryptocurrencies, Coinswap focuses on improving privacy and fungibility within the Bitcoin network itself. The protocol's key innovation lies in its atomic nature – transactions either complete fully or fail entirely, eliminating the risks of partial trades or asset loss during the exchange process. This atomicity is achieved through sophisticated time-locked contracts and cryptographic techniques.

## Protocol Participants and Mechanics

The Coinswap ecosystem consists of two primary participants: takers and makers. Takers initiate the swap and pay fees, while makers provide liquidity and compete in the market to offer the best rates. Maker servers, run by the makers themselves, facilitate swaps and maintain market liquidity through a competitive fee structure.

The protocol operates through a series of carefully orchestrated steps. It begins with funding transactions that lock coins in 2-of-2 multisig outputs, followed by contract transactions that establish time-locked redemption conditions. Hash preimages serve as cryptographic proofs ensuring atomic execution, and the process concludes with a private key handover that completes the swap.

## Protocol Variants and Implementation

Coinswap can be implemented in two main forms, each serving different privacy needs. The simpler two-party swaps enable direct exchanges between participants, while multi-party routed swaps enhance privacy through multiple makers in a system similar to onion routing. This flexibility allows users to choose the implementation that best suits their privacy requirements and transaction needs.

## Technical Innovation and Market Impact

The protocol's market-driven approach ensures competitive maker fees and robust liquidity provision. Bitcoin holders can now earn returns by providing liquidity, creating a sustainable ecosystem that benefits both privacy-seeking users and those looking to generate yield on their holdings.

## Impact on Bitcoin's Privacy Landscape

Through its innovative design and practical implementation, Coinswap demonstrates how privacy enhancements can be achieved without sacrificing the fundamental principles of trustlessness and decentralization that make Bitcoin valuable. As the protocol continues to mature and gain adoption, it promises to play an increasingly important role in preserving user privacy and enhancing Bitcoin's fungibility.
