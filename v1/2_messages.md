# Messages

This section describes the messages exchanged between the participants in the Coinswap protocol. Each message is defined by its type, purpose, and the data it contains. The messages are categorized based on the role of the participant sending or receiving them, such as `TakerHello`, `MakerHello`, `ReqGiveOffer`, and so on.

## Serialization

The messages are serialized in CBOR(Cbor Object Representation) format, a binary data serialization format that is both compact and efficient. The CBOR format is used to encode the message data in a structured and standardized way, allowing for easy parsing and decoding by the participants.

## TakerHello

This is the first interaction between the taker and the maker. The taker sends a `TakerHello` handshake message to the maker to initiate the swap process. The message contains the protocol version range supported by the taker, allowing the maker to determine compatibility.

```rust
{
    protocol_version_min: u32,
    protocol_version_max: u32
}
```

Maximum message data size: **8 bytes**

## MakerHello

The maker responds to the taker's `TakerHello` message with a `MakerHello` message. This message contains the maker's protocol version range, allowing the taker to verify compatibility.

```rust
{
    protocol_version_min: u32,
    protocol_version_max: u32
}
```

Maximum message data size: **8 bytes**

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

Maximum message data size: **144 bytes**

**Fidelity Proof**

```rust
{
    bond: FidelityBond,
    cert_hash: Hash,
    cert_sig: bitcoin::secp256k1::ecdsa::Signature,
}
```

**FidelityBond**

```rust
{
    outpoint: OutPoint,
    amount: Amount,
    lock_time: LockTime,
    pubkey: PublicKey,
    // Height at which the bond was confirmed.
    conf_height: u32,
    // Cert expiry denoted in multiple of difficulty adjustment period (2016 blocks)
    cert_expiry: u64,
}
```

## ReqContractSigsForSender

The taker sends a `ReqContractSigsForSender` message to the maker to request contract signatures for the sender. This message contains the transaction details required for the contract signature.

```rust
{
    txs_info: Vec<ContractTxInfoForSender>,
    hashvalue: Hash160,
    locktime: u16,
}
```

Maximum message data size: **`No. of txs` * 198 + 22 bytes**

## RespContractSigsForSender

The maker responds to the taker's `ReqContractSigsForSender` message with a `RespContractSigsForSender` message containing the contract signatures for the sender.

```rust
{
    sigs: Vec<Signature>,
}
```

Maximum message data size: **`No. of sigs` * 65 bytes**

## RespProofOfFunding

The taker sends a `RespProofOfFunding` message to the maker to provide proof of funding for the swap. This message contains the confirmed funding transactions, the next Coinswap information, the next locktime, and the next fee rate.

```rust
{
    confirmed_funding_txes: Vec<FundingTxInfo>,
    next_coinswap_info: Vec<NextHopInfo>,
    next_locktime: u16,
    next_fee_rate: u64,
}
```

Maximum message data size: **`No. of confirmed_funding_txes` * 234 + `No. of next_coinswap_info` * 130 + 10 bytes**

## ReqContractSigsAsRecvrAndSender

The maker sends a `ReqContractSigsAsRecvrAndSender` message to the taker to request contract signatures as the receiver and sender. This message contains the transaction details required for the contract signatures.

```rust
{
    /// Contract Tx by which this maker is receiving Coinswap.
    receivers_contract_txs: Vec<Transaction>,
    /// Contract Tx info by which this maker is sending Coinswap.
    senders_contract_txs_info: Vec<SenderContractTxInfo>,
}
```

Maximum message data size: **`No. of receivers_contract_txs` * 198 + `No. of senders_contract_txs_info` * 198 bytes**

## ReqContractSigsForRecvr

The taker sends a `ReqContractSigsForRecvr` message to the maker to request contract signatures for the receiver. This message contains the transaction details required for the contract signature.

```rust
{
    txs: Vec<ContractTxInfoForRecvr>,
}
```

Maximum message data size: **`No. of tx` * 165 bytes**

## RespContractSigsForRecvr

The maker responds to the taker's `ReqContractSigsForRecvr` message with a `RespContractSigsForRecvr` message containing the contract signatures for the receiver.

```rust
{
    sigs: Vec<Signature>,
}
```

Maximum message data size: **`No. of sigs` * 65 bytes**

## RespContractSigsForRecvrAndSender

The taker responds to the maker's `ReqContractSigsAsRecvrAndSender` message with a `RespContractSigsForRecvrAndSender` message containing the contract signatures for the receiver and sender.

```rust
{
    /// Sigs from previous peer for Contract Tx of previous hop, (coinswap received by this Maker).
    receivers_sigs: Vec<Signature>,
    /// Sigs from the next peer for Contract Tx of next hop, (coinswap sent by this Maker).
    senders_sigs: Vec<Signature>,
}
```

Maximum message data size: **`No. of receivers_sigs` * 65 + `No. of senders_sigs` * 65 bytes**

## RespHashPreimage

The taker sends a `RespHashPreimage` message to the maker to provide the hash preimage for the swap. This message contains the hash preimage required for the swap completion.

```rust
{
    senders_multisig_redeemscripts: Vec<ScriptBuf>,
    receivers_multisig_redeemscripts: Vec<ScriptBuf>,
    preimage: [u8; 32],
}
```

Maximum message data size: **`No. of senders_multisig_redeemscripts` * 71 + `No. of receivers_multisig_redeemscripts` * 71 + 32 bytes**

## RespPrivKeyHandover

This message is sent by both the taker and the maker to complete the swap. It contains the private keys required for the swap completion.

```rust
{
    multisig_privkeys: Vec<MultisigPrivkey>,
}
```

Maximum message data size: **`No. of multisig_privkeys` * 135 bytes**

**MultisigPrivkey**

```rust
{
    multisig_redeemscript: ScriptBuf,
    key: SecretKey,
}
```