# A Glossary of Terms For Night Owl Contracts

## Boxes

| Box         | Description | Script |  Value | Tokens | Registers|
| ----------- | ----------- | ----------- | ----------- | ----------- | ----------- |
| GameWitnessBox | A box that enforces the rules and funding of Night Owl games. | _GameWitnessContract_ extended to include some game logic | _MinBoxValue_ | (1, _GameWitnessToken_) | {}|

## Tokens
| Token       | Description | Emission Amount | Bootstrap Storage | Circulating Storage | 
| ----------- | ----------- | -----------  | -----------  | -----------  |
| GameWitnessToken | An authentication token that can be used to access funds stored under the _HouseBox_ | Unlimited | _WitnessEmissionBox_ | _GameWitnessBox_




## Terms
| Term        | Description |
| ----------- | ----------- |
| MinBoxValue | A general purpose constant with a value of  ``1000000``, despite the name this is not necessarily the minimum number of nanoErgs an Ergo Box must store, instead, this value is a rough approximation of the minimum number of nanoergs required for a box which is usually fine for most purpose.|

