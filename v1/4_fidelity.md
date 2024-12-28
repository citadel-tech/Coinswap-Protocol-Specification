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

### Bond Data

The `Fidelity Bond` structure contains the following data:

1. `outpoint`: The transaction ID and output index of the fidelity bond.
2. `amount`: The amount of bitcoin locked in the fidelity bond.
3. `lock_time`: The time at which the fidelity bond expires.
4. `pubkey`: The public key of the maker.
5. `conf_height`: The height at which the fidelity bond was confirmed.
6. `cert_expiry`: The expiry of the fidelity bond certificate.

### Bond Certificate

The `Fidelity Bond Certificate` is a message that contains the [Bond Data](#bond-data) in the following format:

```text
fidelity-bond-cert|{outpoint}|{pubkey}|{cert_expiry}|{lock_time}|{amount}|{maker_address}
```

This certificate is used to link the `Fidelity Bond` to the maker's address.

### Fidelity Proof

The `Fidelity Proof` is a structure that contains all the data required to verify that a `Fidelity Bond` belongs to a specific maker.
This structure is sent to the Taker in the [`RespOffer` message](./2_messages.md#respoffer). The Fidelity Proof contains:

1. **Fidelity Bond** structure that has all the parameters of the bond.
2. **Fidelity Bond Certificate Hash**: A double SHA256 hash of the `Fidelity Bond Certificate`
3. **Signature**: The signature of the `Fidelity Bond Certificate Hash` signed by the same private key that corresponds to the public key in the `Fidelity Bond`.

### Verification

Any party can verify the fidelity bond with the `Fidelity Proof` by reconstructing and matching the Bond Certificate Hash, the P2WSH address and validating that the signature corresponds to the public key in the `Fidelity Bond`.
