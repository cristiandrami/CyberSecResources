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















# Challenge part 2

## Introduction
We can access to this challenge through this [link](https://sas.hackthe.space/#/challenges/Binary%201)


From the description of the flag we don't get many useful information.

We just understand that:
```
Your mind races with possibilities. The stakes are higher now, and the challenge more complex. But this is what you live for—pushing the limits, proving yourself, and making a mark.

You will need the following information:

- The [system files](https://sas.hackthe.space/cms/files/dd4b98a8266fb8c74061513f4a2826ea895192f6759d7c52424fcda3981bab56/binary1.zip).
- The system running the binary and the flag can be reached via `nc cyberphant0m.sas.hackthe.space 65432`
- The flag regex ix `FLG_\{[a-z0-9_]+\}`
```
- we can download the files we require for the challenge from this [link](https://sas.hackthe.space/cms/files/dd4b98a8266fb8c74061513f4a2826ea895192f6759d7c52424fcda3981bab56/binary1.zip)
- we can reach the server that runs the binary using `nc cyberphant0m.sas.hackthe.space 65432`





## Binary dynamic analysis
What we do here is to perform a dynamical analysis of the binary, and so we try to use it in order to understand what are its functionalities.

So we can easily give the execution permissions to it:
```shell
chmod +x chall
```

At this point we can execute it:
```shell
./chall
```

We can see that we have a menu here that gives us the possibility to choose what to do:
![[Pasted image 20240528182511.png]]


### Add bot feature interaction 

Let's try to interact with the `Add bot` feature:
![[Pasted image 20240528183003.png]]

We can play with it and we can understand that maybe under the cover there is an array that takes trace about the data we put in it.

The relevant things there are:
- we can choose only `id` between 0 and 9
	- if we try an `id` < 0 or > 9 then the program exit:
		- ![[Pasted image 20240528183230.png]]
- we can choose the `length` of the identifier
- we can arbitrary identifiers (a string) 
	- if we put a string that is greather than the `length` we chose then the program does nothing (maybe it is truncated by the code)

If we try to add 2 times a bot with the same id the program terminates:
![[Pasted image 20240528184649.png]]


If we put a string instead an integer in the `length` or `id` input, the program terminates:
- ![[Pasted image 20240528184737.png]]
- ![[Pasted image 20240528184831.png]]


### Remove bot feature interaction 
Let's try to interact now with `Remove bot`  feature so let's remove the previous added bot (`id` 0):
![[Pasted image 20240528183739.png]]

If we play with it we can see that:
- if we try to remove a bot already removed or never added then the program exit without errors:
	- ![[Pasted image 20240528183900.png]]


If we try to put a string in the `id` field the program terminates:
- ![[Pasted image 20240528184905.png]]

### Attack feature interaction 
Let's try to interact now with `Attack` :
![[Pasted image 20240528184035.png]]

Ok, we have to assign at leat 3 botnets, so lets do it with the add feature and retry to perform the attack.

![[Pasted image 20240528184150.png]]
- so it prints our added botnets

Here we can see that if we put a string longer than the `length` we chose then the program truncate it to the `length`.


### Help feature interaction 
Let's try to interact now with `Help` :
![[Pasted image 20240528184331.png]]

This is not so relevant, it just explains the functionalities of the program, but we can see that it says `The default capacity is 1.000 bots, which can only be changed by our admins.`

Maybe there is a secret feature?

### Quit feature interaction 
This just closes the program:
![[Pasted image 20240528184545.png]]



## Binary static analysis
At this point we cannot retrieve other information from the dynamic usage of the binary, so we can perform a static analysis of it.

First of all we can see the security of the binary using `checksec` command:
```shell
checksec chall
```

The result of this command is:
![[Pasted image 20240528185024.png]]

Mhhh, there is No PIE and Partial RELRO, but what does this mean?
1. **PIE (Position Independent Executable)**: Indicates whether the executable was compiled as a position-independent executable. 
	1. An executable with PIE can be loaded into memory at random addresses, making it more difficult to exploit certain vulnerabilities. 
	2. "No PIE" means that the binary was not compiled with this option and therefore will be loaded into memory at a fixed address (usually 0x400000 on 64-bit systems).
	3. So this means that at each execution the binary is loaded in the same memory address and so the content of the binary will be always in the same position (like the functions in it)
2. **RELRO (Relocation Read-Only)**: This indicates the degree to which the relocation tables of the executable are protected in memory. 
	1. "Partial RELRO" means that only some parts of the relocation tables are made read-only, while others remain writable.
	2. So it is possible to rewrite these parts of the relocation tables.




Now we can try to disassemble the code with ghidra in order to see if we can get useful information about it.


### Using ghidra on the binary

What we can do is -> charge the binary file in ghidra and try to disassemble it.

Looking at the function it seems to be a complete mess, the function have no meaningfull names:
![[Pasted image 20240528190057.png]]

But we can try to rename them if we get the function that represents the main menu...

If we play around we can notice that the main menu is managed by the function `FUN_00401256`:

![[Pasted image 20240528190212.png]]


So let's rename it in `main_menu` using:
- right click on the name (the yellow one in the image) -> Rename function
- ![[Pasted image 20240528190411.png]]


Now, looking at the switch and looking at the functionalities we can easily rename all the features functions:


![[Pasted image 20240528190747.png]]


We can also notice a strange value in the switch that is `0xffffffff`. with a rapid research on internet we can see that it is just `-1`

So let's rename the function in this case as `maintenance_function` as adviced by the print:
- ![[Pasted image 20240528191235.png]]



Another strange function that we can see in the main menu is:
- ![[Pasted image 20240528191342.png]]


If we try to enter in it we can notice that it seems to be a initialization function:
![[Pasted image 20240528191512.png]]

So let's call it `init_function`:
![[Pasted image 20240528191706.png]]

In addition we can see that it initializes a value to `1000`

#### add_bot function analysis

What happens in this function?
![[Pasted image 20240528191819.png]]

1. it checks if the bot with the `id` we insert already exists, it checks the presence of 1 in a certain memory location
2. if this memory location is not equal to 1 
	1. then it asks us for the `length` of the identifier
3. at this point it asks us to insert the identifier (a string)
	1. interesting thing it uses `malloc` to store it, so it is stored in the heap memory space

Now using the `attack` function we know that each bot has an `identifier` string and a `capacity` that for default is `1000.

So maybe the `init_function` is used to initialize the bot array.

So let's construct a structure in ghidra that manages directly these data, in order to make the code more readable.

### bot structure adding 
From the `init_function` we can see that a bot is composed by 3 elements.
![[Pasted image 20240528192641.png]]

1. the first one is of 8 bytes, so maybe it is used as string pointer 
2. the second is of 4 bytes and it is initialized to 1000
3. the third one is of 4 bytes and it is initialized to 0

Maybe they can be 
1. identifier string pointer
2. capacity integer
3. is or not integer


So let's construct a bot structure in this way:
- Data type manager -> chall -> New -> Structure

And add these values:
![[Pasted image 20240528193130.png]]
- let's maintain this order

Give it `bot` name:
![[Pasted image 20240528193411.png]]

Click on the save icon on the right:
![[Pasted image 20240528193206.png]]

![[Pasted image 20240528193421.png]]



At this point we can understand also from the `init_function` that `param_1` is the start pointer for the array of bots, so let's change his name to `bot_array` and the type to `bot *` .


![[Pasted image 20240528193617.png]]
- it really better

To do it just 
1. right click on `param_1 ` -> Rename variable -> put `bot`
2. right click on `bot` -> Retype Variable -> put `bot*`



Let's change each function that uses it and also the variable in the main menu to `bot*`:
- ![[Pasted image 20240528193941.png]]
- ![[Pasted image 20240528194020.png]]
- ![[Pasted image 20240528194114.png]]
- ![[Pasted image 20240528194139.png]]
- ![[Pasted image 20240528194226.png]]
	- we changed here the name of the variable to `bot_array`

Now we have a more comprensible code to analyze.

