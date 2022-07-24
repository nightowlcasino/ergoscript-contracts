# Game Witness Contract

* Author: Krasavice Blasen
* Created: 01-Apr-2022
* Status: Testing

## Description
The game witness contract is platform contract that is used to access house funds and enforce any game logic of any Night Owl game. 
The contract protects a _GameWitnessBox_ which is a box that holds a single _GameWitnessToken_. 

## Contract Requirements:
The only requirement of this contract is that the _GameWitnessToken_ is preserved under the same script. 

The contract can and should be extended to include some game logic but, no matter the extension, the _GameWitnessToken_ must be preserved

```scala 
allOf(
  Coll(

		OUTPUTS(0).propositionBytes == SELF.propositionBytes, 
		OUTPUTS(0).tokens(0)        == SELF.tokens(0), 
		OUTPUTS(0).value            == SELF.value,
		OUTPUTS(0).tokens.size      == 1
		)
)  
```


