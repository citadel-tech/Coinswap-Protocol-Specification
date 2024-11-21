# Protocol Flow

This section outlines the step-by-step process of a Coinswap transaction, detailing the actions taken by the taker, maker, and directory server. The protocol flow is divided into three main stages: initialization, contract execution, and completion. Each stage involves a series of interactions between the participants, culminating in the successful completion of the swap.

We will showcase the protocol flow in a scenario where we have one taker(Alice) and two makers(Bob and Charlie).

From | To | Message | Description
--- | --- | --- | ---
Alice | Bob | `TakerHello` | Alice initiates the swap process by sending a `TakerHello` message to Bob, indicating the protocol version range supported by Alice.
Bob | Alice | `MakerHello` | Bob responds to Alice's `TakerHello` message with a `MakerHello` message, confirming compatibility and establishing the protocol version range.
Alice | Charlie | `TakerHello` | Alice sends a `TakerHello` message to Charlie, initiating the swap process with the second maker.
Charlie | Alice | `MakerHello` | Charlie responds to Alice's `TakerHello` message with a `MakerHello` message, confirming compatibility and establishing the protocol version range.
Alice | Bob | `ReqGiveOffer` | Alice requests an offer from Bob for the swap.
Alice | Charlie | `ReqGiveOffer` | Alice requests an offer from Charlie for the swap.
Bob | Alice | `RespOffer` | Bob responds to Alice's `ReqGiveOffer` message with an `RespOffer` message containing the swap offer details.
Charlie | Alice | `RespOffer` | Charlie responds to Alice's `ReqGiveOffer` message with an `RespOffer` message containing the swap offer details.
Alice | Bob | `ReqContractSigsForSender` | Alice requests contract signatures for the sender from Bob, providing the transaction details required for the contract signature.
Bob | Alice | `RespContractSigsForSender` | Bob responds to Alice's `ReqContractSigsForSender` message with the required contract signatures for the sender.
Alice | | | Alice creates and broadcasts the funding transaction to lock the coins in a 2-of-2 multisig output with Bob.
Alice | Bob | `RespProofOfFunding` | Alice sends the proof of funding to Bob, confirming the completion of the funding transaction.
Bob | | | Bob verifies the proof of funding and acknowledges the completion of the funding transaction.
Bob | Alice | `ReqContractSigsAsRecvrAndSender` | Bob requests contract signatures for both the receiver and sender from Alice, providing the transaction details required for the contract signature.
Alice | Charlie | `ReqContractSigsForSender` | Alice requests contract signatures for the sender from Charlie, providing the transaction details required for the contract signature.
Charlie | Alice | `RespContractSigsForSender` | Charlie responds to Alice's `ReqContractSigsForSender` message with the required contract signatures for the sender.
Alice | Bob | `RespContractSigsAsRecvrAndSender` | Alice responds to Bob's `ReqContractSigsAsRecvrAndSender` message with the required contract signatures for both the receiver and sender.
Bob | | | Bob creates and broadcasts the funding transaction to lock the coins in a 2-of-2 multisig output with Charlie.
Alice | Charlie | `RespProofOfFunding` | Alice sends the proof of funding to Charlie, confirming the completion of the funding transaction.
Charlie | | | Charlie verifies the proof of funding and acknowledges the completion of the funding transaction.
Charlie | Alice | `ReqContractSigsAsRecvrAndSender` | Charlie requests contract signatures for both the receiver and sender from Alice, providing the transaction details required for the contract signature.
Alice | Charlie | `RespContractSigsAsRecvrAndSender` | Alice responds to Charlie's `ReqContractSigsAsRecvrAndSender` message with the required contract signatures for both the receiver and sender.
Charlie | | | Charlie creates and broadcasts the funding transaction to lock the coins in a 2-of-2 multisig output with Alice.
Alice | Bob | `RespHashPreimage` | Alice sends the hash preimage to Bob
Alice | Charlie | `RespHashPreimage` | Alice sends the hash preimage to Charlie
Bob | | | Bob verifies the hash preimage and acknowledges the completion of the hash preimage exchange.
Charlie | | | Charlie verifies the hash preimage and acknowledges the completion of the hash preimage exchange.
Bob | Alice | `RespPrivKeyHandover` | Bob sends the private key to Alice
Alice | Bob | `RespPrivKeyHandover` | Alice sends the private key to Bob
Charlie | Alice | `RespPrivKeyHandover` | Charlie sends the private key to Alice
Alice | Charlie | `RespPrivKeyHandover` | Alice sends the private key to Charlie

----------------
This completes the Coinswap transaction, with the coins successfully swapped between Alice, Bob, and Charlie. The protocol flow demonstrates the trustless and decentralized nature of Coinswap, ensuring secure and private coin exchanges without the need for intermediaries.
