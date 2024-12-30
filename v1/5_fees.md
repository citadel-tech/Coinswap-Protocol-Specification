# Fees

This document outlines the fees that a `Taker` has to pay to a `Maker` when engaging in a `Coinswap`.

## Types of fees Taker has to pay

### 1) **Coinswap Fees**

`Coinswap fees` are charged by Makers for facilitating the coinswap. These fees compensate Makers for their services and are calculated based on specific parameters. The formula for calculating the Coinswap fee is:

```
coinswap_fee = BASE_FEE + Amount Relative Fee + Time Relative Fee
```

Where:

- `Amount Relative Fee` = `(swap_amount * AMOUNT_RELATIVE_FEE_PCT) / 100`
- `Time Relative Fee` = `(swap_amount * REFUND_LOCKTIME * TIME_RELATIVE_FEE_PCT) / 100`

**Definitions:**

- `swap_amount`: The amount the Taker wants to swap.
- `REFUND_LOCKTIME`: The time duration the Maker must wait to claim a refund in case of an unsuccessful coinswap.

**Fee Parameters for Coinswap:**

- **BASE_FEE**: A fixed fee charged by the Maker for their service with a default value of **1000**. This fee is constant and must be paid for any coinswap, regardless of other factors.

- **AMOUNT_RELATIVE_FEE_PCT**: A percentage fee based on the amount being swapped. The higher the amount, the higher the fee, reflecting increased transaction value and risk for the Maker. The default value is **2.50%**.

- **TIME_RELATIVE_FEE_PCT**: A percentage fee based on the `refund locktime`. The longer the locktime, the higher the fee, reflecting the Maker's extended wait time to claim a refund in case of an unsuccessful coinswap. The default value is **0.10%**.

#### Fee Calculation Example:

For a swap amount of 100,000 sats and a refund locktime of 20 blocks:

- **BASE_FEE** = 1,000 sats
- **Amount Relative Fee** = (100,000 \* 2.5) / 100 = 2,500 sats
- **Time Relative Fee** = (100,000 _ 20 _ 0.1) / 100 = 2,000 sats

**Total Fee:** = 1,000 + 2,500 + 2,000 = 5,500 sats

> **Note**: The default fee rates are designed to asymptotically approach `5-10%` of the swap amount as the swap amount and refund locktime increase.

---

### 2) **Mining Fees**

In addition to the Coinswap fees, the Taker is also responsible for paying the **Maker's mining fees**. These are the fees associated with the funding transactions that the Maker broadcasts as part of the coinswap. Each funding transaction requires a miner's fee to incentivize miners to include the transaction in a block. These fees are charge from the Taker and paid separately from the Coinswap fees.

> **Note**: Mining fees are **not** included in the calculation of the Coinswap fee itself.

---

## How Taker pays fees to Maker

Whenever a Maker receives the funding amount from its previous peer (which could either be a Taker or another Maker), the current Maker forwards the amount minus the mining fees for broadcasting the Maker's funding transaction and the Coinswap fee. In equation format:

```text
forwarded_amount = incoming_amount - mining_fees_for_funding_tx - coinswap_fees_by_current_maker
```

Once the Coinswap transaction is successfully completed, the Taker receives the swap amount minus all Coinswap fees and mining fees. This can be represented as:

```text
received_swap_amount = initial_swap_amount - all_coinswap_fees - all_mining_fees
```

Therefore, the Taker ultimately pays all these fees.

However, if the Coinswap transaction fails, the scenario might differ. For more details on successful and unsuccessful Coinswap outcomes, refer to the abort scenarios [here](./6_security.md).

---
