# Messages

This section describes the messages exchanged between the participants in the Coinswap protocol. Each message is defined by its type, purpose, and the data it contains. The messages are categorized based on the role of the participant sending or receiving them, such as `TakerHello`, `MakerHello`, `ReqGiveOffer`, and so on.

## TakerHello

This is the first interaction between the taker and the maker. The taker sends a `TakerHello` handshake message to the maker to initiate the swap process. The message contains the protocol version range supported by the taker, allowing the maker to determine compatibility.

```rust
{
    protocol_version_min: u32,
    protocol_version_max: u32
}
```

Approximate maximum message size: 8 bytes

## MakerHello

The maker responds to the taker's `TakerHello` message with a `MakerHello` message. This message contains the maker's protocol version range, allowing the taker to verify compatibility.

```rust
{
    protocol_version_min: u32,
    protocol_version_max: u32
}
```

Approximate maximum message size: 8 bytes

## ReqGiveOffer

The taker sends a `ReqGiveOffer` message to the maker to request an offer for the swap. This message does not contain any additional data.

```rust
{}
```

## RespOffer

The maker responds to the taker's `ReqGiveOffer` message with an `RespOffer` message containing the swap offer details. The offer includes the maker's fee, the required confirmations, the minimum locktime, the maximum and minimum swap sizes, and other relevant parameters.

```rust
{
    absolute_fee_sat: Amount(u64),
    amount_relative_fee_ppb: Amount(u64),
    time_relative_fee_ppb: Amount(u64),
    required_confirms: u64,
    minimum_locktime: u16,
    max_size: u64,
    min_size: u64,
    tweakable_point: PublicKey,
    fidelity: FidelityProof,
}
```

**Fidelity Proof**

```rust
{
    pub bond: FidelityBond,
    pub cert_hash: Hash,
    pub cert_sig: bitcoin::secp256k1::ecdsa::Signature,
}
```

**FidelityBond**


## ReqContractSigsForSender

The taker sends a `ReqContractSigsForSender` message to the maker to request contract signatures for the sender. This message contains the transaction details required for the contract signature.

```rust
{
    pub txs_info: Vec<ContractTxInfoForSender>,
    pub hashvalue: Hash160,
    pub locktime: u16,
}
```

## RespContractSigsForSender

The maker responds to the taker's `ReqContractSigsForSender` message with a `RespContractSigsForSender` message containing the contract signatures for the sender.

```rust
{
    pub sigs: Vec<Signature>,
}
```

## RespProofOfFunding

The taker sends a `RespProofOfFunding` message to the maker to provide proof of funding for the swap. This message contains the confirmed funding transactions, the next Coinswap information, the next locktime, and the next fee rate.

```rust
{
    pub confirmed_funding_txes: Vec<FundingTxInfo>,
    pub next_coinswap_info: Vec<NextHopInfo>,
    pub next_locktime: u16,
    pub next_fee_rate: u64,
}
```

## ReqContractSigsAsRecvrAndSender

The maker sends a `ReqContractSigsAsRecvrAndSender` message to the taker to request contract signatures as the receiver and sender. This message contains the transaction details required for the contract signatures.

```rust
{
    /// Contract Tx by which this maker is receiving Coinswap.
    pub receivers_contract_txs: Vec<Transaction>,
    /// Contract Tx info by which this maker is sending Coinswap.
    pub senders_contract_txs_info: Vec<SenderContractTxInfo>,
}
```

## ReqContractSigsForRecvr

The taker sends a `ReqContractSigsForRecvr` message to the maker to request contract signatures for the receiver. This message contains the transaction details required for the contract signature.

```rust
{
    pub txs: Vec<ContractTxInfoForRecvr>,
}
```

## RespContractSigsForRecvr

The maker responds to the taker's `ReqContractSigsForRecvr` message with a `RespContractSigsForRecvr` message containing the contract signatures for the receiver.

```rust
{
    pub sigs: Vec<Signature>,
}
```

## RespContractSigsForRecvrAndSender

The taker responds to the maker's `ReqContractSigsAsRecvrAndSender` message with a `RespContractSigsForRecvrAndSender` message containing the contract signatures for the receiver and sender.

```rust
{
    /// Sigs from previous peer for Contract Tx of previous hop, (coinswap received by this Maker).
    pub receivers_sigs: Vec<Signature>,
    /// Sigs from the next peer for Contract Tx of next hop, (coinswap sent by this Maker).
    pub senders_sigs: Vec<Signature>,
}
```

## RespHashPreimage

The taker sends a `RespHashPreimage` message to the maker to provide the hash preimage for the swap. This message contains the hash preimage required for the swap completion.

```rust
{
    pub senders_multisig_redeemscripts: Vec<ScriptBuf>,
    pub receivers_multisig_redeemscripts: Vec<ScriptBuf>,
    pub preimage: [u8; 32],
}
```

## RespPrivKeyHandover

This message is sent by both the taker and the maker to complete the swap. It contains the private keys required for the swap completion.

```rust
{
    pub multisig_privkeys: Vec<MultisigPrivkey>,
}
```
