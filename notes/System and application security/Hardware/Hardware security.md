Hardware security is focused on the protection of physic components of a system in order to avoid unauthorized access.



In this image we can see, for each type of technology used by hardware, the complexity of this technology.

![[Pasted image 20240508174000.png]]



## Chips 
A chip is an electronic component made on a silicon that contains various electronic components as transistors, resistors etc...

We have different types of chips that can be summarized into:
- Microprocessors -> they execute instructions and perform data calculation
- Microcontrollers -> similar to microprocessors but designed for specific applications
- Memory chips -> chips used to store data and instructions
	- *for example RAM, ROM, flash memories etc*
- Communication chips -> these chips enable communication between devices
- Specialized chips -> designed for specific functions

### CPU example
![[Pasted image 20240508174535.png]]



### Memories
We have different types of memory chips:
- ROM -> Read-Only-Memory
	- it is used to store data in a permanent way, so when we turn off the pc those data will remain
	- it is read only because *generally* we can read data but we cannot overwrite it 
	- *for example we can use it to store data about: system firmware etc*
- RAM -> Random-Access-Memory
	- It is used to store temporary data and instructions on which the CPU can access rapdly when it has to execute instructions, so the the program instruction can be found there
		- Random access -> we can read/write on each cell without taking in count other cells; the time to access to each cell is equal
- PROM -> Programmable Read-Only Memory
	- We can program it one time during the fabrication process. We can then only access to it and not modify it.
	- We can program it using *fuses*
- EPROM -> Erasable Programmable Read-Only Memory
	- Similar to PROM but we can erase and reprogram it 
	- The erasing is done using ultraviolet light
- EEPROM -> Electrically Erasable Programmable Read-Only Memory
	- Equal to EPROM but we can erase it using electonic components, or in general using softwares
- NVRAM -> Non-Volatile Random Access Memory
	- It is a mix between a RAM and a ROM 
	- It stores data in a permanent way and for this reason when we shut down the pc we will find the stored data, as a ROM
	- It allows the possibility to read and write data in any possible order and rapdly, as a RAM

In general we have to understand that the names are not alligned with capabilities.
- *For example we can update the BIOS ROM, also if it is a ROM*




### Printed Circuit Boards
A printed circuit borad is a platform on which we can plug electronic components, so basically are platforms that can be managed and used for different purposes with respect to the component plugged on them.

*Example of them:*
- *Arduino*
- *Raspberry PI*
![[Pasted image 20240508180330.png]]




### Useful tools for hardware security
For sure we cna use softwares.

The idea is that the most useful ones are:
- [tweezers](https://wiibrew.org/wiki/Tweezer_Attack), screwdriver, cables, soldering iron
- interfaces (UART,..), debuggers(JTAG,..), adapters
- multimeter, oscilloscope, logic analyzer
- microscope (optical, electron), acid
- electromagnetic probe, focused ion beam (“laser”)




## Attacks classification on hardware security

### Logical vs Physical
A logical attack is an attack that exploit software:
- buffer overflows, permissions, protocols

A physical attack is an attack thta exploit directly an hardware:
- trace connections, read/insert electronic signals

### Passive vs Active
A passive attack is an attack in which we just analyze the functionalities of the device and we don't act on it:
- follow the specification etc

An active attack is an attack in which we manipulate the behavior of the device or in general we interact with it:
- influence the unexpected ways and observe, change/insert signals


### Invasive vs Non-invasive
An invasive attack is an attack in which we start to manipualte the device at all, so we leave marks on it:
- tamper evident, it is very powerful

A non-invasive attack is an attack in which we don't leave permanent marks on the hardware:
- measurements and easy interactions


## Attack goals
The goals can be very different:
- bypass security checks/assertions (smartcards etc)
- escalate privileges to run softwares (consoles etc)
- dump the software to analyze it (IoT, routers etc)
- unlock restricted features (transit cards etc)
- extract cryptographic keys 
- open back doors on the device

### Attack examples

### Time side channel attack

![[Pasted image 20240508181351.png]]
- here an attacker can start to inject first a single letter, when it is correct the code will try to check the second one
	- so the time consumed is higher than the non correct letters because we check letter by letter and when it is non correct we return false


#### Simple solution
![[Pasted image 20240508181548.png]]



### Power analysis side channel attack

![[Pasted image 20240508181619.png]]
- here an attacker can extract the key because when the bit is 1 then the algorithm performs an heavy operation (A = A* m mod n)
	- so if he analyzes the power consumed he can understand when a 1 is used in the key



### Contromeasures to Side channel analysis

1. Make critical algorithm independent from data used in it
2. Use masking and so insert dummy operations
3. use secure components
	1. power balancing (each operation must use constant power)
	2. create noisy power usage
	3. avoid EM radiation (electromagnetic radation), that can be used to perform a power analysis



### Fault injection attack
![[Pasted image 20240508182237.png]]
- here an attacker can force to skip this instruction and return true directly


#### Clock glitching
In this attack we insert a clock pulse that is too fast for the CPU to be processed.

This allows to skip slow instruction such as memory access. But it doesn't affect the fast instructions such as Instruction Pointer increment.


#### Voltage glitching
In this attack we increase or decrease the power for a short period.
- Most of the Integrated Circuits (IC) use an internal clock

It can lead to corrupted memory or to skip instructions.


### Contromeasures to Fault Injection
We can use some hardware protections such as usage of:
- additional capacitors or shielding
- sensors to detect fault injection
- internal filtering


But also software protections are used, such ase using of:
- multiple checks
	- *example: if((err== 0) && (err== 0)) return true;*
	- this because to bypass the check we need to perform double fault injection
- program counters
	- *for example:*
```c++
... 
canary++; 
... 
canary++; 
... 
if((err==0) && (canary == 2)) return true;
```



### Rowhammer: FI triggered by software
This attack can be done without having the access to the hardware even remotely but it is basically an hardware attack.

The idea is to hit repeatedly specific memory cells in order to generate electrical interferences that can allow to generate memroy errors.

It is done over the DRAM (dynamic RAM).

A DRAM is composed by a grid of memory cells, in which every cell stores a bit in the form of an electronic charge, so the attack is based on:
1. A repeated write/read operation is done over specific rows that are close to each other.
2. This causes an electronical interference that can affect neighbors cells (generally we have a bit flipping, where the bits of the neighbor is flipped)

If an attacker can use it to manipulate the data maybe he can corrupt sensitive data or also to perform a privilege escalation flipping maybe the root bit.



### More Countermeasures for protecting sudo operations: Fault Injection

We can huse the harde sudo:
- we can test the sudo activity on expected values
- we can use values for sudo that cannot be changed using the Rowhammer, so basically we can use multiple bit numbers (to make the bit flipping very difficult)


*Example:*
- *in red the wrong solution
- *in green the correct one*

![[Pasted image 20240508185003.png]]



## Other attacks

### Microarchitectural attacks

It can include:
- the abuse hardware-optimizations such as speculative execution
- finding of undocumented instruction or behaviour


### Expensive attacks
Decapping attacks that allows to remove protective layers of chips.

It is used to perform:
- visual inspection, (EM) probing
- fault injection
- reset the fuses

## Secure components != Secure System

In general we have different names but same principles:
- smartcards
- Hardware Security Module (HSM)
- Trusted Platform Module (TMP)
- Enclaves 

![[Pasted image 20240508191907.png]]




### Usecase: HSM 

An example was:
As a certificate authority we could create a root certificate in the HSM.

We use the interface to sign a new certificate and the interface is secured by using layers of access control.
![[Pasted image 20240508193538.png]]

But DigiNotar in 2011 was hacked just skipping some steps.





## RISC-V 
It is an architecture of a set of instructions, designed by thr Instruction Set Architecture (ISA), open and free.
It is designed to be simple, modular and extensible.


The idea is that only the members of the the ISA can use trademark and vote for instructions standard.




## RISC-V for the challenge

We can find the manual [here](https://github.com/riscv/riscv-isa-manual/releases/tag/Ratified-IMAFDQC) 
It is for 32 bit and usess the little endian.

We will focus on chapter 1, 2, 24, 25

Other information can be found:
- Here to encode/decode instructions [here](https://luplab.gitlab.io/rvcodecjs/)
- Compiler / decompiler [here](https://godbolt.org/)

![[Pasted image 20240508194147.png]]


We have to only base integer instruction using:
- `-march=rv32i`
- `-mabi=ilp32`

