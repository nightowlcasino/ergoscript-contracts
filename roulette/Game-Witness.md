# Game Token Witness Contract for all roulette sub-games

* Author: Krasavice Blasen
* Created: 01-Apr-2022

## Description
This contract stores the Night Owl game token, a token which allows the spending of house-contract funds. 
The contract only allows spending of house-contract funds to match a user bet, where this matched bet alligns with the rules of a roulette sub-game.
The spending condition of the contract is as follows:
- Game Token Witness Box and House-contract Box are used as two inputs. 
- Game Token Witness is an output with unchanged value and assets (hence is a witness)
- House-Contract Box is an output with unchanged value, however it has a reduced number of OWLs, with this reduction being at most a matching bet. 
- A matching bet is defined from the amount of funds direceted to the roulette red black green contract by the rules of the red black roulette game

## Game Rules
The user will indicate the sub-game they wish to pay in R4.
The subgames for a wager of x OWLs are:

0. Red/Black (House match bet = x OWLs)
1. Odd/ Even (House match bet = x OWLs)
2. Lower Half/ Upper Half (House match bet = x OWLs)
3. Columns (House match bet = 2x OWLs)
4. Lower third/ Mid third/ Upper third (House match bet = 2x OWLs)
5. Exact number (House match bet = 35x OWLs)

## The contract
```scala
{

/* //// Register Details ////
OUTPUTS(2).R4(Int) : Indication of roulette sub-game the user wishes to play
OUTPUTS(2).R5(Int) : User selection
OUTPUTS(2).R6(Coll[Bytes]) : Winner address */


val betContract = fromBase58("7EVdepYu8UFFW6HocYPQ1oCno2U7GGxCnL1hPUPZq4k2") // Result Contract
val owlId = fromBase58("CqK3dmwgkK83qVnHrc8YLpm46t5aDLWNViwrhmtLqPeh")
val houseContract = fromBase58("5EvG1rG8DgLajfZf1TGyCeLbwDa9H1sgEB9xDjBdoxKk") // House contract ErgoTree
val betMultipliers = Coll(2,2,2,3,3,36)
val betMultiplier = betMultipliers(OUTPUTS(2).R4[Int].get)
val betMatcher = betMultiplier - 1 // Represents the share which the house matches

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
OUTPUTS(2).tokens(0)._1 == owlId)) 
} 
```
