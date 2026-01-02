# Security

In order to keep the protocol secure, we need to mitigate the risks associated with malicious actors and the risk of losing funds.
This section outlines the security measures in place to protect the integrity of the Coinswap protocol, the possible abort scenarios and the mechanisms to prevent them or penalize the malicious actors.

## Aborts

An abort is a scenario where one of the participants in the swap process fails to complete their part of the transaction or disconnects from the network.

### 1. Taker aborts after full setup

This is the situation where the Taker drops connection after broadcasting all the funding transactions.
The Makers identifies this and waits for a timeout for the Taker to come back.
If the Taker doesn't come back within timeout, the Makers broadcasts the contract transactions and reclaims their funds via timelock.
The Taker after coming live again will see unfinished coinswaps in his wallet. He can reclaim his funds via broadcasting his contract transactions and claiming via timelock.

### 2. Maker Drops Before Setup

This is the situation where a Maker prematurely drops connections after doing initial protocol handshake.
This should not necessarily disrupt the round, the Taker will try to find more makers in his address book to carry on as usual.
The Taker will mark this Maker as "bad" and will not swap this maker again.
In case the Taker doesn't find any more makers, it will abort the swap and reclaim his funds via the contract transaction and so will the other makers.

### 3. Maker Drops After Setup

**Case I**: Maker closes connection after receiving a `ContractSigsForRecvrAndSender` and doesn't broadcast its funding txs.
The Taker will wait until a timeout and start recovery after that.

**Case II**: Maker closes connection after sending a `ContractSigsForRecvr`. Funding txs are already broadcasted.
Taker waits for the response until timeout and aborts if the Maker doesn't show up.

**Case III**: Maker closes connection at hash preimage handling. Funding txs are already broadcasted.
Taker waits for the response until timeout; Aborts if the Maker doesn't show up.

## Malice

These are scenarios where a participant tries to cause deliberate harm to the other parties. We will discuss the possible malice scenarios and the mechanisms in place to prevent them.

### 1. Taker Broadcasts contract transactions prematurely

The Makers identify the situation and gets their money back via contract txs. This is a potential DOS on Makers but the Taker would lose money too for doing this.

### 2. Maker Broadcasts contract transactions prematurely

The Taker and other Makers identify the situation and gets their money back via contract txs. This is a potential DOS on other Makers. But the attacker Maker would lose money too in the process.

## Sybil attacks

A Sybil attack is a type of attack where a single entity creates multiple fake identities to gain control of the network. In the context of Coinswap, a malicious actor could host multiple maker servers and gain an unfair advantage in the swap process. To prevent Sybil attacks, the protocol introduces the concept of [**fidelity**](./4_fidelity.md).
