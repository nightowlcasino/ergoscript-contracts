```scala
{
	val desiredToken = fromBase58("9QYqAf7Z3TNPXV4wJmVBaZj3CWsvAPYeguthK4kXvUCh")
	val minTxFee     = 1000000L
	val minBoxValue  = 1000000L
	
	val user          = SELF.R4[Coll[Byte]].get
	val requestAmount = SELF.R5[Long].get
	val publicRefund  = SELF.R6[Long].get
	
	val successor = OUTPUTS(1)
	
	val exchangedTokens = successor.tokens
	
	val validSuccessorScript = successor.propositionBytes == user
	val validTokens = if (exchangedTokens.size != 0){exchangedTokens(0)._1 == desiredToken && exchangedTokens(0)._2 >= requestAmount} else {false} // false branch for refunds with AOT costing consideration
	
	val multiBoxSpendSafety = successor.R7[Coll[Byte]].get == SELF.id
	
	val exchange = (
		validSuccessorScript &&
		validTokens &&
		multiBoxSpendSafety
	)
	
	val deltaErg = SELF.value - successor.value
	
	val validDeltaErg = deltaErg <= minTxFee + minBoxValue
	val validHeight   = HEIGHT >= publicRefund
	
	val refund = (
		validSuccessorScript &&
		validDeltaErg &&
		multiBoxSpendSafety &&
		validHeight
	)
	
	sigmaProp(exchange || refund) 
}
```
	
