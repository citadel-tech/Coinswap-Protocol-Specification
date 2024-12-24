# Introduction

## What is it?
Coinswap is a Bitcoin scripting protocol that enables trustless, decentralized atomic swaps between multiple participants without relying on intermediaries.
The protocol shares similarities with the Lightning Network, utilizing a two-transaction structure: a funding transaction and a contract transaction.
The funding transaction is a 2-of-2 multisignature (multisig) output, while the contract transaction employs a Hash Time-Locked Contract (HTLC),
locked by a hash to the receiver and time to the sender. When the swap is successful, each party gains exclusive control over the corresponding funding
multisig UTXOs.

## A bit of history
The atomic nature of Coinswap ensures that either the swap completes successfully, or all participants recover their funds via a time-lock.
This is achieved through the release of a hash preimage, typically provided by the swap initiator.
First proposed by Greg Maxwell in a [2013 Bitcoin Talk post](https://bitcointalk.org/index.php?topic=321228.0), this is one of the oldest applications of HTLCs in Bitcoin literature, even before lightning was conceptualized. But it never got implemented beyond the concept draft.

The idea got a significant boost in 2020 with contributions from Chris Belcher. Belcher presented refinements in a [bitcoin-dev mailing list post](https://gnusha.org/pi/bitcoindev/be095fe7-77ca-f471-43e4-981076f48ed2@riseup.net/) and started a [proof-of-concept implementation](https://github.com/bitcoin-teleport/teleport-transactions) around late 2020.

Unfortunately, Belcher halted work in mid-2021 due to health complications, and the project stalled. In July 2022, Citadel developers resumed the initiative, building on Belcher’s original code.

## Our mission
Belcher’s current status is unknown, and we wish him well. As a tribute, the Citadel implementation of [coinswap](https://github.com/citadel-tech/coinswap)
maintains its lineage to Belcher’s work. The commit history reflects the evolution of the project, starting from [his first commit](https://github.com/bitcoin-teleport/teleport-transactions/tree/9d1ba50ce7a9dca4f1470a555eb32615a8360513). Our mission is to complete what he started.


## Scope of the project
While initially designed to ["massively improve Bitcoin fungibility"](https://gist.github.com/chris-belcher/9144bd57a91c194e332fb5ca371d0964), Coinswap has broader implications for the
Bitcoin ecosystem. Decentralized cross-chain atomic swaps could enhance scalability by facilitating interoperability between Bitcoin and other UTXO-based sidechains like
Liquid or Spacechain. In high-fee environments, Coinswap could enable users to swap into low fee sidechains for payments and later return to Bitcoin L1 cold storage. For federated sidechains like Liquid, this eliminates the reliance on PegIn/PegOut mechanisms, offering a seamless SwapIn/SwapOut experience and enhancing market liquidity.

Atomic swaps are not new. Services like [Boltz](https://boltz.exchange/) already provide swaps between Bitcoin and Lightning using [submarine swaps](https://docs.lightning.engineering/the-lightning-network/multihop-payments/understanding-submarine-swaps) and are exploring Bitcoin-to-Liquid atomic swaps with similar HTLC contracts. However, all existing cross-chain swap solutions are centralized, compromising user privacy and exposing funds to theft or seizure. Coinswap’s fully decentralized design addresses these vulnerabilities.

The Coinswap protocol employs a "smart-client, dumb-server" model. `Taker` clients coordinate the swap process, communicating with multiple `Maker` servers. Makers only respond to protocol messages from Takers and have no visibility into the full swap route or communication with other Makers. This lightweight, scalable architecture minimizes operational complexity for Makers, so node runners can run it with minimal capital. To prevent fraud, the protocol incorporates [fidelity bonds](/v1/4_fidelity.md), and both Maker and Taker applications include built-in watchtower functionality for recovery in case of malicious behavior. The protocol exclusively communicates over Tor to preserve privacy, and a simple `DNS` server facilitates market discovery. Future releases will explore more decentralized DNS solutions.

## Impact on Bitcoin ecosystem
This project re-imagines atomic swaps by layering a decentralized communication protocol atop them, creating a distributed swap market with no central points of failure. Any node operator can run a Maker server akin to Boltz, bridging multiple chains and earning swap revenues. As a swap practically disjoins the transaction graph, one side effect of the protocol is, it also provides passive fungibility for non coinswap users, by having the total anonymity set equal to all P2SH UTXOs. Every P2SH UTXO could potentially be a swap, thus "massively improving fungibility". That was Belcher's vision.

We believe that without sufficient decentralization and economic incentives for node operators as well as clients, Belcher’s vision will remain unrealized. This project does not compete with existing swap services but provides open-source infrastructure to empower anyone to deploy scalable swap servers. By prioritizing decentralization, aligning incentives and creating a marketplace, the project aims to get atomic swaps adopted in Bitcoin ecosystem, to eventually fulfill Belcher’s vision of "massively improving Bitcoin’s fungibility."

## Build and run
Building from a simple POC, the project now has all the components to compose a decentralized swap market that resists censorship, DDoS attacks, and protocol breaches. All protocol components—the Maker server, Taker client and DNS system implemented in Rust—are available in the [repository](https://github.com/citadel-tech/coinswap), along with operational instructions.

The [v0.1.0 release](#) marks the first public beta release of Coinswap, available for testing on Signet. Follow the [demo documentation](#) for setup instructions. Please open bug reports or runtime problems in our [issues](https://github.com/citadel-tech/coinswap/issues), take part in the development discussions in our [community](https://discord.gg/7zsry4sbdK).

## Further reads
This documentation provides a comprehensive technical overview of the protocol. This introduction outlines Coinswap’s purpose, high-level functionality, and potential impact. The [Architecture](./1_architecture.md) section details the protocol's components, while [Protocol Flow](./3_protocol-flow.md) explains transaction mechanics. [Fidelity](./4_fidelity.md) discusses fidelity bonds, [Fees](./5_fees.md) describes the fee model, and [Security](./6_security.md) addresses protocol safeguards.