# Privacy

One of the advantages of CoinSwap is the privacy it provides. CoinSwap is a way of trading one coin for another coin in a non-custodial way.
Alice can send Bob coins without Bob knowing the origin of the coins.
This is achieved by breaking the link between the sender and the receiver.

```text
Alice's Address ----> Maker Address 1
```

```text
Maker Address 2 ----> Bob's Address
```

In the above transaction Alice sends coins to Maker Address 1 and the Maker gives the control of the same amount of coins to Alice. Alice sends these coins from Maker Address 2 to Bob's Address.

Privacy is improved because an observer of the blockchain cannot link Alice's Address to Bob's Address as there is no transaction between them.
The only transactions on the blockchain are between Alice's Address and CoinSwap Address 1 and between CoinSwap Address 2 and Bob's Address.
Therefore, CoinSwap breaks the transaction graph heuristic used by blockchain analysis tools.

Another advantage of using CoinSwap is that the transactions blend in with the rest of bitcoin transactions.
CoinSwap would provide much more privacy than existing equal-output CoinJoin implementations at a cheaper cost.

## Routed CoinSwap

A routed CoinSwap with multiple makers would provide even more privacy as no single maker would know the origin and destination of the coins.
This would make it harder for an attacker to link the sender and receiver as all the communication between the makers will happen via the takers.
No maker would be aware of the other makers involved in the swap.

```text
Alice's Address ----> CoinSwap Address 1
```

```text
Coinswap Address 2 ----> Coinswap Address 3
```

```text
Coinswap Address 4 ----> Bob's Address
```

In the above example, there are three transactions: Alice sends coins to CoinSwap Address 1, CoinSwap Address 2 sends coins to CoinSwap Address 3, and CoinSwap Address 4 sends coins to Bob's Address.

## Amount Correlation

This approach still leaves room for amount correlation. If the amounts are the same, it is possible to link the transactions.

```text
Alice's Address ----> CoinSwap Address 1 (1 BTC)
```

```text
Coinswap Address 2 ----> Bob's Address (1 BTC)
```

In the above example, Alice sends 1 BTC to CoinSwap Address 1 and CoinSwap Address 2 sends 1 BTC to Bob's Address. An observer can link the two transactions as the amounts are the same.

To mitigate this, Alice can use multiple transactions with different amounts.

```text
Alice's Address ----> CoinSwap Address 1 (5 BTC)
```

```text
CoinSwap Address 2 ----> Alice(2 BTC)
CoinSwap Address 3 ----> Alice(2 BTC)
CoinSwap Address 4 ----> Alice(1 BTC)
```

Now Alice has received the entire amount from different unrelated transactions which makes correlating amounts excessively difficult.

## Combining Routing with Multiple Transactions

Routing and multi-transaction can be combined to get benefits from both.

```text
Alice
(6 BTC) (8 BTC) (1 BTC)
   |       |       |
   |       |       |
   v       v       v
          Bob
(5 BTC) (5 BTC) (5 BTC)
   |       |       |
   |       |       |
   v       v       v
        Charlie
(9 BTC) (5 BTC) (1 BTC)
   |       |       |
   |       |       |
   v       v       v
        Dennis
(7 BTC) (4 BTC) (4 BTC)
   |       |       |
   |       |       |
   v       v       v
         Alice
```

Where the downward arrow symbol is a single CoinSwap hash-time-locked contract. Each hop uses multiple transactions so no maker (Bob, Charlie, Dennis) is able to use amount correlation to find addresses not directly related to them, but at each hop the total value adds up to the same amount 15 BTC. And all 3 makers must collude in order to track the source and destination of the bitcoins.
