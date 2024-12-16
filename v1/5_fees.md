# Fees

In order to facilitate the coinswap, the makers charge a fee from the taker. This fee has three components:

1. **Absolute Fee** - This is the fee that is constant for all swaps. It is the minimum amount of fee that needs to be paid for any swap irrespective of other parameters of the swap.
   
2. **Amount Relative Fee** - This component is charged as a percentage of the total amount being swapped.
   
3. **Time Relative Fee** - This component is a multiple of the relative locktime period that the maker needs to wait in case the contract transaction is broadcasted to be able to claim the refund.

Note that the this fee does not include the transaction fees and it is calculated separately.