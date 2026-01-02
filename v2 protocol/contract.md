# Contract Transactions
The foundation of Coinswap V2 protocol is built on top of a single `Contract Transaction` that includes both the mutlisig and HTLC logic in a single taproot tree.

Unlike V1, V2 doesn't need a separate contract transaction. Reducing the fees of swap failure and recovation transaction, while reducing the number of messaging steps
in the protocol and makes swaps faster.

The mutlisig is expressed with a 2of2 Musig2 in the taproot keypath spending.

The HTLC is expressed as a taproot tree of two leaves.
 - leaf 1: Timelocked to the sender.
 - leaf 2: Hashlocked to the receiver.

## Transaction Types

All on-chain flow uses single output P2TR (Pay-to-Taproot) contract outputs.

### Contract Transactions

Each “hop” in the cycle creates a contract of:

```
P2TR Output:
├── Internal Key: MuSig2_KeyAgg(partyA_pubkey, partyB_pubkey)
├── Script Tree:
│   ├── Hashlock Script:   OP_SHA256 <hash> OP_EQUALVERIFY <receiver_pubkey> OP_CHECKSIG
│   └── Timelock Script:   <locktime> OP_CLTV OP_DROP <sender_pubkey> OP_CHECKSIG
└── TapTweak: Derived from merkle root of script tree
```

Where:
- **partyA, partyB** are the adjacent parties for this hop
- **hash** is set by the taker, for all hops (single preimage per swap)
- **locktime** is staggered as in Lightning to guarantee atomicity

#### Script snippets

- Hashlock:
  ```text
  OP_SHA256 <hash> OP_EQUALVERIFY <receiver_pubkey> OP_CHECKSIG
  ```
- Timelock:
  ```text
  <locktime> OP_CLTV OP_DROP <sender_pubkey> OP_CHECKSIG
  ```

Script tree and P2TR output are constructed using a function:
```rust
create_taproot_script(hashlock_script, timelock_script, internal_pubkey)
```

***

### Spending Paths

Each output can be spent in one of three ways:

1. **Cooperative Path (MuSig2, Happy Path):**
   - Both parties hold keys: funds can be claimed immediately with aggregated Schnorr signature.
   - No script revealed on-chain.

2. **Hashlock Path (Receiver, Non-cooperative):**
   - Receiver uses preimage with hashlock script if party fails to provide MuSig2 cooperation.
   - Script path is revealed.

3. **Timelock Path (Sender Recovery):**
   - Sender reclaims coins after expiration if receiver does not claim.
   - Also reveals script path.

#### Cooperative spend
```text
input.witness = [aggregated_schnorr_sig]
```

#### Hashlock spend
```text
input.witness = [schnorr_sig, preimage, hashlock_script, control_block]
```

#### Timelock spend
```text
input.witness = [schnorr_sig, <empty>, timelock_script, control_block]
```

***

## Core Protocol Flow

```text
Taker ──→ Maker0 ──→ Maker1 ──→ ... ──→ Taker
```

Funds flow cyclically, transfers are private and break on-chain linkage.

### Phase 1: Discovery & Negotiation

- Taker queries each maker for parameters and fees.
- Negotiates swap amount, timelock, and constructs the cycle.

### Phase 2: Contract Creation (Cyclic Flow)

- For each hop: contracts are constructed as Taproot outputs with MuSig2 keys, hashlock/timelock scripts.
- Contracts are chained so each participant only sees their respective adjacent links.  
- Construction functions:
    - `create_hashlock_script(hash, pubkey)`
    - `create_timelock_script(locktime, pubkey)`
    - `create_taproot_script(hashlock, timelock, internal_pubkey)`

### Phase 3: Private Key Handover & Sweeping

- After all contracts are live, each party hands over their *outgoing* contract key to the next participant (forward-only—no cycles).
- With both private keys for an incoming contract, the receiver (next party) can independently generate MuSig2 nonces and signatures for immediate cooperative spend—*no further message exchange required*.
- No nonce-sharing is required: each party creates their own MuSig2 session locally.

```text
    Key Handover:
    [Taker out key] → Maker0
    [Maker0 out key] → Maker1
    ...
    [MakerN out key] → Taker
```

***

## Sighash and Signing

Each party computes a taproot sighash of their spending transaction for MuSig2 aggregation:

```rust
let sighash = SighashCache::new(&spending_tx)
    .taproot_key_spend_signature_hash(0, &prevouts, TapSighashType::Default)?;
let msg = Message::from(sighash);
```

- Both parties must compute the identical message for MuSig2 signing.

***

## Fee Model

The swap fee is computed by:

```rust
fn calculate_coinswap_fee(
    swap_amount: u64,
    refund_locktime: u16,
    base_fee: u64,
    amt_rel_fee_pct: f64,
    time_rel_fee_pct: f64,
) -> u64 {
    total_fee = base_fee
        + (swap_amount * amt_rel_fee_pct) / 100
        + (swap_amount * refund_locktime * time_rel_fee_pct) / 100;
}
```

***

## MuSig2 API Outline

- **Key aggregation:** `get_aggregated_pubkey(pubkey1, pubkey2) -> XOnlyPublicKey`
- **Nonce generation:** `generate_new_nonce_pair(pubkey) -> (SecretNonce, PublicNonce)`
- **Partial signing:** `generate_partial_signature(message, agg_nonce, sec_nonce, keypair, tap_tweak, pubkeys)`
- **Signature aggregation:** `aggregate_partial_signatures(message, agg_nonce, tap_tweak, partial_sigs, pubkeys)`

***