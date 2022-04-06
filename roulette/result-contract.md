```scala
{ // ROULETTE RESULT CONTRACT
val index = HEIGHT - SELF.creationInfo._1 - 2
val randomEntropy = byteArrayToLong(CONTEXT.headers(index).id)
val rouletteNumber = randomEntropy % 37
val userPaymentAddress = SELF.R6[Coll[Byte]].get
val houseContract = fromBase58("5EvG1rG8DgLajfZf1TGyCeLbwDa9H1sgEB9xDjBdoxKk")
val tokensValid = allOf(Coll(
OUTPUTS(0).tokens(0)._1 == fromBase58("CqK3dmwgkK83qVnHrc8YLpm46t5aDLWNViwrhmtLqPeh"),
OUTPUTS(0).tokens(0)._2 == SELF.tokens(0)._2))
val selectedGame = SELF.R4[Int].get
val userGuess = SELF.R5[Int].get
val range = userGuess - rouletteNumber // Used for section sub-games


val userWins = 
// Even Odd // User Input: Even is 0 Red is 1
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
