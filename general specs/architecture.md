# Architecture

The coinswap architecture involves two types of participants: makers and takers.
Makers provide liquidity for the swap and compete to offer the best facilitation fee rates, while takers initiate swaps and pay fees.

The protocol operates through a series of transactions. These transactions are of two types in the V1 protocol(funding transactions and contract transactions). In V2 protocol we only have taproot based contract transactions.

## The Taker

The taker is the party that initiates the swap.
They fetch the offers made by different makers, select the most suitable one, and proceed with the swap process. The taker pays the fees associated with the swap and ensures the completion of the transaction.

The taker also serves as the intermediary between the makers, facilitating the exchange of messages and ensuring the smooth execution of the swap. This will be more clear in the [protocol flow](./3_protocol-flow.md) section.

```text
Maker1 <----> Taker <----> Maker2
```

## The Maker

The maker is the party that provides liquidity and competes in the market to offer the best rates.
Makers run servers that facilitate swaps and maintain market liquidity through a competitive fee structure.
They respond to taker requests, lock funds in multisig outputs, and execute the swap process.

Anyone with a Bitcoin fullnode access, and dedicated home server can participate in the Coinswap market as a maker.

## The Marketplace 

The market place is where takers find makers to swap with. The makers are like shops that displays their offered swap rates and liquidity. Along with a valid fidelity bond. Takers chooses makers from the market upon verifying their fidelity proof and choose the makers with lowest fee rates to swap with.

This marketplace is seeded in the Bitcoin Blockchain itself. Each maker while creating their fidelity bonds, also embeds their Tor network address in the same transaction. Creating an 
immutable record of the market data, accessible without trusted-third-party. The market is censorship resistant by the properties of the Bitcoin Blockchain itself.

In order to speed up the market discovery process, the makers also post their fidelity details to a set of nostr relays. The taker upon bootstrapping, instead of scanning the blockchain, it fetches the initial market data from the nostr network. Once initial market discovery is completed the taker keep an watch for all new Bitcoin blocks to check for appearances of new makers in the market. At that stage reliance on the nostr relays are not needed anymore.

This design allows us to create a fully trustless decentralized marketplace without a huge overhead on the discovery process.

The market place cannot be sybill attacked by rough makers, as takers will only swap with makers with valid fidelity bonds, and creating fidelity bonds are provable costly. 