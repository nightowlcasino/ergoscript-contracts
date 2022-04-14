# Game Witness Contract for all roulette sub-games

* Author: Krasavice Blasen
* Created: 01-Apr-2022

## Description
This contract stores the Night Owl game token, a token which allows the spending of house-contract funds. 
The contract only allows spending of house-contract funds to match a user bet, where this matched bet alligns with the rules of a roulette sub-game.
The spending condition of the contract is as follows:
- Game Witness Box and House-contract Box are used as two inputs. 
- Game Witness is an output with unchanged value and assets (hence is a witness)
- House-Contract Box is an output with unchanged value, however it has a reduced number of OWLs, with this reduction being at most a matching bet. 
- A matching bet is defined from the amount of funds direceted to the rouletteResult contract by the rules of the roulette sub-game being played.

## Game Rules
The user will indicate the sub-game they wish to play in R4.
The subgames for a wager of x OWLs are:

0. Red/Black (House match bet = x OWLs)
1. Odd/ Even (House match bet = x OWLs)
2. Lower Half/ Upper Half (House match bet = x OWLs)
3. Columns (House match bet = 2x OWLs)
4. Lower third/ Mid third/ Upper third (House match bet = 2x OWLs)
5. Exact number (House match bet = 35x OWLs)

## Box Schema for gameplay

Input Boxes:
- Self (Game Witness Box)
- House Contract 
- User Boxes (Select unspent UTXOs with x OWLs, minBoxValue + txFee)

Output Boxes:
- Self (Game Witness Box):
  - Value and assets unchanged (set to self)
- House Contract:
  - Value Unchanged, 
  - token(0) LP token unchanged,
  - tokens(1) reduced by game multiplier
- Result Contract:
  -  Value minBoxValue, 
  -  Assets(User Wager + Matched Bet) OWLS, 
  -   R4 encoded int from 0-5, 
  -  R5: User's bet (See result contract), 
  -  R6: Prize winning receipient address (encoded ergotree)



## The contract
```scala


{
/* //// Register Details ////
OUTPUTS(idx).R4(Int) : Indication of roulette sub-game the user wishes to play
OUTPUTS(idx).R5(Int) : User selection
OUTPUTS(idx).R6(Coll[Bytes]) : Winner address */

// Constants //
val betContract = fromBase58("Y3xGKyAbCTa3DnrhYdRdLvADQe1sgUupJsaqrekHuh4") // Result Contract
val owlId = fromBase58("CqK3dmwgkK83qVnHrc8YLpm46t5aDLWNViwrhmtLqPeh")
val houseContract = fromBase58("5EvG1rG8DgLajfZf1TGyCeLbwDa9H1sgEB9xDjBdoxKk") // House contract ErgoTree

// Get Total bet Multiplier //
// Get all OUTPUTS from 2 to OUTPUT before list fee
val collectionSize = OUTPUTS.size - 1
val outputsToScan = OUTPUTS.indices.slice(2,collectionSize)
// Put all r4 from outputs list in a collection
val r4Ints :Coll[Int] = outputsToScan.map{
  (idx: Int) => OUTPUTS(idx).R4[Int].get}
// Sum Ints in r4 List
val betMultipliers = Coll(2,2,2,3,3,36)
val r4Sum = 
      r4Ints.fold(0, {(z: Int, base:Int) => z + betMultipliers(base)})
val betMultiplier = r4Sum
val betMatcher = betMultiplier - outputsToScan.size // Represents the share which the house matches

// Get total wager//
// Put all token amounts from outputs list in a collection
val tokenAmountList :Coll[Long] = outputsToScan.map{
  (idx: Int) => OUTPUTS(idx).tokens(0)._2
}
// Sum total tokens (wager)
val wager = 
      tokenAmountList.fold(0L, {(z: Long, base:Long) => z + base})

// Check OUTPUTS to result contract valid //
val validOutputsList: Coll[Boolean] = outputsToScan.map{
  (idx: Int) => allOf(Coll(
OUTPUTS(idx).tokens(0)._1 == owlId,
blake2b256(OUTPUTS(idx).propositionBytes) == betContract,
OUTPUTS(idx).tokens(0)._2 == wager * betMultipliers(OUTPUTS(idx).R4[Int].get)/ r4Sum))
}
val allOutputsValid = allOf(validOutputsList)

allOf(Coll(
INPUTS(0).propositionBytes == SELF.propositionBytes, // Game Guard Witness
blake2b256(INPUTS(1).propositionBytes) == houseContract, // House Matching Bet

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
OUTPUTS(1).propositionBytes == INPUTS(1).propositionBytes, 
OUTPUTS(1).tokens(1)._1 == INPUTS(1).tokens(1)._1,
OUTPUTS(1).tokens(1)._2 == INPUTS(1).tokens(1)._2 - (betMatcher * wager / betMultiplier), // Decrease OWLS by the amount due from the house
OUTPUTS(1).tokens(0)._1 == INPUTS(1).tokens(0)._1, 
OUTPUTS(1).tokens(0)._2 == INPUTS(1).tokens(0)._2,
OUTPUTS(1).value == INPUTS(1).value,
// Bet Contract
allOutputsValid)) 
 
}
```
