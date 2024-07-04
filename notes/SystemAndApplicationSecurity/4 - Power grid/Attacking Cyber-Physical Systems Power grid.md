
# Power grid (rete elettrica)
## Continuous Balance of Supply and Demand
When we talk about power grid we have to consider that one of the most important thing is: how is managed the balancing between supply and demand of energy.


**==The electric power can be stored in capacitors, batteries and in reservoir power stations. But it cannot be stored in a large scale.==**

So demand and supply must be kept in balance.

## From local to continental grid

Initially electric supply started in a decentralized fashion with a single generator and each generator was single point of failures.

Local networks were later merged to a national and continental grid.

![[Pasted image 20240614154437.png]]



## Keeping balanced

One important thing is that **==costumers are uncontrolled and so they can use the electricity whenever they want==**:
- *example we can plug the charger of the pc and so on*

Only large costumers such as steel mills have to report their plan of power consumption in advance.



**==The electricity operators need to control their power plants to adjust supply in accordance with the demand.==**

So for these reason load profiles are used to predict power demand:
- ![[Pasted image 20240614154713.png]]

And for example this for mobile band:
- ![[Pasted image 20240614155009.png]]
	- always high


This for street lights:
- ![[Pasted image 20240614155039.png]]



If something is imbalanced then there are numerous power plants that are in stand-by mode.
1. **==Primary reserve==** -> very fast, just some seconds to respond
2. **==Secondary reserve==** -> in some minutes
3. **==Tertiary reserve==** -> slow, more than 15 minutes and are manually scheduled


## Measuring Imbalance

The system frequency is continuosly measured to determine power imbalance.
- ![[Pasted image 20240614155352.png]]


For a system frequency the **==nominal frequency is 50 Hz==**.
- an <mark style="background: #FF5582A6;">higher frequency is caused by supply exceeding</mark>
- a <mark style="background: #FF5582A6;">lower frequency is caused by demand exceeding </mark>

In general it is typically around the nominal value:
- ![[Pasted image 20240614160011.png]]


## Reason for Frequency Deviation 
The reason is the Newton's second law (Rotation force):
- ![[Pasted image 20240614160146.png]]

The **==imbalance in demand and supply accelerate or decelerate the power plants' turbines==**



## Emergency routines

When a stand-by electric central cannot face the frequency deviation then we use:
- **==load shedding==** -> reducing of the work
- **==automated disconnection of wind generators==**
- **==disconnection of all electric centrals to avoid problems==** 


The action are:
- ![[Pasted image 20240614160948.png]]



# Attacking

## General attack strategy 

The idea to perform an attack here is:
1. build or rent a botnet
2. change the power consumption of the botnet in a coordinated fashion
3. destabilization of the power grid


### Attack 1 - Static load attack
The idea is to **==overload the electric system increasing the electric load in a very restricted amount of time==**.
- ![[Pasted image 20240614161351.png]]
	- upper row -> how conventional electric centrals respond to static load attack
	- lower row -> how renewable power plants respond to it

### Attack 2 - Dynamic load attack
The idea here is to perform **==anomal variation of electric load in a period of time and so increasing and decreasing the load==**:
- ![[Pasted image 20240614161637.png]]

### Attack 3 - Interzone attack
The idea **==is to act on different zones (like regions) of the power grid in a syncronized and coordinated way==**:
- ![[Pasted image 20240614162013.png]]


## Bots
Another used typology of attack is the botnet attack in which the attacker takes the control of **==millions of bots in order to change in a syncronous way their power consumption.==**

Where the bots can be:
- ![[Pasted image 20240614162144.png]]
And  the botnet:
- ![[Pasted image 20240614162156.png]]
- ![[Pasted image 20240614162207.png]]


**==However to be able to attack a power grid what we need is to control around 10 millions of bots:==**
- ![[Pasted image 20240614162254.png]]



# Bitcoins

In the era of bitcoins we have to consider that the energy efficiency becames more and more lower:
- ![[Pasted image 20240614162427.png]]

And the consumption is:
- ![[Pasted image 20240614162454.png]]

In Europe:
- ![[Pasted image 20240614162516.png]]



# Aftermath

## Blackout
A blackout happens when an entire powergrid loses the possibility to serve energy to a entire region.
When we have an entire city shut down that it is a black out, not when a single district is without energy.


## Dispatching
**==In general operators have to reschedule their power plants to dispatch the energy.==**

The reserve capacities are more responsive type of power plants and are generally constructed by gas turbines.

## Resyncronization
The idea is to separate synchronous grid into separate desynchronized areas.

**==The resynchronization or reconnection requires a lot of effort.==**

## Desynchronization (2006)

In 2006 a line of 380 kV was disconnected and the power was deviated to other lines.

One of these lines was interruped (since the current was too high).

This was the situation:
- ![[Pasted image 20240614163656.png]]
- ![[Pasted image 20240614163713.png]]

However then we had a Resynchronization:
- ![[Pasted image 20240614163756.png]]


## Black start
**==After a black out we need to restart the power plants and this is called black start.==**

**==A power plant (centrale) need a working power grid (rete) to restart their activities.==**

<mark style="background: #BBFABBA6;">The power plants that are able to easily restart their self after a black out (black start) must restart in a fast way to allow other centrals to connect to them to restart.</mark>


### Northeast Blackout (2003)
It afflicted 55 million of people in US and Canada and full restoration needed up to two weeks.

(No experience on how to bring up the power grid)

## Renewables
The usage of renewables can lead to higher frequency deviations:
- because many renewable don't have rotants components and so the system can be afflicted 
