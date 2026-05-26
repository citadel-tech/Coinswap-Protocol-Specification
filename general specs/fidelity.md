# Fidelity Bonds

The idea behind fidelity bonds is to increase resistance to Sybil attacks by making it costly to create multiple fake identities.
This concept is borrowed from JoinMarket's [fidelity bonds](https://github.com/JoinMarket-Org/joinmarket-clientserver/blob/master/docs/fidelity-bonds.md).
The current version is a simple implementation of this fidelity bond concept.

## Sybil Attacks

A Sybil attack is a type of attack where a single entity creates multiple fake identities to gain control of a network.
In the context of Coinswap, a malicious actor could host multiple maker servers and gain an unfair advantage in the swap process.
To prevent Sybil attacks, the protocol introduces the concept of **fidelity bonds**.

## What are Fidelity Bonds?

A fidelity bond is a mechanism in which a user deliberately locks a certain amount of bitcoin, making it costly to create a new cryptographic identity.
The lockup is done in such a way that its value can be publicly verified.
In the Coinswap process, takers are more likely to initiate coinswaps with makers who have staked more valuable fidelity bonds.
This makes Sybil attacks significantly more expensive because an attacker would need to lock up a large amount of bitcoin to frequently be chosen as a counterparty in the swaps.

For makers, locking bitcoin as a fidelity bond can increase their participation in coinswaps and, consequently, their fee earnings.
The most practical way to establish a fidelity bond is by sending bitcoin to a time-locked address using the opcode [OP_CHECKLOCKTIMEVERIFY](https://en.bitcoin.it/wiki/Timelock#CheckLockTimeVerify).
In this case, the "sacrifice" comes from the time-value of money, as the bitcoin is locked and unusable for a period.
However, long-term bitcoin holders (or hodlers) can create time-locked fidelity bonds at minimal cost, assuming they don’t plan to use their funds for transactions in the near future.

### Bond Value

The bond value is calculated based on the locked coins, interest rate, and locktime. The formula for bond value is as follows:

```text
bond_value = (locked_coins * (exp(interest_rate * locktime) - 1))^x
```

1. The bond value goes as the (locked_coins)^x of sacrificed value.
   For example if x is 1.3, the current default, and your sacrificed value is 5 BTC then the fidelity bond value is ~ 8.1.
    If instead you sacrificed 6 BTC the value is ~ 10.3.
    The point of this is to create an incentive for makers to lump all their coins into just one bot rather than spreading it over many bots. It makes a sybil attack much more expensive.
2. The longer you lock for the greater the value. The value increases as the interest_rate, which is configurable in the config file with the option interest_rate.
   By default it is 1.5% per annum and because of tyranny-of-the-default takers are unlikely to change it.
   This value is probably not too far from the "real" interest rate, and the system still works fine even if the real rate is something like 3% or 0.1%.
3. The above formula would suggest that if you lock 3 BTC for 10000 years you get a fidelity bond worth 2.03e+85 (~ 2 followed by 85 zeros).
   This does not happen because the sacrificed value is capped at the value of the burned coins.
   So in this example the fidelity bond value would be just ~ 4.17. This feature is not included in the above simplified equation.
4. After the locktime expires and the coins are free to move, the fidelity bond will continue to be valuable, but its value will exponentially drop following the interest rate.
   So it would be good for you as a maker to create a transaction with the UTXO spending it to another time-locked address, but it's not a huge rush (specifically, there's likely no need to pay massive miner fees, you can probably wait until fees are low).

## Protocol Constants

| Constant | Value | Description |
|---|---|---|
| `MIN_FIDELITY_TIMELOCK` | 12,960 blocks | Minimum lock duration (~3 months) |
| `MAX_FIDELITY_TIMELOCK` | 25,920 blocks | Maximum lock duration (~6 months) |

These bounds are enforced during verification — a bond whose relative lock duration falls outside this range is rejected.

## Fidelity Structure

### Fidelity Transaction

This is the transaction that locks the maker's bitcoin in a time-locked output.
The output of this transaction serves as a reference to the fidelity bond.
Since this output is unique to the maker, it can be used to identify the maker in the network.
The current version locks the bitcoin to a **P2WSH** output.

The redeem script for the fidelity output is constructed as follows:

```text
<pubkey> <OP_CHECKSIGVERIFY> <locktime> <OP_CLTV>
```

The fidelity transaction also includes an OP_RETURN output encoding the maker's Tor address and bond locktime:

```text
{onion_address}#{locktime_height}
```

The `.onion` suffix is stripped before encoding — only the base address is stored. This serves as the canonical on-chain record. For discovery, makers also publish their fidelity details to Nostr relays.

### Bond Data

The `Fidelity Bond` structure contains the following data:

1. `outpoint`: The transaction ID and output index of the fidelity bond.
2. `amount`: The amount of bitcoin locked in the fidelity bond.
3. `lock_time`: The absolute block height at which the fidelity bond expires.
4. `pubkey`: The public key of the maker.
5. `conf_height`: The block height at which the fidelity bond was confirmed.
6. `is_spent`: Whether the bond UTXO has been redeemed.
7. `bond_index`: The child index used in the BIP32 derivation path `m/175'/2/<bond_index>`.

### Bond Certificate

The `Fidelity Bond Certificate` is a message that binds the bond to a specific maker identity. It is formatted as:

```text
fidelity-bond-cert|{outpoint}|{pubkey}|{lock_time}|{amount}|{onion_addr}|{tweakable_point}
```

- `outpoint` — the bond UTXO (`txid:vout`)
- `pubkey` — the bond's public key
- `lock_time` — the absolute locktime (block height)
- `amount` — the locked amount in satoshis
- `onion_addr` — the maker's `.onion` address
- `tweakable_point` — the maker's BIP32 identity public key (see [BIP32 Bond Ownership](#bip32-bond-ownership))

The certificate hash is computed using Bitcoin's message signing convention:

```text
cert_hash = SHA256d("\x18Bitcoin Signed Message:\n" + varint(len(cert)) + cert)
```

### Fidelity Proof

The `Fidelity Proof` is a structure that contains all the data required to verify that a `Fidelity Bond` belongs to a specific maker.
This structure is sent to the Taker in the [`RespOffer` message](./2_messages.md#respoffer). The Fidelity Proof contains:

1. **Fidelity Bond** — the full `FidelityBond` structure.
2. **Certificate Hash** — the double SHA256 hash of the certificate message (see above).
3. **Signature** — an ECDSA signature over the certificate hash, signed by the private key corresponding to the bond's public key.

## BIP32 Bond Ownership

Each fidelity bond's `pubkey` is derived deterministically from the maker's identity key (the `tweakable_point`) via BIP32. This cryptographically binds the bond to the maker's identity — a bond cannot be claimed by a different maker.

Derivation path:

```text
m/175'/2/<bond_index>
```

The purpose code `175'` (BIP43 purpose `0x800000AF`) is taken from [BIP175 — Pay-to-Contract Protocol](https://github.com/bitcoin/bips/blob/master/bip-0175.mediawiki).

During verification, the verifier constructs an `Xpub` from the tweakable point and its chain code (following the BIP175/Pay-to-Contract derivation convention), then derives the child key at path `[Normal(2), Normal(bond_index)]`. The derived public key must match the bond's public key.

This ensures a maker cannot reuse another maker's bond.

## Fidelity Verification

Any party can verify a fidelity bond using the `FidelityProof` and the on-chain transaction. Verification consists of six checks performed in order:

**Step 1 — Timelock range**

Compute the relative lock duration: `lock_time - conf_height`. This must fall within `[MIN_FIDELITY_TIMELOCK, MAX_FIDELITY_TIMELOCK]` (12,960–25,920 blocks).

**Step 2 — Bond not expired**

`current_height` ≤ bond locktime. An expired bond is invalid.

**Step 3 — Certificate hash integrity**

Reconstruct the certificate string from the proof fields and re-compute the hash. The result must match the certificate hash carried in the proof.

**Step 4 — BIP32 ownership**

Derive the public key from the maker's tweakable point at path `[Normal(2), Normal(bond_index)]`. The derived key must match the bond's public key. This confirms the bond belongs to the claimed maker identity.

**Step 5 — Script match**

Derive the P2WSH redeem script from the bond's locktime and public key. The `scriptPubKey` of the on-chain output must match the derived script.

**Step 6 — ECDSA signature**

Verify the signature in the proof against the certificate hash using the bond's public key. This proves the maker controls the bond's private key.

If all six checks pass, the fidelity proof is valid.

## Redeeming Fidelity Bond

When the locktime of the fidelity bond expires, the maker can redeem the fidelity bond by spending the output to a new address.
