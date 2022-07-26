```scala
{
	val InitiallyLockedLP   = 9000000000000002L
	val gameWitnessToken    = fromBase58("")
	val sigUSDTokenId       = fromBase58("")
	val owlTokenId          = fromBase58("")

    val MinValue            = 1000000 // this many number of nanoErgs are going to be permanently locked

    val poolNFT0 = SELF.tokens(0)
    val heldLP0  = SELF.tokens(1)
    val heldOwl0 = SELF.tokens(2)
 
    val successor = OUTPUTS(0)
    
    val poolNFT1 = successor.tokens(0)
    val heldLP1  = successor.tokens(1)
    val heldOwl1 = successor.tokens(2)
	
	val supplyLP0 = InitiallyLockedLP - heldLP0._2
    val supplyLP1 = InitiallyLockedLP - heldLP1._2
	
    val validSuccessorScript = successor.propositionBytes == SELF.propositionBytes

    val preservedPoolNFT     = poolNFT1 == poolNFT0
    val validLP              = heldLP1._1 == heldLP0._1
	val validOwl             = heldOwl1._1 == heldOwl0._1
    // since tokens can be repeated, we ensure for sanity that there are no more tokens
    val noMoreTokens         = successor.tokens.size == 3
    val validMinValue        = successor.value >= MinValue

    val deltaLP  = heldLP1._2 - heldLP0._2
	val deltaOwl = heldOwl1._2 - heldOwl0._2
	
    val lpValue0 = supplyLP0 / heldOwl0._2
    val lpValue1 = supplyLP1 / heldOwl1._2

    val validLPValue = lpValue0 >= lpValue1 // Ensures the current value of an LP token has not decreased
   
    val validExchange = (
        validSuccessorScript &&
        preservedPoolNFT &&
        validLP &&
        validLPValue &&
        noMoreTokens &&
        validMinValue
    )
    
    // Also allow for deposits
    
    val validDeltaOwl = deltaOwl > 0
    val validDeltaLp  = deltaLP >= 0
    
    val validDeposit = (
        validSuccessorScript &&
        preservedPoolNFT &&
        validDeltaLp &&
        noMoreTokens &&
        validDeltaOwl &&
		validMinValue
    )
	
	// Allow for bets
	
	val gameWitness = INPUTS(0)
	
	val validWitness = gameWitness.tokens(0)._1 == gameWitnessToken

	val validBet = (
		validWitness
	) 
		
    sigmaProp(validDeposit || validExchange || validBet) 
}


```
