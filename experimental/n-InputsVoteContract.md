Experimental work to allow for n-inputs. This contract is a work in progress

Validation Rule: Loop-Level too high

```scala
// VOTING CONTRACT

// Box Contents:
// Value : Unrestricted
// Tokens: Index 0 has vote tokens
// Registers: 
// R4 Int - Vote Reference Number
// R5 Coll[Byte] - Asset Destination Ergotree
// R6 Coll[(Long, Coll[Byte])] - Vote owners and their vote share

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

// ################## Combine - BEGIN #################################

// Calculate sum of vote tokens across inputs (works for any input size)
val listOfVoteTokens : Coll[Long] = INPUTS.map {(box: Box) => box.tokens(0)._2}
val sumOfVotes = listOfVoteTokens.fold(0L, {(z: Long, base: Long) => z + base})

// Check all voters are voting for same reference and destination
// And check all Inputs have the promised number of tokens
val voteRequirements = INPUTS.forall {(box: Box) => 
// Count number of tokens promised
val listOfPromised = box.R6[Coll[(Long, Coll[Byte])]].get.map {(share: (Long, Coll[Byte])) => share(0)}
val sumOfPromised = listOfPromised.fold(0L, {(z: Long, base: Long) => z + base})
allOf(Coll(
// Check number of tokens matches promise
box.tokens(0)._2 == sumOfPromised,
box.tokens(0)._1 == voteTokenId,
// Check reference and destination
box.R4[Int].get == OUTPUTS(0).R4[Int].get,
box.R5[Coll[Byte]].get == OUTPUTS(0).R5[Coll[Byte]].get)) }

// Get desired output owner list (only works for 2 inputs)
val ownerList = INPUTS(0).R6[Coll[(Long, Coll[Byte])]].get.append(INPUTS(1).R6[Coll[(Long, Coll[Byte])]].get)
val correctOwnerList = OUTPUTS(0).R6[Coll[(Long, Coll[Byte])]].get == ownerList

// Check all inputs carry the tokens suggested in R6


val combineConditions = allOf(Coll(
OUTPUTS(0).tokens(0)._2 == sumOfVotes,
voteRequirements,
correctOwnerList))

combineConditions
}
```
