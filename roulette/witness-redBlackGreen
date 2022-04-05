# Game Token Witness Contract for Red Black Green roulette games

* Author: Krasavice Blasen
* Created: 01-Apr-2022

## Description
This contract stores the Night Owl game token, a token which allows the spending of house-contract funds. 
The contract only allows spending of house-contract funds to match a user bet, where this matched bet alligns with the rules of Red Black Green roulette.
The spending condition of the contract is as follows:
- Game Token Witness Box and House-contract Box are used as two inputs. 
- Game Token Witness is an output with unchanged value and assets (hence is a witness)
- House-Contract Box is an output with unchanged value, however it has a reduced number of OWLs, with this reduction being at most a matching bet. 
- A matching bet is defined from the amount of funds direceted to the roulette red black green contract by the rules of the red black roulette game

## Game Rules
If a user selects red or black with a wager of x OWLS, the matching bet will be x OWLs from the house.
If a user selects green, the matching bet will be 35x OWLs from the house. 

## The contract
```scala
// ROULETTE GAME NFT GUARD BOX
{
val betContract = fromBase64("abcdef") // Contract that describes the outcome of the game
val owlId = fromBase58("CqK3dmwgkK83qVnHrc8YLpm46t5aDLWNViwrhmtLqPeh")
val houseContract = fromBase64("abcd") // House contract ErgoTree
val betMultiplier = if(OUTPUTS(2).R4[Long].get == 2) {35} else {2} // Represents payout multiplier (R4 = 2 represents green choice)
val betMatcher = betMultiplier - 1 // Represents the share which the house contract matches

allOf(Coll(

INPUTS(0).propositionBytes == SELF.propositionBytes, // Game Guard Witness
INPUTS(1).propositionBytes == houseContract, // House Matching Bet

// Game Guard has all tokens and value unchanged
// Assumes game guard holds just one token 
// Parallel Game Contracts not considered
OUTPUTS(0).propositionBytes == SELF.propositionBytes, 
OUTPUTS(0).tokens(0)._1 == INPUTS(0).tokens(0)._1, 
OUTPUTS(0).tokens(0)._2 == INPUTS(0).tokens(0)._2,
OUTPUTS(0).value == INPUTS(0).value,

// House Contract has all tokens and value unchanged (minus matching bet)
// Assumes house contract holds just LP and OWL tokens
// Parallel House Contracts not considered
// Requires OUTPUTS(2) to be betContract, with OWLS in token index 0
OUTPUTS(1).propositionBytes == houseContract, 
OUTPUTS(1).tokens(0)._1 == INPUTS(1).tokens(0)._1,
OUTPUTS(1).tokens(0)._2 == INPUTS(1).tokens(0)._2 - (betMatcher * OUTPUTS(2).tokens(0)._2 / betMultiplier), // Decrease OWLS by the amount due from the house
OUTPUTS(1).tokens(1)._1 == INPUTS(1).tokens(1)._1, 
OUTPUTS(1).tokens(1)._2 == INPUTS(1).tokens(1)._2,
OUTPUTS(1).value == INPUTS(1).value,

// Bet Contract
OUTPUTS(2).propositionBytes == betContract,
OUTPUTS(2).tokens(0)._1 == owlId,
OUTPUTS(2).R5[Coll[Byte]].isDefined)) // Simple assertion to help ensure R5 has wager address (could extend with a length check)
} 
```
