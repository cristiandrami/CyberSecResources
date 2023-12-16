A mechanism design is the inverse of game theory.

In game theory we want to predict what happens given a game.

In mechanism design we want to create games that incentivise a specific behavior.

Some desired behaviours could be:
- game is strategy proof 
- individual rationality
- efficiency 
- fairness


## Mechanism design types
Without money -> matching problems and facility location
With money -> auctions


## Mechanism design with money
## Auctions
We are selling a given item x
There are N different players that want to buy it

Each player has his personal valuation of the item -> vi (x) for player i

Each player pays a price -> pi(x)
- if the player i doesn't get the item because another one gets it with his offer then pi(x) = 0

Each player has a quasi-linear utilities -> ui(x) = vi(x) - pi(x)



### First price Auctions
The player who gives the highest offer gets the item and pays a price equal to his offer.

![[Pasted image 20231214184923.png]]
- the winner has utility equal to 0 if the valutation of the player is equal to the offer he gives
- so this incentives to lie and report a lower offer in order to increase the utility



### Second price Auctions
The player who gives the highest offer gets the item and pays a price equal to the second higher offer.
![[Pasted image 20231214185403.png]]

So in this case players are incentiveted to give an offer that is the real evaluation.




## VCG Auctions
With auctions we have two main problems that are:
- determining the winner 
- make the winner pay an appropriate price

If we can make the winner to pay an appropriate price then is convenient for him to say the truth.

VCG auctions are one of these mechanisms.


### Combianational Auctions
Here we have:
- m items
- n players
Each player values combinations of items 

![[Pasted image 20231214190230.png]]

The allocation of the items is done maximizing social welfare.


B gets a
C gets b 

The total welfare is 5+2 = 7


## How we choose which price must B and C pay?
per ogni giocatore prima tolgo il giocatore dal gioco e calcolo il massimo social welfare possibile con gli altri giocatori

pA -> tolgo A e calcolo le nuove allocazioni e prendo il valore e gli sottraggo la somma dei valori della allocazione reale senza A 

tipo pA lo tolgo e le nuove allocazioni sono sempre a B b C 
quindi nuovo valore = 7 ora vado nei vecchi valori e tolgo il valore di A ma non c'era quindi rimane 7
pA = 7  - 7 = 0 


pC -> tolgo C e le nuove allocazioni diventano a->B b->A con valore 5+1=6
dal valore iniziale del welfare tolgo C e quindi 7-2 = 5 quindi
pC=6-5 = 1

![[Pasted image 20231214193438.png]]


## VCG Mechanism formalization
![[Pasted image 20231214195141.png]]![[Pasted image 20231214195149.png]]