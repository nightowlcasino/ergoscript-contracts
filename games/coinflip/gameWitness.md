``` scala
{
	// Constants
	val BetResultScript     = fromBase58("") 
	val OwlId               = fromBase58("")
	val HouseContractScript = fromBase58("") 
	val BetMultipliers      = Coll(2) // Represents multiplier for each sub-game
    val MaxPayoutMultiplier = 67
	
	// All extensions of the game witness contract must include validGameWitnesss
	val validGameWitnesss = 
	allOf(
	  Coll(
			OUTPUTS(0).propositionBytes == SELF.propositionBytes, 
			OUTPUTS(0).tokens(0)        == SELF.tokens(0), 
			OUTPUTS(0).value            == SELF.value,
			OUTPUTS(0).tokens.size      == 1
		)
	)  
	
	val housePredecessor = INPUTS(1)
	val houseSuccessor   = OUTPUTS(1)
	
	val owl0 = housePredecessor.tokens(1)._2
	val owl1 = houseSuccessor.tokens(1)._2
	
	val deltaOwl = owl0 - owl1
	
	val validHouseScript     = blake2b256(housePredecessor.propositionBytes) == HouseContractScript
	val validSuccessorScript = houseSuccessor.propositionBytes == housePredecessor.propositionBytes 
	val unchangedValue       = houseSuccessor.value == housePredecessor.value
	val unchangedLP          = houseSuccessor.tokens(0) == housePredecessor.tokens(0)
	val validOwlId           = houseSuccessor.tokens(1)._1 == housePredecessor.tokens(1)._1
	
	val betOuputsSize  = OUTPUTS.size - 2 // Assumes 1 change and 1 fee box
	val outputsToScan  = OUTPUTS.slice(2,betOuputsSize)

	// Get total that house must match
	// Put the match amount from each output into a collection
	val tokenAmountList :Coll[Long] = outputsToScan.map{
	  (box: Box) => (BetMultipliers(box.R4[Int].get) - 1) * box.tokens(0)._2 / BetMultipliers(box.R4[Int].get)
	}
	
	// Sum total of output token amounts
	val houseMatch    = tokenAmountList.fold(0L, {(z: Long, base:Long) => z + base})
	val validDeltaOwl = deltaOwl == houseMatch
	
	// Check all relevant outputs are to the result contract 
	val validOutputsList: Coll[Boolean] = outputsToScan.map{
		(box: Box) => allOf(Coll(
		box.tokens(0)._1 == OwlId,
		blake2b256(box.propositionBytes) == BetResultScript))
	}

	val allOutputsValid = allOf(validOutputsList)
	
	val underMaxPayout = deltaOwl * MaxPayoutMultiplier <= owl0

	sigmaProp(
		validHouseScript &&
		validSuccessorScript &&
		unchangedLP &&
		validOwlId &&
		validDeltaOwl &&
		unchangedValue &&
		allOutputsValid &&
		validGameWitnesss &&
		underMaxPayout
	)
}
```
