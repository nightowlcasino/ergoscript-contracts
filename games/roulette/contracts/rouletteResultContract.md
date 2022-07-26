# Roulette Result Contract

* Author: Krasavice Blasen
* Created: 01-Apr-2022
* Status: Testing

## Description
This contract guards a _RouletteResultBox_ which is a box containing some amount of OWL tokens and some register details which indicate a user's selection in the game _Roulette_. The contract calls for a _dataInput_ which provides historical ETH hashes and ERG boxIds to the contract to produce a random number for the roulette game. Depending on the produced random number, either the house will be able to receive the OWLs within the box or the user will. 

## Contract Requirements
- Read a dataInput from an _oracle_ and authenticate the oracle by checking the _OraclePoolNFT_
- Check the given coordinates for the RouletteResultBox id match the actual RouletteResultBox id.
- Get the roulette number by converting the second eth hash after the given coordinates to a long mod 37
- Check whether the user won by evaluating their game selection and guess as per the _RouletteGameSchema_
- If the user has won, ensure the winner box is under the provided user address, else ensure the winner box is under the house funds address.

```scala
{
	val HouseContract = fromBase58("")
	val OwlId         = fromBase58("")
	val OracleNFT     = fromBase58("")

	val userGame    = SELF.R4[Int].get
	val userGuess   = SELF.R5[Int].get
	val userAddress = SELF.R6[Coll[Byte]].get

	val heldTokens = SELF.tokens(0)

	val winner = OUTPUTS(0)
	val oracle = CONTEXT.dataInputs(0)

	val ethIndex = winner.R4[Int].get
	val ergIndex = winner.R5[Int].get

	val validOracle      = oracle.tokens(0)._1 == OracleNFT
	val oracleEthPayload = oracle.R4[Coll[Coll[Byte]]].get
	val oracleErgPayload = oracle.R5[Coll[Coll[Coll[Byte]]]].get

	val winnerTokens = winner.tokens(0)

	val entropy        =  byteArrayToLong(oracleEthPayload(ethIndex + 2))
	val rouletteNumber = entropy % 37

	val validTokens = winnerTokens == heldTokens

	val validEthHash = oracleErgPayload(ethIndex)(ergIndex) == SELF.id

	val range = userGuess - rouletteNumber // Used for section sub-games

	// red/black bet logic
	val winRedBlack = if (rouletteNumber <= 10 || (rouletteNumber >= 19 && rouletteNumber <= 28)) {
		rouletteNumber % 2 == userGuess
	} else {
		(rouletteNumber % 2 != userGuess) &&
		(userGuess == 1 || userGuess == 0)
	}

	val userWins = 
	anyOf(
		Coll(
			// Red Black // User Input: Black is 0, Red is 1
			(userGame == 0 && winRedBlack),

			// Even Odd // User Input: Even is 0 Odd is 1
			(userGame == 1 && rouletteNumber % 2 == userGuess),

			// Lower Upper // User Input: Lower is 10 Upper is 28
			(userGame == 2 && range >= -8 && range <= 9),

			// Columns // User Input: Column 0,1,2
			(userGame == 3 && rouletteNumber % 3 == userGuess), 

			// Lower Mid Upper // User Input: Lower is 6 Mid is 18 Upper is 30
			(userGame == 4 && range >= -5 && range <= 6),

			// Pick Exact Number
			(userGame == 5 && userGuess == rouletteNumber)
		)
	)

	val paymentProp = 
	if (userWins) {
		winner.propositionBytes == userAddress
	} else {
		blake2b256(winner.propositionBytes) == HouseContract
	}

	sigmaProp(
		paymentProp &&
		validEthHash &&
		validOracle &&
		validTokens
	)
}
```
