# Messages

This section describes the messages exchanged between participants in the Taproot Coinswap protocol. Each message is defined by its type, purpose, and the data it contains. Messages are categorized by protocol phase and sender/receiver role.
***

## Phase 1: Discovery & Negotiation

### GetOffer

The taker initiates protocol communication, requesting parameters from the makers by sending a `GetOffer` message to each of them.

```rust
{
    protocol_version_min: u32,
    protocol_version_max: u32,
    number_of_transactions: u32,
}
```
Maximum message data size: **12 bytes**

***

### RespOffer

The maker responds taker's message with a `RespOffer` message which contains the following terms for the current swap session -:

```rust
{
    tweakable_point: PublicKey,
    base_fee: u64,
    amount_relative_fee: f64,
    time_relative_fee: f64,
    minimum_locktime: u16,
    fidelity: FidelityProof,
    min_size: u64,
    max_size: u64,
}
```
Maximum message data size: **144 bytes**

**FidelityProof:**
```rust
{
    bond: FidelityBond,
    cert_hash: Hash,
    cert_sig: bitcoin::secp256k1::ecdsa::Signature,
}
```

**FidelityBond:**
```rust
{
    outpoint: OutPoint,
    amount: Amount,
    lock_time: LockTime,
    pubkey: PublicKey,
    conf_height: Option<u32>,
    cert_expiry: Option<u32>,
}
```

***

### SwapDetails

The taker proposes swap specifics by sending `SwapDetails` message to the makers.

```rust
{
    amount: Amount,
    no_of_tx: u8,
    timelock: u16,
}
```
Maximum message data size: **16 bytes**

***

### AckResponse

The maker accepts or rejects the swap details by responding with the message `AckResponse`.

```rust
Ack | Nack
```
Maximum message data size: **1 byte**

***

## Phase 2: Contract Creation (Cyclic Flow)

### SendersContract

Taker creats a contract transaction and broadcasts it,and send data to the next participant via `SendersContract` message.

```rust
{
    contract_txs: Vec<Txid>,
    pubkeys_a: Vec<PublicKey>,
    hashlock_scripts: Vec<ScriptBuf>,
    timelock_scripts: Vec<ScriptBuf>,
    next_party_tweakable_point: PublicKey,
    internal_key: Option<bitcoin::secp256k1::XOnlyPublicKey>,
    tap_tweak: Option<SerializableScalar>,
}
```
Maximum message data size: **variable – scaled by legs count**

***

### SenderContractFromMaker

Maker responds with the contract setup for their link in the cycle.

```rust
{
    contract_txs: Vec<Txid>,
    pubkeys_a: Vec<PublicKey>,
    hashlock_scripts: Vec<ScriptBuf>,
    timelock_scripts: Vec<ScriptBuf>,
    internal_key: Option<bitcoin::secp256k1::XOnlyPublicKey>,
    tap_tweak: Option<SerializableScalar>,
}
```
Maximum message data size: **variable – scaled by legs count**

***

## Phase 3: Private Key Handover & Sweeping

### PrivateKeyHandover

After contract setup, each party passes forward their outgoing contract’s private key for the swap completion.

```rust
{
    secret_key: bitcoin::secp256k1::SecretKey,
}
```
Maximum message data size: **32 bytes**

***

## Supporting Type Definitions

**SerializablePublicNonce**
```rust
{
    nonce: Vec<u8>, // 66 bytes
}
```
**SerializableScalar**
```rust
{
    scalar: Vec<u8>, // 32 bytes
}
```
**SerializablePartialSignature**
```rust
{
    signature: Vec<u8> // 32 bytes
}
```
**Preimage**
```rust
[u8; 32]
```

***

## Message Enums

### Taker ⇒ Maker

```rust
GetOffer(GetOffer)
SwapDetails(SwapDetails)
SendersContract(SendersContract)
PrivateKeyHandover(PrivateKeyHandover)
```

### Maker ⇒ Taker

```rust
RespOffer(Offer)
AckResponse(AckResponse)
SenderContractFromMaker(SenderContractFromMaker)
PrivateKeyHandover(PrivateKeyHandover)
```