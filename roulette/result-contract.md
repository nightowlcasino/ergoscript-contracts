# Roulette Result Contract
* Author: Krasavice Blasen
* Created: 01-Apr-2022

## Description
This contract contains a user's wager and the house's matched funds. It obtains some randomness to determine the spending path of these assets.
The contract contains different spending paths for each sub-game of roulette.

## Assumptions
The Roulette Result Box has the following register data:
- R4: Roulette Sub-game Selection (0-5) 
- R5: User's Guess for a particular sub-game
- R6: User's receipient ErgoTree

## Reading User Selection (R5) for each sub-game
1. Odd/ Even:
R5 should be set to:
- 0 for even
- 1 for odd
2. Lower Half/ Upper Half:
R5 should be set to:
- 10 if the user is selecting 1-18
- 28 if the user is selecting 19-36
3. Columns:
R5 should be set to:
- 1 for the first column
- 2 for the second column
- 3 for the third column
4. Lower third/ Mid third/ Upper third:
R5 should be set to:
- 6 if the user is selecting 1-12
- 18 if the user is selecting 13-24
- 30 if the user is selecting 25-36
5. Exact Number
R5 should be set to:
- The number the user selects (0-36)

## Box Schema for spending
Input Boxes:
- Self (Result Contract)

Output Boxes:
- User or House Contract Box
  - Value and assets of input box (result contract)


```scala
{ // ROULETTE RESULT CONTRACT
// Get oracle box
val box = CONTEXT.dataInputs(0)

// Find element in oracle report that contains self from mempool
val elementContainingBox = box.R5[Coll[Coll[Coll[Byte]]]].get.filter{
(innerBox : Coll[Coll[Byte]]) => innerBox.exists{(bytes : Coll[Byte]) => (bytes == SELF.id)}
}
// Get index of found element
val elementIndex = box.R5[Coll[Coll[Coll[Byte]]]].get.indexOf(elementContainingBox(0),0)
// Use Eth Hash as entropy
val entropy =  byteArrayToLong(box.R4[Coll[Coll[Byte]]].get(elementIndex + 1))
val rouletteNumber = entropy % 37

val userPaymentAddress = SELF.R6[Coll[Byte]].get
val houseContract = fromBase58("5EvG1rG8DgLajfZf1TGyCeLbwDa9H1sgEB9xDjBdoxKk")
val tokensValid = allOf(Coll(
OUTPUTS(0).tokens(0)._1 == fromBase58("CqK3dmwgkK83qVnHrc8YLpm46t5aDLWNViwrhmtLqPeh"),
OUTPUTS(0).tokens(0)._2 == SELF.tokens(0)._2))
val selectedGame = SELF.R4[Int].get
val userGuess = SELF.R5[Int].get
val range = userGuess - rouletteNumber // Used for section sub-games


val userWins = 
// Even Odd // User Input: Even is 0 Odd is 1
anyOf(Coll((selectedGame == 1 && rouletteNumber % 2 == userGuess),

// Lower Upper // User Input: Lower is 10 Upper is 28
(selectedGame == 2 && range >= -8 && range <= 9),

// Columns // User Input: Column 0,1,2
(selectedGame == 3 && rouletteNumber % 3 == userGuess), 

// Lower Mid Upper // User Input: Lower is 6 Mid is 18 Upper is 30
(selectedGame == 4 && range >= -5 && range <= 6),

// Pick Exact Number
(selectedGame == 5 && userGuess == rouletteNumber)))

val paymentProp = if (userWins) OUTPUTS(0).propositionBytes == SELF.R6[Coll[Byte]].get
else blake2b256(OUTPUTS(0).propositionBytes) == houseContract

sigmaProp(tokensValid && paymentProp)
}
```
