```scala
// VOTING CONTRACT

// Box Contents:
// Value : Unrestricted
// Tokens: Index 0 has vote tokens
// Registers: 
// R4 Int - Vote Reference Number
// R5 Coll[(Coll[Byte], Int)] - Vote owners and their vote share

// Spending Paths:
// Refund (Send all votes to owners)
// Combine (Combine new votes from new owners)

{

// ################## Constant Values #################################

val voteTokenId = fromBase58("")
val minValue = 100000

// ################## Spending Path - Combine #########################

// Inputs
/* All inputs assumed to have R4 (Int) and R5 (Coll[(Coll[Byte], Int)]) 
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

// Get desired output owner list (only works for 2 inputs)
val ownerList = INPUTS(0).R5[Coll[(Coll[Byte], Int)]].get.append(INPUTS(1).R5[Coll[(Coll[Byte], Int)]].get)

allOf(Coll(
OUTPUTS(0).tokens(0)._2 == sumOfVotes,
OUTPUTS(0).R5[Coll[(Coll[Byte], Int)]].get = ownerList))
}

```
