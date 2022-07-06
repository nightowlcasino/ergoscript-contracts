# Vote Contract

* Status: Draft
* Created: 06-Jul-2022

## Description 
This contract can be used to change the p2s address of some script through some defined voting system. 
This contract holds vote tokens that have some designated governance value.
This contract allows for voters to use their vote tokens toward some specific vote through a reference number, where the voter can vote for an ammended p2s address.
This contract allows for voters to combine their votes under one UTXO, as well as allowing voters to withdraw their votes through the maintenance of a collection of tuples in R6. 



```scala
// VOTING CONTRACT

// Box Contents:
// Value : Unrestricted
// Tokens: Index 0 has vote tokens
// Registers: 
// R4 Int - Vote Reference Number
// R5 Coll[Byte] - Asset Destination Ergotree
// R6 Coll[(Long, GroupElement)] - Vote owners PK and their vote share

// Spending Paths:
// Refund (Send all votes to owners)
// Combine (Combine new votes from new owners)

{

// ################## Constant Values #################################

val voteTokenId = fromBase58("BTavg5arCrFyhRjEpn15aiYH9dLVLx2wGUKsJYuQi6XT")
val minValue = 100000

// ################## Spending Path - Combine #########################

// Inputs
/* All inputs assumed to have R4, R5 and R6
   defined with vote tokens in tokens(0) (all inputs should have
   self.propBytes, but there is no need to check this) */

// Outputs
/* Output(0) - Combination box, with min value and all tokens combined. 
   Reference number must be the same and owners list should be input(0) 
   zipped over all other inputs */
// Output(1) - Mining box, with no tokens and min value 

// Remarks 
/* If there is not enough erg to form the outputs, voters should refund 
   and use higher erg value */
/* This version of the contract only allows for 2 inputs, combining more
   than 2 inputs is not as trivial and is an area of further research */
/* Voters cannot add votes under the same vote address, instead they can
   refund and delegate more tokens or use a different address they control*/

// ################## Combine - BEGIN #################################

// Calculate sum of vote tokens across inputs (works for any input size)
val listOfVoteTokens : Coll[Long] = INPUTS.map {(box: Box) => box.tokens(0)._2}
val sumOfVotes = listOfVoteTokens.fold(0L, {(z: Long, base: Long) => z + base})


// Count number of tokens promised in input 0
val listOfPromised = INPUTS(0).R6[Coll[(Long, GroupElement)]].get.map {(share: (Long, GroupElement)) => share(0)}
val sumOfPromised = listOfPromised.fold(0L, {(z: Long, base: Long) => z + base})

// Count number of tokens promised in input 1
val listOfPromised1 = INPUTS(1).R6[Coll[(Long, GroupElement)]].get.map {(share: (Long, GroupElement)) => share(0)}
val sumOfPromised1 = listOfPromised1.fold(0L, {(z: Long, base: Long) => z + base})

val correctTokenAmount = allOf(Coll(
// Check number of tokens matches promise
INPUTS(0).tokens(0)._2 == sumOfPromised,
INPUTS(0).tokens(0)._1 == voteTokenId,
INPUTS(1).tokens(0)._2 == sumOfPromised1,
INPUTS(1).tokens(0)._1 == voteTokenId))

// Check all voters are voting for same reference and destination
val referenceAndDestination = INPUTS.forall {(box: Box) => 
// Check reference and destination
box.R4[Int].get == OUTPUTS(0).R4[Int].get &&
box.R5[Coll[Byte]].get == OUTPUTS(0).R5[Coll[Byte]].get }

// Get desired output owner list (only works for 2 inputs)
val ownerList = INPUTS(0).R6[Coll[(Long, GroupElement)]].get.append(INPUTS(1).R6[Coll[(Long, GroupElement)]].get)
val correctOwnerList = OUTPUTS(0).R6[Coll[(Long, GroupElement)]].get == ownerList

val combineConditions = allOf(Coll(
OUTPUTS(0).tokens(0)._2 == sumOfVotes,
OUTPUTS(0).tokens(0)._1 == voteTokenId,
OUTPUTS(0).value >=  minValue,
OUTPUTS(0).propositionBytes == SELF.propositionBytes,
OUTPUTS(1).value == minValue,
OUTPUTS(1).tokens.size == 0,
referenceAndDestination,
correctTokenAmount,
correctOwnerList))

// ################## Spending Path - Refund #########################

// Inputs
/* Self as Input 0 and any other inputs needed to fund mining fee */

// Outputs
/* Output(0) - Self, with one voter's tokens removed and their share
   removed from R6.  */
/* Output(1) - Voter's refunded tokens. All tokens from voter are refunded. 
   Voter public key written in R4[GroupElement] and share in R5[Long] */
// Output(2) - Mining box, with no tokens and min value 

// Remarks 
/* If there is only one voter in share list, Output(0) still exists with
   an empty R6 and min Value */

// ################## Refund - BEGIN #################################

// Get voter and shares
val voterPk = OUTPUTS(1).R4[GroupElement].get
val voterShare = OUTPUTS(1).R5[Long].get

// Get desired result owner list
val starterOwnerList = SELF.R6[Coll[(Long, GroupElement)]].get
val resultOwnerList = starterOwnerList.filter {
(share: (Long, GroupElement)) => 
share(0) != voterShare || share(1) != voterPk}

// Check output 0 valid
val correctOutput0 = allOf(Coll(
OUTPUTS(0).tokens(0)._2 == SELF.tokens(0)._2 - voterShare,
OUTPUTS(0).tokens(0)._1 == voteTokenId,
OUTPUTS(0).value >=  minValue,
OUTPUTS(0).propositionBytes == SELF.propositionBytes,
SELF.R4[Int].get == OUTPUTS(0).R4[Int].get,
SELF.R5[Coll[Byte]].get == OUTPUTS(0).R5[Coll[Byte]].get,
resultOwnerList  == OUTPUTS(0).R6[Coll[(Long, GroupElement)]].get,
// Some additional safety conditions
SELF.tokens(0)._2 == 2))



if (OUTPUTS(0).tokens.size < 2) {
sigmaProp(combineConditions)
} else {
sigmaProp(correctOutput0 && proveDlog(voterPk))
} 


}
```
