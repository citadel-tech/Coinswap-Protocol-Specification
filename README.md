# Coinswap Protocol Documentation
This repository contains the complete documentation for the **Coinswap Protocol**, an atomic swap protocol that enables trustless atomic swaps through a decentralized, Sybil-resistant marketplace using Bitcoin-based fidelity bonds. This documentation provides an in-depth overview of the protocol's architecture, functionality, and other important details.

## Documentation layout

### Protocol V1

All documentation for the first version of the protocol is located in the [v1](v1) directory.Reading v1 is recommended to understand the historical context and design motivations that influenced later revisions.

The v1 docs are organized as follows:

1. [Architecture](./v1%20protocol/architecture.md): An explanation of the protocol’s architecture, participants, and transaction types.
2. [Messages](./v1%20protocol/messages.md): A detailed description all messages exchanged during a swap.
3. [Protocol Flow](./v1%20protocol/protocol-flow.md): A Step-by-Step description of a Coinswap.

### Protocol V2

All documentation for the second version of the protocol is located in the [v2](v2) directory.The refinement of protocol and adoption of Taproot and MuSig2 for improved privacy and efficiency.

The v2 docs are organized as follows:

1. [Architecture](./v2%20protocol/architecture.md): Updated architecture, participants, and transaction types for the Taproot–MuSig2-based design.
2. [Messages](./v2%20protocol/messages.md): A detailed specification of all messages exchanged during a swap, including fields and semantics.
3. [Protocol Flow](./v2%20protocol/protocol-flow.md): Step-By-Step description of a Taproot–MuSig2 Coinswap.

### General Documents

All other general documents that applies to both the protocols are as follows:

1. [Introduction](./general%20specs/introduction.md): An overview of the Coinswap Protocol, its history, and core features.
2. [Architecture](./general%20specs/architecture.md): An overview of the Coinswap server, client and the marketplace.
3. [Fidelity](./general%20specs/fidelity.md): Explanation of fidelity bonds and their role in the protocol.
4. [Fees](./general%20specs/fees.md): Overview of how fees are structured and accounted for within Coinswap transactions.
5. [Security](./general%20specs/security.md): A discussion of the security considerations in the Coinswap Protocol.
6. [Privacy](./general%20specs/privacy.md): A discussion on how coinswap provides privacy.
7. [Tor Address Derivation](./general%20specs/tor-address.md): How a maker's Tor onion address is deterministically derived from the wallet master key.
