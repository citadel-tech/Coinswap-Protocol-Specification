# Architecture

The architecture involves two types of participants: makers and takers. Makers provide liquidity and compete to offer the best rates, while takers initiate swaps and pay fees. The protocol operates through a series of funding and contract transactions, ensuring atomic execution and eliminating the risks of partial trades or asset loss.

## Taker

The taker is the party that initiates the swap. They fetch the offers made by makers, select the most suitable one, and proceed with the swap process. The taker pays the fees associated with the swap and ensures the completion of the transaction.

The taker also serves as the intermediary between the makers, facilitating the exchange of information and ensuring the smooth execution of the swap.

```text
Maker1 <----> Taker <----> Maker2
```

## Maker

The maker is the party that provides liquidity and competes in the market to offer the best rates. Makers run servers that facilitate swaps and maintain market liquidity through a competitive fee structure. They respond to taker requests, lock funds in multisig outputs, and execute the swap process.

## Directory Server

The directory server acts as a lookup service that helps takers discover available makers and their offers. It provides information about the makers' liquidity, fees, and reputation, enabling takers to make informed decisions when selecting a maker for the swap.

## Transactions

The foundation of Coinswap rests on two primary transaction types that work in concert:

### Funding Transactions

These transactions lock coins in 2-of-2 multisig outputs, ensuring that the funds are secure and available for the swap process. Funding transactions are the initial step in the swap process and establish the foundation for the subsequent contract transactions.

The redeem scripts for the funding transactions are constructed as follows:

```shell
OP_PUSHNUM_2 <PubKey1> <PubKey2> OP_PUSHNUM_2 OP_CHECKMULTISIG
```

### Contract Transactions

These transactions establish time-locked redemption conditions and serve as the mechanism for completing the swap. Contract transactions include hash preimages that act as cryptographic proofs, ensuring atomic execution of the swap process. The completion of the swap involves the exchange of private keys, finalizing the transaction.

The redeem scripts for the contract transactions are constructed as follows:

```shell
/*
    opcodes                  | stack after execution
                             |
                             | <sig> <preimage>
    OP_SIZE                  | <sig> <preimage> <size>
    OP_SWAP                  | <sig> <size> <preimage>
    OP_HASH160               | <sig> <size> <hash>
    H(X)                     | <sig> <size> <hash> H(X)
    OP_EQUAL                 | <sig> <size> 1|0
    OP_IF                    |
        pub_hashlock         | <sig> <size> <pub>
        32                   | <sig> <size> <pub> 32
        1                    | <sig> <size> <pub> 32 1
    OP_ELSE                  |
        pub_timelock         | <sig> <size> <pub>
        0                    | <sig> <size> <pub> 0
        locktime             | <sig> <size> <pub> 0 <locktime>
    OP_ENDIF                 |
    OP_CHECKSEQUENCEVERIFY   | <sig> <size> <pub> (32|0) (1|<locktime>)
    OP_DROP                  | <sig> <size> <pub> (32|0)
    OP_ROT                   | <sig> <pub> (32|0) <size>
    OP_EQUALVERIFY           | <sig> <pub>
    OP_CHECKSIG              | true|false
    */

    //spent with witnesses:
    //hashlock case:
    //<hashlock_signature> <preimage len 32>
    //timelock case:
    //<timelock_signature> <empty_vector>
```

## Protocol Architecture

```text
  | Alice           | Bob             | Charlie         |
  |=================|=================|=================|
 0. A unsign htlc ---->               |                 |
 1.               <---- A htlc B/2    |                 |
 2. ***** BROADCAST AND MINE ALICE FUNDING TXES ******  |
 3. A fund+htlc+p ---->               |                 |
 4.                 | B unsign htlc ---->               |
 5.                 |               <---- B htlc C/2    |
 6. ******* BROADCAST AND MINE BOB FUNDING TXES ******* |
 7.                 | B fund+htlc+p ---->               |
 8.               <---------------------- C unsign htlc |
 9.    C htlc A/2 ---------------------->               |
 A. ***** BROADCAST AND MINE CHARLIE FUNDING TXES ***** |
 B.               <---------------------- C fund+htlc+p |
 C. hash preimage ---------------------->               |
 D. hash preimage ---->               |                 |
 E.    privA(A+B) ---->               |                 |
 F.                 |    privB(B+C) ---->               |
 G.               <---------------------- privC(C+A)    |
```

The interaction diagram above illustrates the process of a Coinswap transaction involving three parties: Alice, Bob, and Charlie. The steps are as follows:

0 - Alice creates an unsigned hash time-locked contract (HTLC) transaction.

1 - Alice sends the HTLC transaction to Bob, who signs it and sends it back.

2 - Alice broadcasts and mines her funding transactions.

3 - Alice sends the funding transaction, HTLC transaction and a nonce to Bob.

4 - Bob creates an unsigned HTLC transaction and sends it to Charlie.

5 - Bob receives the signed HTLC transaction from Charlie.

6 - Bob broadcasts and mines his funding transactions.

7 - Bob sends the funding transaction, HTLC transaction, and nonce to Charlie.

8 - Charlie creates an unsigned HTLC transaction and sends it to Alice.

9 - Charlie receives the signed HTLC transaction from Alice.

A. Charlie broadcasts and mines his funding transactions.

B. Charlie sends the funding transaction, HTLC transaction, and nonce to Alice.

C. Alice sends the hash preimage to Charlie.

D. Alice sends the hash preimage to Bob.

E. Alice sends her private key to Bob.

F. Bob sends his private key to Charlie.

G. Charlie sends his private key to Alice.