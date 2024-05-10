### Cristian Domenico Dramisino (12338532)

## Introduction

The challenge consists of 3 parts, where in each part we have to retrieve a flag.

Everything we need to play with the challenge can be dowloaded from [here](https://sas.hackthe.space/cms/files/330932a00699ec428bd0a4da1f13d66a4e4370c8149fa8d6aef20c8e938fd9c4/challenge3.zip)


# Challenge part 1 

## Introduction

This challenge can be accessed through this [link.](https://sas.hackthe.space/#/challenges/RISC-V%20part%201)

From the description of the challenge we can retrieve some useful information that we need take in mind:
```
SecPriv Intl. is proud to announce work on a custom chip based on the open ISA RISC-V! Unfortunately some people took this too literal and leaked internal documentation and the emulator SPOKE (SecPriv Originals: Kewl Emulator!).

The first flag is a warmup. It's lying around somewhere in memory between addresses 0x00001000 and 0x0000f000, all you need to do is find and print it!

Resources:

- The [leaked files and tools](https://sas.hackthe.space/cms/files/330932a00699ec428bd0a4da1f13d66a4e4370c8149fa8d6aef20c8e938fd9c4/challenge3.zip)
- The server hosting the actual flags can be reached at `spoke.sas.hackthe.space` port `8286`, e.g. with netcat `nc spoke.sas.hackthe.space 8286`.
- The [2019 spec](https://github.com/riscv/riscv-isa-manual/releases/tag/Ratified-IMAFDQC) Volume 1. Only the "RV32I" things are relevant here, which are chapters 1, 2, 24, and 25 (ignoring all the extensions), you can safely skip the rest.

General hints:

- The fake flags in the emulator and the real flags on the server are always at the same memory locations, no need to bruteforce anything on the server.
- Develop your exploits locally, and once they work, run them remotely.
- While you will have to write _some_ assembly, we don't expect you to become a RISC-V master in an afternoon. All parts can be solved in a handful of lines.
- The flag regex is `FLG_PT\d\{[A-Za-z0-9_\-]+\}`.
- All flags can be obtained without reversing the spoke binary. (of course you still can do that if you really want to)
- A hex editor might come in useful.
- Loops are super useful, and can be easily implemented with the various B_*_ and JAL(R) instructions!
```
- The ==**server that hosts the flags can be reached at**== `spoke.sas.hackthe.space` on port `8286`
	- <mark style="background: #BBFABBA6;">we can connect on it using</mark> `nc spoke.sas.hackthe.space 8286`
- We can download ==**everything we need for the challenges from**== [here](https://sas.hackthe.space/cms/files/330932a00699ec428bd0a4da1f13d66a4e4370c8149fa8d6aef20c8e938fd9c4/challenge3.zip)
- We can take a look on the documentation of RISC-V (focusing on chapter 1, 2, 24, and 25) from this [link](https://github.com/riscv/riscv-isa-manual/releases/tag/Ratified-IMAFDQC)
- **Loops are super useful, and can be easily implemented with the various B *  and JAL(R) instructions!** **==So maybe we need to use them to obtain the flags.==**



But the most important thing here is:
- **==The first flag is a warmup. It's lying around somewhere in memory between addresses 0x00001000 and 0x0000f000, all you need to do is find and print it!==**
	- this means that we could need to find a way to scan the memory on the server (from address  `0x00001000` to address `0x0000f000`) to obtain the first flag 


## Analysis of the downloaded resource

At this point we can extract the .zip file and take a look at the content:
![[Pasted image 20240510162536.png]]


### README.md analysis
The first thing to do is to read the `README.md` file, that could contain useful information. The content of this file is:
```
# README

Challenge 3 for Systems and Application Security.

## SPOKE

This contains the "SecPriv Originals: Kewl Emulator" (SPOKE) as binary executable.
You can pass it a RISC-V 32bit little endian binary, which it will execute directly from the first address.

To recreate the server setup, run `make docker-build` and start the container with `make docker-start`. To terminate, run `make docker-stop`. It will use port 8286. Note the the firmware on the server differs. Please test your exploits locally, the flags have the exact same length and locations in memory.

Reversing `spoke` is an option, but likely much more work than understanding the code execution and memory layout.

## ACE

To avoid writing bytes by hand, `ace` is a basic assembler/disassembler written in python, you can find more information in `ace/README.md`.

## LEAKS.md

The leaks directory includes additional ressources that are essential to understand the customizations of the implementation.
```
- From "You can pass it a RISC-V 32bit little endian binary, which it will execute directly from the first address" 
	- we **==can easily understand that the binary==** `spoke` **==can be used to execute RISC-V 32bit little endian binary files.==**
- From "To avoid writing bytes by hand, `ace` is a basic assembler/disassembler written in python, you can find more information in `ace/README.md`." 
	- we can understand that **==ACE is a tool used to assemble or disassemble files that contain RISC-V assembly instructions.==**
	- **Maybe it would be a good idea to investigate this tool**
- From "The leaks directory includes additional ressources that are essential to understand the customizations of the implementation." 
	- **==we can understand that maybe there are some customizations in the assembler/disassembler==**
	- **We need to investigate that**
- We ==**can recreate the server in local executing**== `make docker-build` ==**and start the container with**== `make docker-start` 


### ace tool analysis
We can open the ace folder and see that it contains some files:
![[Pasted image 20240510163527.png]]



If we take a look at the `README.md` file we can see that it explains how the assembler/disassembler works and which are the supported CRISP-V instructions.

Howver we can retrieve some useful information from this file that are:
```
## Installation

Run `pip install -e .` in this directory to make `ace` available on the commandline. Alternatively, install the dependencies from the `pyproject.toml` and run `python3 ace/main.py`.

### Custom Instructions

print rd, rs1, format

rs1 is the register whose content gets printed.
rs2 is the format, encoded as a number, like the immediates for the shifts:
0: utf-8 (lossy)
1: default 
2: binary
3: hex
4: utf-8 (lossy) with newline
5: default with newline
6: binary with newline
7: hex with newline
rd is the register where the return code gets written: 0 for success, 1 for errors.

```
- From the `## Installation` section
	- we can understand that the tool can be used directly with python, using `main.py`
- From `### Custom Instructions` section
	- we can understand that the tool supports a custom instruction called `print` that can be used to print the content of a register, and that allows different formats
	- the instruction has this format -> `print rd (register to store the result of printing), rs1 (register to print), format (that is an integer)`


Now we can try to understand how to use the assembler.

So looking at the documentation of RISC-V 32-bit we can write a simple assembly code to print the string 'a':
```assembly
_start:
    li a0, 97
    print t0,a0,0
```
- So here with `li a0, 97` we charge in the register `a0` the value `97` that represents `a` in ASCII code
- with `print t0,a0,0` we are printing the content of `a0` using the `utf-8` format and we are storing the result of the printing into the register `t0` 


>NOTE: Here we are using the custom tool instruction `print`, we cannot find it in the RISC-V documentation


At this point we can easily store it into the `print_a.txt` file and try to use the tool to convert it into binary bytes.


To understand how to use the tool we can execute:
```shell
python3 main.py
```

And we obtain:
![[Pasted image 20240510165249.png]]

In addition if we open the `main.py` file we can see:
![[Pasted image 20240510165414.png]]
- So we need to use also `--input` to insert the input file we want to assemble


So the final command is:
```shell
python3 main.py a --input print_a.txt
```

And the result is:
![[Pasted image 20240510165644.png]]
- quite incomprensible but no errors from the tool, so we are sure it works.


At this point we can try to execute it on Spoke...



## Spoke usage
First of all we can use it in local and so we can connect on it using:
```shell
nc localhost 8286
```

The result of this command is:
![[Pasted image 20240510170355.png]]



So we need to encode our binary file produced by `ace` tool with base64. This is an easy task, since we can use the `base64` command:
```shell
python3 main.py a --input print_a.txt | base64
```
- this sends the output of the python script into the `base64` command
- ![[Pasted image 20240510170542.png]]


So we copy and paste `NwUAABMFFQYLBQUA` on the local "server" and we obtain:
![[Pasted image 20240510170635.png]]
- as we can see it printed `a`



## Flag getting assembly code


Now we know how everything works and for this reason we can start to write an assembly code that:
1. initializes 2 registers with the start address and the end address of the memory we can analyze
	1. we will use registers `t1` and `t2` since they are temporary registers (from the documentation page 137)
2. executes a loop in which we 
	1. check if the register `t1` is greater or equal to `t2` (this because we will increase `t1` by one byte per time in order ot scan the memory range)
		1. when this is true, then the program must stop
	2. read the content of the register `t1` 
	3. check if the content of the register `t1` is not equal to 0 
		1. if it is equal to zero, then it is empy and so we don't need to print it
		2. if it is not equal to zero then we "call the method" `print_no_zero` to print the content 

>NOTE: the `ace` tool doesn't accept comments so you need to use this code:

``` c
_start:
    li t1, 0x00001000   
    li t2, 0x0000f000

loop:
    bgt t1, t2, end 
    lbu t5, 0(t1)    
    bnez t5, print_no_zero
    addi t1, t1, 1
    j loop

 
print_no_zero:
    print t0,t5, 0
    addi t1, t1, 1
    j loop

end: 
    li a7, 10          
    ecall  
```




But here you can find the commented one:
``` c
_start:
	#start address 
    li t1, 0x00001000  

	#end address
    li t2, 0x0000f000

loop:   
	#if t1 is bigger than t2 then we have to stop the program
    bgt t1, t2, end 

	# copy the content of t1 into t5, 0() means we start to copy from offset 0 of t1
    lbu t5, 0(t1)    

	#if t5 is not 0 then we want to print it
    bnez t5, print_no_zero

	#we add 1 to t1 and we store the result on t1 in order to move of 1 byte in the memory
    addi t1, t1, 1

	#we jump to the loop "method", so we are looping at all
    j loop

 
print_no_zero:
	#here we print the content of t5 in utf-8 format and store the result of the printing into t0
    print t0,t5, 0

	#we add 1 to t1 and we store the result on t1 in order to move of 1 byte in the memory
    addi t1, t1, 1
	#we jump to the loop "method", so we are looping at all
    j loop

end: 

	#this block is used to stop the execution, nothing relevant
    li a7, 10           
    ecall   
```



So we try to execute it in local:
```shell
python3 main.py a --input part1.txt | base64
```
![[Pasted image 20240510172440.png]]

And then:
![[Pasted image 20240510172504.png]]




So it works, now we need to take the real flag, so we perform the same identical steps, but we connect to:
```shell
nc spoke.sas.hackthe.space 8286
```

![[Pasted image 20240510172632.png]]



<mark style="background: #BBFABBA6;">So the part 1 flag is</mark>: `FLG_PT1{l00k_mum_n0_c0mp1l3r_ju5t_a4s5embly}`




# Challenge part 2

## Introduction

This challenge can be accessed through this [link.](https://sas.hackthe.space/#/challenges/RISC-V%20part%202)


From the description of the challenge we can retrieve some useful information that we need take in mind:
```
This flag is at a place where you only have limited access, and you don't seem to have the necessary privileges to print it... But I'm sure you'll manage :)

Hints:

- Take a look around the leaked files
```
- From "you don't seem to have the necessary privileges to print it" 
	- we can understand that there is some protection mechanisms used to manage the privileges on the memory accessing
- From the hint "Take a look around the leaked files"
	- we can understand that we need to analyze the leaked files 




## Leaked files analysis

If we take a look at the `leak` folder we can see that it contains some different files:
![[Pasted image 20240510173249.png]]

So we can start to analyze them

### chat.log file analysis
If we open the `chat.log` file we can retrieve some useful information:
```
----------
< Dave
< Dave
< A customer said they were implementing some OS thing, and asked me how to do this on our chip, since it's in user mode by default?
< Dave
> Tell them they are free to code and ship their own firmware, that runs in supervisor mode.
< They say that doesn't cut it, they're trying to make a full OS.
> Are they on our trusted partners list?
< According to Carla, yes.
> Then remind them they signed an NDA, and then tell them about the secret supervisor instruction: Encoded like a U- or J-type, opcode for custom-1 block, rd encodes the mode to switch to (0 is user, 1 is supervisor), the immediate bits are 0010 0100 1010 0000 0010 (beware of little endian).
> Also make sure they understand that if they use it, they are responsible for managing the rest of the chip, and can't use ECALLs, because they reset the privilege mode to user mode at the end.
< Thank you Dave!
< Isn't this a bit insecure?
> Oh, would you look at the calendar, it's Friday!
< ?
< What does that mean
> You know what, how about we go for coffee?
> Like, right now.
----------
< Good morning :)
> Good morning
< I know we're making the SP-24 a small embedded SoC, and as such don't implement anything from the privileged spec.
> True
< And that's why we wanted to make our own privilege system
> yes
< Aaanyway, I found a really sweeeet deal on some privilege management chips! It implements all three modes (user, supervisor, machine), and is adaptable enough that we should be able to just map our stuff on there and call it a day!
< User mode by default, supervisor for the firmware stuff... and machine for the secret stuff. I can totally see this working!
> Huh
> I guess that could work
> Using machine mode for key storage feels a little janky, but as long as no code ever runs in machine mode, it should be fine?
< Yeah, right?
< I'll talk to Emma about ordering them.
----------
```
- From the section "Then remind them they signed an NDA, and then tell them about the secret supervisor instruction: Encoded like a U- or J-type, opcode for custom-1 block, rd encodes the mode to switch to (0 is user, 1 is supervisor), the immediate bits are 0010 0100 1010 0000 0010 (beware of little endian)."
	- **==we can understand that there is a secret instruction that can allow us to change our mode==**
	- **==the value 0 is for a normal user, the value 1 for a supervisor user==**


### sp-24.md file analysis
If we open the `sp-24.md` file we can retrieve some useful information:
```
# SP-24

Our newest SoC, the SP-24, improves upon our older designs in various ways:
* The SP-24 is SecPriv's first RISC-V based general purpose SoC
* 4GB of RAM*
* ...

*The SP-24 reserves a small part of memory (addresses 0xffff0000 - 0xffffffff) for its internal usage.
This memory is not accessible to user programs, and is used for firmware calls.
To support secure key storage, the last few addresses (0xffffff00 - 0xffffffff) can only be acessed during manufacturing.
```
- From the section "The SP-24 reserves a small part of memory (addresses 0xffff0000 - 0xffffffff) for its internal usage. This memory is not accessible to user programs, and is used for firmware calls. To support secure key storage, the last few addresses (0xffffff00 - 0xffffffff) can only be acessed during manufacturing."
	- **==we can understand that the addresses from==** `0xffff0000` ==**to**== `0xffffffff` are ==**reserved to the internal usage, so not accessible as a normal user but maybe accessible for a supervisor user**==
	- **in additionthe addresses from**`0xffffff00` **to** `0xffffffff` can be acessed only during manufacturing 

### christmas_party folder analysis
From this folder we can retrieve this image:
![[Pasted image 20240510174423.png]]

That is a confirmation about the info discovered before...


### What we know
1. **==there is a secret instruction that can allow us to change our mode (0 for normal user, 1 for supervisor user)==**
2. **==the memory addresses from==** `0xffff0000` ==**to**== `0xffffffff` ==**are reserved to the internal usage, so not accessible as a normal user but maybe accessible for a supervisor user

So let's try to understand if we can use this secret instruction...

## Instructions analysis
If we open the `ace` tool source code we can notice a file called `instruction.py`, so we can start on taking a look on it.

We can see in it that it encodes all the CRISP-V supported instructions as classes.

With a deep analysis of the file we can discover that at the end of it we can find this interesting class:
![[Pasted image 20240510174832.png]]
- it seems to be a custom instruction
- it requires a value called mode and from assemble method we can also understand that mode is a integer
- it seams to represent the secret instruction we can read about in the `chat.log` file

So let's understand how the tool parses the file that contains the CRISP-V code we give him...

## parse.py file analysis


If we take a look at this file we can see that when the CRISP-V instructions file is parsed, the tool uses a switch for each line of the file.

This switch checks the presence of CRISP-V instructions and create the equivalent object to encode it: 
![[Pasted image 20240510175305.png]]
- we can see that it contains also the custom instructions, but in this case only the `print` custom instruction

It seems that the privilege one is not considered here...


So, what if we change the switch to add the parsing of the privilege one?

## parse.py file changing to unlock the privilege instruction

So let's change the `parse.py` file in this way:
![[Pasted image 20240510175706.png]]
- in this way, when a line starts with `privilege` python creates a `CustomPrivilege` object and returns it

Now we know that 0 is used for normal user and 1 for supervisor user, and we know that `CustomPrivilege` needs a integer (`mode`) so we use directly:
```python
return instruction.CustomPrivilege(1)
```


Let's try to understand if we are able to read the protected memory with these changes...

## Flag getting assembly code

The idea is to reuse completely the assembly code used for the first part of the challenge but changing the memory addresses and adding a `privilege` line. So let's try to use:

```c
_start:
    li t1, 0xffff0000   
    li t2, 0xffffffff
    privilege 

loop:
    bgt t1, t2, end 
    lbu t5, 0(t1)    
    bnez t5, print_no_zero
    addi t1, t1, 1     
    j loop

 
print_no_zero:
    print t0,t5, 0
    addi t1, t1, 1
    j loop
    
end: 
    li a7, 10          
    ecall
```
- The only changes are in the _ start section.
	- We changed the memory addresses 
	- We added the line privilege





So we try to execute it in local:
```shell
python3 main.py a --input part2.txt | base64
```
![[Pasted image 20240510180650.png]]

And then:
![[Pasted image 20240510180707.png]]


So it seems to work...

Let's try it on the real server:
![[Pasted image 20240510180740.png]]

<mark style="background: #BBFABBA6;">So the part 2 flag is</mark>: `FLG_PT2{0n3s_l34k_1s_4n0th3rs_d0cum3ntat10n}`

