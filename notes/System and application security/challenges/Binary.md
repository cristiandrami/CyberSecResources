 ### Cristian Domenico Dramisino (12338532)

## Introduction

For these challenges we will use some interesting tools:
- [Ghidra](https://ghidra-sre.org/)


# Challenge part 1 

## Introduction
We can access to this challenge through this [link](https://sas.hackthe.space/#/challenges/Binary%200)


From the description of the flag we don't get many useful information.

We just understand that:
```
You will need the following information:

- The [binary file](https://sas.hackthe.space/cms/files/47d2dc9ac76f0b965c0617decce9d5e432a6b06b004066c543c261462f54b727/binary0).
- The system running the binary and the flag can be reached via `nc provinggrounds.sas.hackthe.space 31337`
- The flag regex is `FLG_\{[a-z0-9_]+\}`
```
- we can download the bianry file from this [link](https://sas.hackthe.space/cms/files/47d2dc9ac76f0b965c0617decce9d5e432a6b06b004066c543c261462f54b727/binary0)
- we cna reach the server that runs the binary using `nc provinggrounds.sas.hackthe.space 31337`




## Binary dynamic analysis
What we do here is to perform a dynamical analysis of the binary, and so we try to use it in order to understand what are its functionalities.

So we can easily give the execution permissions to it:
```shell
chmod +x binary0
```

At this point we can execute it:
```shell
./binary0
```

We can see that we have a menu here that gives us the possibility to choose what to do:
![[Pasted image 20240524155620.png]]


By now, the most interesting sections here are:
- Administration area
- Check privilege

### Privileged features interaction 

Let's try to interact with them.

If we try to access to `Administration area` we get this message:
![[Pasted image 20240524155725.png]]
- so we need privileges to access to it

If we try to access to `Check privilege` we get this message:
![[Pasted image 20240524155846.png]]


Nothing to see there.

So let's interact with `Manage bugs`


### Manage bugs interaction

When we access to the area`Manage bugs` we can see another internal menu:
![[Pasted image 20240524160056.png]]


At this point we can play with these features and we can notice some interesting stuff:

#### Add bug
`Add bug` gives us the possibility to add a new bug in the list, composed by a description and a security level:
![[Pasted image 20240524160225.png]]

#### Delete bug
`Delete bug` gives us the possibility to remove a bug from the list using his `id`:
![[Pasted image 20240524160324.png]]

#### Change bug
`Change bug` gives us the possibility to change the severity of a bug using his `id`:
![[Pasted image 20240524160444.png]]


But if we play there with the values we can see an interesting thing...
Differently from others input values it seems that the `id` value we insert here <mark style="background: #FF5582A6;">is not checked or validated before it is used.
</mark>

In fact, if we put a value higher than `9` there, <mark style="background: #FF5582A6;">we get a strange result, and so basically it seems that we have access to a memory address that must not be accessible...</mark>

*Examples:*
1. ![[Pasted image 20240524160910.png]]
2. ![[Pasted image 20240524160827.png]]


#### List bugs
`List bugs` shows us the bug we had insert before:
![[Pasted image 20240524161057.png]]




## Binary static analysis
At this point we cannot retrieve other information from the dynamic usage of the binary, so we can perform a static analysis of it.

First of all we can see the security of the binary using `checksec` command:
```shell
checksec binary0
```

The result is:
![[Pasted image 20240524161230.png]]
- the binary seems to be well protected, but the strange behavior we got before could be interesting


What we can do now is to try to disassemble the binary and study the code using ghidra:
![[Pasted image 20240524161507.png]]


> NOTE: if you don't see this info on ghidra just click on `Window -> Decompiler view` and then  `Window -> Listing` 


So starting from the function `main_menu` we can move and inspect the code of the features provided by the binary file.


The first thing to do is to analyze the administration area and the privileged functions.


### option_check_privileges function analysis
This function basically prints our privileges:
![[Pasted image 20240524161934.png]]

### option_admin_area function analysis
This is a very interesting function for our challenge because it:
1. checks for privileges 
2. if we are `AUTORIZED` then it prints the flag

![[Pasted image 20240524162100.png]]


This because the function `is_authorized` basically checks the content of the variable `security_level`:
![[Pasted image 20240524162131.png]]




<mark style="background: #BBFABBA6;">So maybe to solve the challege we need a way to change our privilege to</mark> `AUTHORIZED` 
<mark style="background: #BBFABBA6;">And so, we need to change the variable</mark> `security_level` <mark style="background: #BBFABBA6;">content</mark>.


At this point we can study the other parts of the code.

Since we have done a dynamic analysis on the binary and since we didn't get any strange behavior except in the `Change bug` feature we can start on analyzing this part of the code.

So let's go back to `main_menu` and follow the code to reach `option_change_bug` (`main_menu -> option_manage_todos -> option_change_bug`)

###  option_change_bug function analysis
We have noticed that if we put a value greather than `9` we get a strange behavior.
So let's see what happens in the code:
![[Pasted image 20240524163252.png]]




It takes in input an integer using `get_input`, so let's see what `get_input` does:
![[Pasted image 20240524163356.png]]
- basically nothing relevant, it takes an input with `scanf` and returns 1 if the read value is an integer, 0 otherwise


So let's continue on `option_change_bug`. 

The function reads an integer and then:
1. checks the content of `* (long * )(bug_list + (long)local_18 * 0x20)` and so basically 
	1. takes the address of the "array" `bug_list` adds to the value we read in input multiplied by 32 (`0x20` is 32 in integers)
		1. if this value is `-1` then prints a message, otherwise gives us the possibility to add o subtract 1 to the content of this address


Now, the idea is that with this approach we can modify the value of the bugs, since we can access to them using the `id`, but the `id` is not validated so we can access to each memory address, passing an `id`.


This portion of code gives us the possibility to add 1 to the content of the address we are accessing if the value is not 5. It gives also the possibility to subtract 1 if the content is not 1:
![[Pasted image 20240524164333.png]]



So the big idea here is:
1. try to access to the memory address of the variable `security_level`
2. see what happens if we subtract 1 or if we add 1 to the content of this memory address


Another interesting thing here is that we have the access to addresses starting from the address of `bug_list`:
- so if we put as id `-1` we can have the access to the memory address `bug_list_address + (-1 * 32)`
- so if we put as id `1` we can have the access to the memory address `bug_list_address + (1 * 32)`


### security_level address getting
So at this point, to access exactly to the variable `security_level` we need to understand the offset we have between `bug_list` and `security_level`. 

We can do it performing a difference between their addresses.

From ghidra we can easily see the addresses double clicking on the name of the variables:

Double click on `bug_list`:
![[Pasted image 20240524165212.png]]
- so the address is `00105f00`

Double click on `securso if we put as id `-1` we can have the access to the memory address `bug_list_address + (-1 * 32)`ity_level`:
![[Pasted image 20240524165138.png]]
- so the address is `00104020`



We can easily calculate the difference between these hex values using a calculator:
![[Pasted image 20240524165401.png]]

Or also on this [website](https://www.calculator.net/hex-calculator.html?number1=105f00&c2op=-&number2=104020&calctype=op&x=Calculate) :
![[Pasted image 20240524165504.png]]



At this point we know that the offset is `7904`.
But we know from the line `* (long * )(bug_list + (long)local_18 * 0x20)` that our input is multiplied by `32`.


So to hit the right security_level address we need to divide `7904` by `32`. 

The result is `247`.


Now at this point, `bug_list`has an higher address related to `security_level`, so our input must be `-247`.


## Trying to hit security_level address

Let's try our idea to see if it works.

So tun the binary and add a dummy bug:
![[Pasted image 20240524165909.png]]


Let's try now to modify the `security_level` variable:
![[Pasted image 20240524170047.png]]
- in this case we are descreasing it by one

Let's see if our operation afflicted the variable we want to afflict.
So let's go back to the main menu and check the privileges:
![[Pasted image 20240524170149.png]]


Boom, something happened, so we are hitting the right address.

Now let's try what happens if we add 1 to the content of the variable.
But first re-execute the binary and the dummy bug adding.


At this point try to add 1:
![[Pasted image 20240524170410.png]]

So let's go back to the main menu and check the privileges:
![[Pasted image 20240524170442.png]]


The initial `U` disappeared. Very nice.

Let's add another 1 to the variable:
![[Pasted image 20240524170537.png]]

BINGO. 
Now try to access to the `Administration area`:
![[Pasted image 20240524170624.png]]

WE GOT THE PRIVILEGES.





## Flag getting
Now connet to the remote server using:
```shell
nc provinggrounds.sas.hackthe.space 31337
```


At this point we can re-do the same actions done in local and so:
- add a dummy bug
- increase 2 times the bug with `id` equal to `-247`
- access to the `Administration area`

![[Pasted image 20240524171004.png]]




<mark style="background: #BBFABBA6;">So the flag for part 1 is:</mark> `FLG_{4ppr0v3d_by_z3r06r4v17y_0bbefbabae14eb234}`
















