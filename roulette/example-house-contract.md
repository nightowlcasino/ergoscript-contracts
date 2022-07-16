This contract is used to test the roulettte game and does not reflect the final state of the house contract

```scala
{
val gameNFT = fromBase58("PNvnzqNEmBWhHoxdHGH4aJL5a8HpVM88PTHjE3b5ueM")
val casinoBet = INPUTS(0).tokens(0)._1 == gameNFT
casinoBet
}
