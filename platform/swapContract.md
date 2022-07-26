# Game Witness Contract

* Author: Krasavice Blasen
* Created: 25-Jul-2022
* Status: Testing

## Description

## Contract Requirements

## Contract

```scala 
{
	val heldERG0 = SELF.value
	val heldX0   = SELF.tokens(0)
	val heldY0   = SELF.tokens(1)
	
	val successor = OUTPUTS(0)
	
	val heldERG1 = successor.value
	val heldX1   = successor.tokens(0)
	val heldY1   = successor.tokens(1)
	
	val deltaERG = heldERG1 - heldERG0
	val deltaX   = heldX1._2 - heldX0._2
	val deltaY   = heldY1._2 - heldY0._2
	
	val validSuccessorScript = successor.propositionBytes == SELF.propositionBytes
	
	val validDeltaERG = deltaERG >= 0
	
	val preservedTokens = heldX1._1 == heldX0._1 && heldY1._1 == heldY0._1
	val validTokenSize  = successor.tokens.size == 2 // for sanity
	
	val validSwapAmount = deltaX * - 1 == deltaY // change in token X should be the negation of change in y
	
	sigmaProp(
		validSuccessorScript &&
		validDeltaERG &&
		preservedTokens &&
		validTokenSize &&
		validSwapAmount
	)
}
```
