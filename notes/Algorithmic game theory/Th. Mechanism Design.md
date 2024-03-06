A mechanism design is the inverse of game theory.
In game theory we want to predict what happens given a game.

==In mechanism design we want to create games that incentivise a specific behavior.==



In real life we cannot assume that what agents do is trustable.
==So the idea is to define some rules in order to force the agets to say the truth.==


## Basic concepts 
<mark style="background: #BBFABBA6;">Each agent i is associated with a type Oi, that represents the real and private preferences of the agent.</mark>

Each agent has a strategy that is the action the agent wants to do. (So they are the preferences that the agents says publically)


If we consider the vector of the joint strategies s = (s1, ..., sI). We have that each agent i has a utility that depends on:
- the chosen strategy of the agent i
- the strategies of the other agents
- the private type of the agent (private preferencies)



A nash equilibrium is a strategy profile s = (s1,..., sI) such that for each agent i and for each s'i != si then:
- ui(si, s of other players, Oi) >=  ui(si', s of other players, Oi)
- Non gli conviene deviare


But to play a nash equilibrium each player must have information about other players choices and all players must select the same Nash equilibrium.



## Dominant strategy
==A dominant strategy is a strategy that is better to play in any case.==
<mark style="background: #BBFABBA6;">When we have a dominant strategy then we don't care about the other players choices, because we will alway play this strategy.</mark>


In general there are no Dominant strategies but a Nash equilibrium always exist.



## Mechanism and implementation 
### Social choice function
The idea here is to implement a mechanism such that in all possible nash equilibrium we have the rule we introduce must select the outcome that is chosen by the social choice function.

![[Pasted image 20231214201648.png]]
- when the strategy profile is in nash equilibrium then we want to select the outcome of the social function



==A mechanism is a tuple M = ( S1, ..., SI, g()) where ==
- ==Si is the set of all avajilable strategies of the agent i ==
- g is an outcome rule such that
  - ==given a strategy profile s = (s1,..., sI)==
  - selects an outcome g(s)


M implements in dominant stratey the social choice function f if, 
- for each type vector O = (O1, ..., OI) (tutti le private preferences dei players)
- g selects as output the output of the social function for that vector
- when strategy profile s = (s1,..., sI) is a dominant strategy profile (each strategy is dominant for each player i)



## Direct revelation mechanisms
<mark style="background: #BBFABBA6;">A direct revelation mechanism is a mechanism in which each strategy is restricted to a declaration about the private type.</mark>



A direct revelation mechanism is strategy-proof if truth-revelation is dominant strategy for each agent. 
So if the mechanism implements a function f then f=g

But the problem is how we can implement a function f such that:
- when the truth is said then g=f
- when the truth is not said then there is a penalization

We can use payments


## Payments
To introduce the truthfulness we use the money compensation.

A utility is quasi -linear if it has the following form
- ui = vi - pi
	- vi is the value the agent gives to a preference 
	- pi is the payment of the agent


## VCG mechanisms
==This mechanism selects the outcome O* that maximizes the summatory of the social outcome value==

A payment is of the form:
- <mark style="background: #BBFABBA6;">pi = ( value of the optimal outcome if we remove the player i ) - (current social outcome value - value of the player i )</mark>



### Fairness 
The idea is to use the monetary balancing in order to improve the fairness.
![[Pasted image 20231214205003.png]]



## Approach to verification
In order to verify if a agent is saying the truth we can have two types of verification:
- partial verification
- probabilistic verification 

In general the idea is to use punishments to enforce truthfulness.

Verification is done via sensing and this might be problematic to decide whether an observed discrepancy between verified values and declared ones is due to a strategic behavior or to sensing errors.

So we also have a full verification
- we have no punishment
- we have error tollerance





## EXAMPLE OF VERIFICATION ON ALLOCATION PROBLEM
In this specific case the idea is based on the fact that:
- the verifier knows the private types of the players

==The goal is to ensure the Optimal Allocations and to ensure the truthfulness (each player must say the truth) beacuse each player can provide fake valutations of the goods:==


![[Pasted image 20231219154556.png]]



![[Pasted image 20231219154611.png]]


We can see in this case the second allocation (when the first player doesn't say the truth) is not optimal.



The verification is done only on the final selected items (the items assigned to the players).


There is a useful lemma we can leverge on to create a mechanism:
1. ==consider an optimal allocation for the game (according to the declared preferencies of the players)==
2. ==delete the goods that are not allocated ==
3. ==choose an arbitrary coalition (maybe different from the grand coalition)==
4. ==compute the optimal allocation for this new coalition (only for the goods that were allocated in the initial one)==

The new allocation is optimal for the new coalition even if all goods were available. (L'allocation è ottima anche se tutti gli oggetti fossero disponibili)


The mechanism is based on the shapley value given a player j:
1. calculate the value of the coalition val(C)
2. calculate the value of coalition without j val(C - j)
3. calculate the marginal contribution of j over C
	1. val(C) - val(C - j)

Then we can compute the shapley value for the player j.


The using of shapley value gives us the truthfulness.
So the idea is to create a coalitional game over this game and pay each player according the shapley value.

==So each agent obtains:==
- ==(value of shapley value) - (sum of value of objects assigned to the player)==
- ==if an agent gets 0 objects then his payment is basically his shapley value==
For this reason each one player will say the truth.


Players cannot give a value to an object that is higher than the real one they consider because the verifier performs the validation on the private types (it knows them).