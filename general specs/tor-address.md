# Tor Address Derivation

Every Maker in the Coinswap network is reachable via a Tor hidden service. The `.onion` address is derived **deterministically** from the wallet's BIP32 master key. This means that the same wallet always produces the same Tor address — no separate key material needs to be stored, backed up, or managed.

## Source Key

The derivation starts from the wallet's BIP32 master extended private key (`Xpriv`), stored as `WalletStore::master_key` in the wallet database.

```text
WalletStore
└── master_key: Xpriv          ← secp256k1 BIP32 extended private key
        └── private_key        ← secp256k1 scalar (32 bytes)
```

The raw 32-byte scalar is extracted.

## Derivation Steps

The secp256k1 scalar is converted into an Ed25519 expanded private key following the procedure defined in **RFC 8032 Section 5.1.5**.

**Step 1 — SHA-512 hash**

Hash the 32-byte scalar with SHA-512, producing 64 bytes.

```text
h = SHA-512(private_key)
```

**Step 2 — Clamp**

Apply the standard Ed25519 clamping to the hash output:

```text
h[0]  &= 248    // clear the three lowest bits
h[31] &= 127    // clear the highest bit
h[31] |= 64     // set the second-highest bit
```

This clamping ensures `h` is a valid Ed25519 expanded private key scalar (it forces the scalar into the valid cofactor-cleared range required by the Ed25519 group).

**Result**: a 64-byte Ed25519-V3 expanded private key.

## Registering the Hidden Service

The 64-byte key is passed to Tor's control port using the `ADD_ONION` command. Tor internally multiplies this key by the Ed25519 base point to derive the corresponding public key and encodes it as a 56-character Base32 `.onion` hostname.

**Encoding the key for the control protocol**:

```text
tor_private_key = "ED25519-V3:" + base64(64-byte key)
```

**Control port command**:

```text
ADD_ONION {tor_private_key} Flags=Detach Port={COINSWAP_PORT},127.0.0.1:{local_port}
```

Tor responds with:

```text
250-ServiceID=<56-char-base32>
```

The final onion address is `<56-char-base32>.onion`.

## Full Derivation Flow

```text
WalletStore::master_key (Xpriv)
        │
        │
        ▼
 32-byte secp256k1 scalar
        │
        │  SHA-512
        ▼
 64-byte hash
        │
        │  RFC 8032 §5.1.5 clamping
        │  [0]  &= 248
        │  [31] &= 127
        │  [31] |= 64
        ▼
 64-byte Ed25519-V3 expanded key
        │
        │  "ED25519-V3:" + base64(...)
        ▼
 Tor control port  ADD_ONION
        │
        │  250-ServiceID=<base32>
        ▼
 <56-char-base32>.onion
```

## References

- [RFC 8032 §5.1.5](https://www.rfc-editor.org/rfc/rfc8032#section-5.1.5) — Ed25519 key generation: SHA-512 hash and clamping procedure
- [SLIP-10 Test Vector 1 for Ed25519](https://github.com/satoshilabs/slips/blob/master/slip-0010.md#test-vector-1-for-ed25519) — seed → Ed25519 private key derivation test data
- [SLIP-10](https://github.com/satoshilabs/slips/blob/master/slip-0010.md) — HD key derivation for Ed25519 (coinswap derives directly from the BIP32 master key rather than a SLIP-10 path)
- [Tor control port spec — ADD_ONION](https://spec.torproject.org/control-spec/commands.html#add_onion) — `ED25519-V3` key format and ephemeral hidden service registration
