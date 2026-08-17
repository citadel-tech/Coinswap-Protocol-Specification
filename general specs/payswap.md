# PaySwap

PaySwap is an exact-amount payment mode for Openswap. Instead of sweeping the
final incoming swapcoins to an address controlled by the taker, the taker
settles them directly to a third-party receiver.

It is supported by both the Legacy and Taproot protocols. It does not add
new taker-to-maker messages or require maker support. The receiver is not an
interactive protocol participant. Maker negotiation and contract construction
remain unchanged; the differences are local to the taker's route quote and the
destination of its final settlement transactions.

```text
Payer/Taker -> Maker 0 -> ... -> Maker N -> Payer's incoming swapcoins
                                                    |
                                                    v
                                      Receiver's Bitcoin address
```

## Exact-amount semantics

For an ordinary Openswap, the amount supplied by the taker is the gross route
amount and the taker receives what remains after maker and mining fees. For a
PaySwap, the requested amount is instead the exact amount the receiver must
obtain.

The quote therefore distinguishes:

- **Receiver amount**: the exact value paid to the third party.
- **Settlement budget**: mining-fee headroom reserved for spending the final
  incoming swapcoins through the most expensive available claim path.
- **Route amount**: the gross value that must enter the first maker so that the
  final hop contains the receiver amount plus the settlement budget.
- **Taker funding fee**: the estimated mining fee paid by the taker's wallet on
  top of the route amount.


## Route quote

Let:

- `P` be the exact receiver amount in satoshis;
- `n` be the number of parallel swapcoins (`tx_count`);
- `S_protocol` be the settlement budget for one final incoming swapcoin; and
- `R = P + n * S_protocol` be the amount that must remain after the final maker.

For maker hop `i`, let `x` be the amount entering the hop and define:

```text
maker_fee_i(x) = ceil(
    base_fee_i
    + x * amount_relative_fee_pct_i / 100
    + x * refund_locktime_i * time_relative_fee_pct_i / 100
)

forward_i(x) = x - maker_fee_i(x) - n * per_contract_mining_fee
```

Starting with `R`, the taker walks the selected route backward. At every hop it
finds the smallest gross input whose `forward_i` value is exactly the amount
required by the next hop. It then replays every deduction forward and MUST
obtain `R` exactly before proceeding. A fee schedule for which no exact input
exists is rejected.

The resulting first-hop input is the PaySwap route amount. It is checked against
every selected maker's advertised minimum and maximum size and against the
taker's available balance.

The quote is bound to the selected route and the offers used to solve it. If a
maker changes its fee terms or fails during negotiation, the taker MUST abort
before broadcasting funding. It MUST NOT substitute a spare maker without
solving and confirming a new quote.

## Settlement budget

The settlement budget is sized for the most expensive path by which the taker
may have to claim each final incoming swapcoin:

| Protocol | Budgeted path | Reference virtual size |
| --- | --- | ---: |
| Legacy | Publish the contract transaction, then spend it through the hashlock | `150 + 150 vB` |
| Taproot | Spend the contract output through the hashlock script path | `155 vB` |

For a minimum fee rate of `2 sat/vB`, these budgets are `600 sats` per Legacy
swapcoin and `310 sats` per Taproot swapcoin. Implementations using another fee
rate MUST recompute the budgets from the same worst-case paths.

On the cooperative path, settlement is cheaper: Legacy spends the 2-of-2
funding output directly, while Taproot uses the key path. The receiver output
remains exact, so any unused budget becomes additional mining fee. PaySwap does
not create a change output back to the taker because doing so would link the
taker's wallet to the receiver's settlement.

## Protocol flow

1. The taker accepts a receiver address and an exact receiver amount.
2. The address is checked against the taker's Bitcoin network. The number of
   swapcoins MUST be non-zero, and the requested amount MUST be at least
   `546 * n` satoshis so every settlement output can clear the dust floor.
3. After selecting makers, the taker snapshots their offers and solves the
   gross route amount as described above.
4. The ordinary Legacy or Taproot negotiation and contract exchange proceeds
   using that gross route amount. Existing exact per-hop amount verification
   enforces the quoted value at the final hop.
5. When the final incoming swapcoins are constructed, the taker assigns each
   one a persistent payment target containing the receiver script and exact
   output value. These targets MUST be persisted before settlement or recovery.
6. After private-key handover, the taker spends each final incoming swapcoin
   directly to the receiver. If cooperative settlement is unavailable, the
   same target is used by the taker's hashlock recovery path.
7. A PaySwap is reported as confirmed only after every settlement transaction
   has at least one confirmation and their outputs sum exactly to the requested
   receiver amount.

With `f_j` denoting the value of final incoming swapcoin `j`, its settlement
output is:

```text
receiver_output_j = f_j - S_protocol
sum(receiver_output_j) = P
```

There is one settlement transaction per final incoming swapcoin. Each contains
one receiver output whose value is fixed before signing; the entire input-output
difference is its mining fee.

## Recovery and reporting

The receiver script and amount are binding wallet state on every incoming
swapcoin. Consequently, a restart or a switch from cooperative settlement to hashlock recovery does not
silently redirect the payment to an internal taker address.

A payment result records the requested amount, delivered amount, settlement
transaction IDs, receiver address, and confirmation status. Partial or
unconfirmed settlement MUST NOT be represented as a confirmed payment.

The receiver learns payment finality from the Bitcoin transaction and its
confirmations. PaySwap does not ask for a receiver acknowledgement, or a 
cryptographic commitment by the payer before the settlement transaction is published.

## Privacy considerations

The receiver address is not included in any maker protocol message. Makers see
the same route negotiation and contract data as in a regular openswap. The
receiver output is public when settlement is broadcast, and a maker that
monitors a directly adjacent contract may observe its spend, as with an
ordinary final openswap sweep.

Avoiding a taker change output prevents a direct on-chain link between the
receiver and the taker's wallet.

## Reference command

In the reference taker, `--amount` becomes the exact receiver amount when
`--payment-address` is present:

```shell
taker openswap \
  --amount 500000 \
  --makers 2 \
  --tx-count 3 \
  --protocol taproot \
  --payment-address <receiver-address>
```

Before funding, the taker displays the receiver amount, settlement budget,
route amount, estimated taker funding fee, and total estimated wallet cost.
