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
	- **==we can understand that the tool can be used directly with python, using==** `main.py`
- From `### Custom Instructions` section
	- ==**we can understand that the tool supports a custom instruction called**== `print` ==**that can be used to print the content of a register, and that allows different formats**==
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
1. ==**initializes 2 registers with the start address and the end address of the memory we can analyze**==
	1. we will use registers `t1` and `t2` since they are temporary registers (from the documentation page 137)
2. ==**executes a loop**== in which we 
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
    ebreak  
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
    ebreak   
```



So we try to execute it in local:
```shell
python3 main.py a --input part1.txt | base64
```
![[Pasted image 20240518125511.png]]
- so the base64 is `NxMAABMDAwC38wAAk4MDAGPAYwIDTwMAYxYPABMDEwBv8B//iwIPABMDEwBv8F/+cwAQAA==`


And then:
![[Pasted image 20240518125609.png]]




So it works, now we need to take the real flag, so we perform the same identical steps, but we connect to:
```shell
nc spoke.sas.hackthe.space 8286
```

![[Pasted image 20240518125649.png]]



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
- From "*you don't seem to have the necessary privileges to print it*" 
	- ==**we can understand that there is some protection mechanisms used to manage the privileges on the memory accessing**==
- From the hint "*Take a look around the leaked files*"
	- ==**we can understand that we need to analyze the leaked files** ==




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
- From the section "*The SP-24 reserves a small part of memory (addresses 0xffff0000 - 0xffffffff) for its internal usage. This memory is not accessible to user programs, and is used for firmware calls. To support secure key storage, the last few addresses (0xffffff00 - 0xffffffff) can only be acessed during manufacturing.*"
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
    ebreak
```
- The only changes are in the _ start section.
	- We changed the memory addresses 
	- We added the line privilege





So we try to execute it in local:
```shell
python3 main.py a --input part2.txt | base64
```
![[Pasted image 20240510180650.png]]
- so the base64 is `NwP//xMDAwC38///k4Pz/6sgoCRjwmMCA08DAGMWDwATAxMAb/Af/wsFDwATAxMAY1RzAG/wH/5zABAA`



Try to run it in local:
![[Pasted image 20240518125829.png]]


So it seems to work...

Let's try it on the real server:
![[Pasted image 20240518125944.png]]

<mark style="background: #BBFABBA6;">So the part 2 flag is</mark>: `FLG_PT2{0n3s_l34k_1s_4n0th3rs_d0cum3ntat10n}`



# Challenge part 3

## Introduction

This challenge can be accessed through this [link.](https://sas.hackthe.space/#/challenges/RISC-V%20part%203)


From the description of the challenge we can retrieve some useful information that we need take in mind:
```
Now this flag is protected a lot better. It requires you to know a secret, but that secret doesn't seem to contained in the leak (it's a secret after all). But maybe there is a place where you can find it?

Hints:

- If you haven't gotten flag 2 from the server yet, you probably don't have all the pieces to put this one together...
```
- From "*But maybe there is a place where you can find it?*" 
	- ==**we can understand that there is some useful information hidden somewhere**==
- From the hint "*If you haven't gotten flag 2 from the server yet, you probably don't have all the pieces to put this one together...*"
	- **==we can understand that to find the last flag it is necessary that we have already discovered the second flag.==**
		- <mark style="background: #BBFABBA6;">So maybe there is something related with the privileges that we can exploit</mark>


In addition we have another hint from professor Jakob:
```
Hi! We don't give major hints in DMs, but I'd like to point out the README in the zip contains the line "Note the the firmware on the server differs." (from the `public_firmware`), which is relevant for flag 3.
```
- **==So maybe we have to focus on the differences between the public firmware contained in the leak and the firmware used by the remote server==**


At this point we can start on trying to extract the firmware from the remote server.


## Remote firmware extraction

To try to extract the firmware from the remote server we could use the same approach used to print the flag in the part 2.

So basically what we want to do now is to print the content in the range `0xffff0000 - 0xffffffff`, since we know from the leaked file `sp-24.md` that there is contained in it the firmware.

To do this we can use this RISC-V "code", that is really similar to the one used to get the flag 2:
```C
_start:
    li t1, 0xffff0000
    li t2, 0xffffffff
    privilege


loop:
    bgt t1, t2, end 
    lw t5, 0(t1) 
    print t0,t5, 7
    addi t1, t1, 4
    j loop 

end: 
    ebreak
```
- we use the `privilege` instruction since we need the `supervisor` privileges to have access to this memory portion
- we add 4 bytes per time to `t1` because we know that RISC-V instruction are 32 bits and so 4 bytest1

>NOTE: **==differently from the "code" used for flag 2 here we print also 0 values, because they we want to extract everything from this portion of memory, since we need to reconstruct the firmware byte by byte==**


> NOTE: we print in `hex format` -> `7` to don't lose information



At this point we can save this "code" into the `firmware_extractor.txt` file and then execute it on the remote server.

We always use `ace` to assemble it and `base64` to represent it in this encoding format:
```shell 
python3 main.py a --input firmware_extractor.txt | base64
```
- the base64 is `NwP//xMDAwC38///k4Pz/6sgoCRjymMAAy8DAIsCfwATA0MAb/Af/3MAEAA=`


And then executing it using:
![[Pasted image 20240518130454.png]]

But we can notice that there are a lot of initial 0s.
So maybe we can reduce the memory range...

If we play with the memory ranges, we can see that ==**the firmware starts effectively at memory location**== `0xffff1000`, since everything before it are 0s.

In addition **==we can also reduce the the range to==** `0xffff1100` since everything after it are 0s.


So we can change the "code" in:
```C
_start:
    li t1, 0xffff1000
    li t2, 0xffff1100
    privilege


loop:
    bgt t1, t2, end 
    lw t5, 0(t1) 
    print t0,t5, 7
    addi t1, t1, 4
    j loop 

end: 
    ebreak
```


The base64 for this "code" is `NxP//xMDAwC3E///k4MDEKsgoCRjymMAAy8DAIsCfwATA0MAb/Af/3MAEAA=`


The result is:
```
10000293
0058fc63
00589893
00000297
01088893
005888b3
00088067
00000893
00000073
00000000
00000000
00000000
00000000
00000000
00000000
ff1870e3
00080283
0002800b
00180813
fe0298e3
fcdff06f
00000000
00000000
00000513
fb187ee3
fa078ce3
00082283
00550533
fff78793
fedff06f
00000000
00381513
01050533
01050533
f95ff06f
00000000
00000000
00000000
00000000
46880ab7
630a8a93
8d2a6aab
f0100293
0002a303
5f4742b7
c4628293
00000513
00628263
00150513
dac08ab7
a1ca8a93
8d2a6aab
f4dff06f
00000000
00000000
00000000
00000000
00000000
00000000
00000000
00000000
00000000
00000000
00000000
00000000
```



At this point we need to store this result into a file called `hex_strings.txt`,  convert it into a binary file, in order to reconstruct the firmware, and then disassemble it using `ace`.


>NOTE:<mark style="background: #FF5582A6;"> when we process these hex values, we have to convert it into little endian.</mark> Otherwise it won't work at all.

To convert these hex values to a binary file we can write a python script.

### Hex to binary firmware conversion

So let's write a python script that:
1. reads the file containing the hex strings obtained before
2. convert these strings into little endian form
3. store these converted values into a binary file


```python
def hex_to_little_endian(hex_str):
    # this splits the hex string into groups of 2 characters, e.g., 00000093 -> ['00', '00', '00', '93']
    two_chars_split_list = [hex_str[i:i+2] for i in range(0, len(hex_str), 2)]
    
    # this reverses the order of the list we have created, e.g., ['00', '00', '00', '93'] -> ['93', '00', '00', '00']
    little_endian_list = two_chars_split_list[::-1]

    # returns the entire little endian hex: 93000000
    return ''.join(little_endian_list)

def create_binary_file(hex_list, file_name):
    with open(file_name, 'wb') as bin_file:
        for hex_str in hex_list:
            little_endian_hex = hex_to_little_endian(hex_str)
            little_endian_bytes = bytes.fromhex(little_endian_hex)
            bin_file.write(little_endian_bytes)


# file from which to read hex strings
input_file_name = "hex_strings.txt"

# output binary file name
output_file_name = "firmware.bin"

# read hex strings from the file
hex_strings = []
with open(input_file_name, 'r') as file:
    for line in file:
        hex_strings.append(line)

# create the binary file
create_binary_file(hex_strings, output_file_name)
```

At this point we can execute:
```shell
python3 hex_to_binary_firmware.py
```

To obtain: 
-  ![[Pasted image 20240517201439.png]]





<mark style="background: #BBFABBA6;">Now we have everything we need to compare the leaked firmware to the remote used one.</mark>


## Firmware comparison (remote one vs leaked one)

So we can start to compare the firmware.

The first step to do it is to disassemble them and we can do it using `ace`.

We can find the public firmware at `server-config/public_firmware.bin`. We can copy and paste it in the ace directory.

So we execute:
```shell
python3 main.py d --input public_firmware.bin | base64
```

The result will be:

```c
addi x5,x0,256
bgeu x17,x5,24
slli x17,x17,5
auipc x5,0
addi x17,x17,16
add x17,x17,5
jalr x0,x17,0
addi x17,x0,0
ecall
0x0
0x0
0x0
0x0
0x0
0x0
bgeu x16,x17,-32
lb x5,0(x16)
0x2800b
addi x16,x16,1
bne x5,x0,-16
jal x0,-52
0x0
0x0
addi x10,x0,0
bgeu x16,x17,-68
beq x15,x0,-72
lw x5,0(x16)
add x10,x10,5
addi x15,x15,-1
jal x0,-20
0x0
slli x10,x16,3
add x10,x10,16
add x10,x10,16
jal x0,-108
0x0
0x0
0x0
0x0
0x4
0x0
0x0
0x0
0x0
0x0
0x0
0x0
0x0
```


With the same process we disassemble the remote firmware we extracted before:
```shell
python3 main.py d --input firmware.bin
```

And the result is:
```c
addi x5,x0,256
bgeu x17,x5,24
slli x17,x17,5
auipc x5,0
addi x17,x17,16
add x17,x17,5
jalr x0,x17,0
addi x17,x0,0
ecall
0x0
0x0
0x0
0x0
0x0
0x0
bgeu x16,x17,-32
lb x5,0(x16)
0x2800b
addi x16,x16,1
bne x5,x0,-16
jal x0,-52
0x0
0x0
addi x10,x0,0
bgeu x16,x17,-68
beq x15,x0,-72
lw x5,0(x16)
add x10,x10,5
addi x15,x15,-1
jal x0,-20
0x0
slli x10,x16,3
add x10,x10,16
add x10,x10,16
jal x0,-108
0x0
0x0
0x0
0x0
lui x21,1183318016
addi x21,x21,1584
0x8d2a6aab
addi x5,x0,-255
lw x6,0(x5)
lui x5,1598504960
addi x5,x5,-954
addi x10,x0,0
beq x5,x6,4
addi x10,x10,1
lui x21,-624918528
addi x21,x21,-1508
0x8d2a6aab
jal x0,-180
0x0
0x0
0x0
0x0
0x0
0x0
0x0
0x0
... other 0s and non relevant stuff
```



The first thing we can see in both firmwares is that they contain a strange value that is:
`0x2800b`

==**Another thing we can see is that the remote firmware contains some instruction not contained in the public one**==:
```c
lui x21,1183318016
addi x21,x21,1584
0x8d2a6aab
addi x5,x0,-255
lw x6,0(x5)
lui x5,1598504960
addi x5,x5,-954
addi x10,x0,0
beq x5,x6,4
addi x10,x10,1
lui x21,-624918528
addi x21,x21,-1508
0x8d2a6aab
jal x0,-180
```

So what we need to do is to understand what this value is.

Let's try to execute the firmware.


### Firmware execution

==**From the documentation of RISC-V we know that when we call the instruction**== `ecall` ==**and the register**== `a7` ==**has a value greater than 0 then the firmware is executed in supervisor mode.**==

In addition if we try to run the binary file `spoke` with the `--help` we can see that we can also define which firmware we have to use in the spoke execution:
- ![[Pasted image 20240517210457.png]]


So first of all let's create a RISC-V "code" to trigger the firmware execution, and save it in a file called `firmware_execution.txt`:
```C
_start:
    addi a7, x0, 1
    ecall
```
- Let's assemble and convert it in base64
	- the result will be: `kwgQAHMAAAA=`




At this point we have to set the remote firmware into the `spoke` binary, and since we are there we can use also the debug mode to see the execution instruction by instruction:
```shell
./spoke -d -f firmware.bin 
```
- copy the file `firmware.bin` into the `spoke` directory to directly execute it with this command


We put the base64 value calculated before and the result is:
```
pc: 0, all others: 0
Addi { rd: 17, rs1: 0, imm: 1 }
pc: 4, x17: 1, all others: 0
Ecall
pc: ffff1000, x17: 1, all others: 0
Addi { rd: 5, rs1: 0, imm: 256 }
pc: ffff1004, x5: 100, x17: 1, all others: 0
Bgeu { rs1: 17, rs2: 5, imm: 24 }
pc: ffff1008, x5: 100, x17: 1, all others: 0
Slli { rd: 17, rs1: 17, shamt: 5 }
pc: ffff100c, x5: 100, x17: 20, all others: 0
Auipc { rd: 5, imm: 0 }
pc: ffff1010, x5: ffff100c, x17: 20, all others: 0
Addi { rd: 17, rs1: 17, imm: 16 }
pc: ffff1014, x5: ffff100c, x17: 30, all others: 0
Add { rd: 17, rs1: 17, rs2: 5 }
pc: ffff1018, x5: ffff100c, x17: ffff103c, all others: 0
Jalr { rd: 0, rs1: 17, imm: 0 }
pc: ffff103c, x5: ffff100c, x17: ffff103c, all others: 0
Bgeu { rs1: 16, rs2: 17, imm: 4294967264 }
pc: ffff1040, x5: ffff100c, x17: ffff103c, all others: 0
Lb { rd: 5, rs1: 16, imm: 0 }
pc: ffff1044, x5: ffffff93, x17: ffff103c, all others: 0
CustomPrint { rd: 0, rs1: 5, format: 0 }
����pc: ffff1048, x5: ffffff93, x17: ffff103c, all others: 0
Addi { rd: 16, rs1: 16, imm: 1 }
pc: ffff104c, x5: ffffff93, x16: 1, x17: ffff103c, all others: 0
Bne { rs1: 5, rs2: 0, imm: 4294967280 }
pc: ffff103c, x5: ffffff93, x16: 1, x17: ffff103c, all others: 0
Bgeu { rs1: 16, rs2: 17, imm: 4294967264 }
pc: ffff1040, x5: ffffff93, x16: 1, x17: ffff103c, all others: 0
Lb { rd: 5, rs1: 16, imm: 0 }
pc: ffff1044, x5: 8, x16: 1, x17: ffff103c, all others: 0
CustomPrint { rd: 0, rs1: 5, format: 0 }
pc: ffff1048, x5: 8, x16: 1, x17: ffff103c, all others: 0
Addi { rd: 16, rs1: 16, imm: 1 }
pc: ffff104c, x5: 8, x16: 2, x17: ffff103c, all others: 0
Bne { rs1: 5, rs2: 0, imm: 4294967280 }
pc: ffff103c, x5: 8, x16: 2, x17: ffff103c, all others: 0
Bgeu { rs1: 16, rs2: 17, imm: 4294967264 }
pc: ffff1040, x5: 8, x16: 2, x17: ffff103c, all others: 0
Lb { rd: 5, rs1: 16, imm: 0 }
pc: ffff1044, x5: 10, x16: 2, x17: ffff103c, all others: 0
CustomPrint { rd: 0, rs1: 5, format: 0 }
pc: ffff1048, x5: 10, x16: 2, x17: ffff103c, all others: 0
Addi { rd: 16, rs1: 16, imm: 1 }
pc: ffff104c, x5: 10, x16: 3, x17: ffff103c, all others: 0
Bne { rs1: 5, rs2: 0, imm: 4294967280 }
pc: ffff103c, x5: 10, x16: 3, x17: ffff103c, all others: 0
Bgeu { rs1: 16, rs2: 17, imm: 4294967264 }
pc: ffff1040, x5: 10, x16: 3, x17: ffff103c, all others: 0
Lb { rd: 5, rs1: 16, imm: 0 }
pc: ffff1044, x16: 3, x17: ffff103c, all others: 0
CustomPrint { rd: 0, rs1: 5, format: 0 }
pc: ffff1048, x16: 3, x17: ffff103c, all others: 0
Addi { rd: 16, rs1: 16, imm: 1 }
pc: ffff104c, x16: 4, x17: ffff103c, all others: 0
Bne { rs1: 5, rs2: 0, imm: 4294967280 }
pc: ffff1050, x16: 4, x17: ffff103c, all others: 0
Jal { rd: 0, imm: 4294967244 }
pc: ffff101c, x16: 4, x17: ffff103c, all others: 0
Addi { rd: 17, rs1: 0, imm: 0 }
pc: ffff1020, x16: 4, all others: 0
Ecall
pc: 8, x16: 4, all others: 0
Could not decode instruction!
```


#### 0x2800b instruction decoding

From these lines:
```
pc: ffff1018, x5: ffff100c, x17: ffff103c, all others: 0
Jalr { rd: 0, rs1: 17, imm: 0 }
pc: ffff103c, x5: ffff100c, x17: ffff103c, all others: 0
Bgeu { rs1: 16, rs2: 17, imm: 4294967264 }
pc: ffff1040, x5: ffff100c, x17: ffff103c, all others: 0
Lb { rd: 5, rs1: 16, imm: 0 }
pc: ffff1044, x5: ffffff93, x17: ffff103c, all others: 0
CustomPrint { rd: 0, rs1: 5, format: 0 }
```

**==We can easily understand that==** `CustomPrint` **==referes to the==** `0x2800b` value. 

In fact, in the firmware we can see these lines, that represent the ones extracted from the firmware execution:
```
bgeu x16,x17,-32
lb x5,0(x16)
0x2800b
```

So `0x2800b` is a custom instruction that maybe cannot be disassembled by `ace`.

Now we have to understand what `0x8d2a6aab`. So maybe it could be another custom instruction that cannot be disassembled by `ace`.


To understand what this value does we have to force the execution of these lines:
```c
lui x21,1183318016
addi x21,x21,1584
0x8d2a6aab
addi x5,x0,-255
lw x6,0(x5)
lui x5,1598504960
addi x5,x5,-954
addi x10,x0,0
beq x5,x6,4
addi x10,x10,1
lui x21,-624918528
addi x21,x21,-1508
0x8d2a6aab
jal x0,-180
```




## Firmware execution with new remote lines

At this point we have to find a way to execute that instructions.

<mark style="background: #BBFABBA6;">But, if we open the hex strings file that represents the remote firmware and we try to understand which lines represent the new instructions?</mark>

We can do it looking at the disassembled remote firmware and the hex strings of the remote firmware:
![[Pasted image 20240518120618.png]]

So we know that the instructions we need to study are represented with the hex values:
```hex
46880ab7
630a8a93
8d2a6aab
f0100293
0002a303
5f4742b7
c4628293
00000513
00628263
00150513
dac08ab7
a1ca8a93
8d2a6aab
f4dff06f
```



What we need to do is to copy these lines, put them in the top of a new `hex_strings_modified.txt` file, and revert it into a binary file, using the python script as before.
- <mark style="background: #BBFABBA6;">In this way we are reconstructing the firmware, but with the instruction we want to execute at the beginning.</mark>

The new `hex_strings_modified.txt` file will be:
```hex
46880ab7
630a8a93
8d2a6aab
f0100293
0002a303
5f4742b7
c4628293
00000513
00628263
00150513
dac08ab7
a1ca8a93
8d2a6aab
f4dff06f
10000293
0058fc63
00589893
00000297
01088893
005888b3
00088067
00000893
00000073
00000000
00000000
00000000
00000000
00000000
00000000
ff1870e3
00080283
0002800b
00180813
fe0298e3
fcdff06f
00000000
00000000
00000513
fb187ee3
fa078ce3
00082283
00550533
fff78793
fedff06f
00000000
00381513
01050533
01050533
f95ff06f
00000000
00000000
00000000
00000000
46880ab7
630a8a93
8d2a6aab
f0100293
0002a303
5f4742b7
c4628293
00000513
00628263
00150513
dac08ab7
a1ca8a93
8d2a6aab
f4dff06f
00000000
00000000
00000000
```


The script need to be changed, just for the file names:
```python
def hex_to_little_endian(hex_str):
    # this splits the hex string into groups of 2 characters, e.g., 00000093 -> ['00', '00', '00', '93']
    two_chars_split_list = [hex_str[i:i+2] for i in range(0, len(hex_str), 2)]
    
    # this reverses the order of the list we have created, e.g., ['00', '00', '00', '93'] -> ['93', '00', '00', '00']
    little_endian_list = two_chars_split_list[::-1]

    # returns the entire little endian hex: 93000000
    return ''.join(little_endian_list)

def create_binary_file(hex_list, file_name):
    with open(file_name, 'wb') as bin_file:
        for hex_str in hex_list:
            little_endian_hex = hex_to_little_endian(hex_str)
            little_endian_bytes = bytes.fromhex(little_endian_hex)
            bin_file.write(little_endian_bytes)

# file from which to read hex strings
input_file_name = "hex_strings.txt"

# output binary file name
output_file_name = "modified_firmware.bin"

# read hex strings from the file
hex_strings = []
with open(input_file_name, 'r') as file:
    for line in file:
        hex_strings.append(line)

# create the binary file
create_binary_file(hex_strings, output_file_name)
```


So at this point we can execute:
```shell
python3 hex_to_binary_firmware.py
```

Then in the `ace` directory:
```shell
python3 main.py d --input modified_firmware.bin 
```

And the result will be:
```c
lui x21,1183318016
addi x21,x21,1584
0x8d2a6aab
addi x5,x0,-255
lw x6,0(x5)
lui x5,1598504960
addi x5,x5,-954
addi x10,x0,0
beq x5,x6,4
addi x10,x10,1
lui x21,-624918528
addi x21,x21,-1508
0x8d2a6aab
jal x0,-180
addi x5,x0,256
bgeu x17,x5,24
slli x17,x17,5
auipc x5,0
addi x17,x17,16
add x17,x17,5
jalr x0,x17,0
addi x17,x0,0
ecall
0x0
0x0
0x0
0x0
0x0
0x0
bgeu x16,x17,-32
lb x5,0(x16)
0x2800b
addi x16,x16,1
bne x5,x0,-16
jal x0,-52
0x0
0x0
addi x10,x0,0
bgeu x16,x17,-68
beq x15,x0,-72
lw x5,0(x16)
add x10,x10,5
addi x15,x15,-1
jal x0,-20
0x0
slli x10,x16,3
add x10,x10,16
add x10,x10,16
jal x0,-108
0x0
0x0
0x0
0x0
lui x21,1183318016
addi x21,x21,1584
0x8d2a6aab
addi x5,x0,-255
lw x6,0(x5)
lui x5,1598504960
addi x5,x5,-954
addi x10,x0,0
beq x5,x6,4
addi x10,x10,1
lui x21,-624918528
addi x21,x21,-1508
0x8d2a6aab
jal x0,-180
0x0
0x0
0x0
```


The result we want.

### Modified firmware execution

Let's execute another time spoke with debug arg and the new modified firmware:
```shell
./spoke -d -f modified_firmware.bin 
```

We will use the same base64 to trigger the firmware execution `kwgQAHMAAAA=`

The result will be:
```
Please provide your program as a single base64 encoded string terminated by a newline
kwgQAHMAAAA=
pc: 0, all others: 0
Addi { rd: 17, rs1: 0, imm: 1 }
pc: 4, x17: 1, all others: 0
Ecall
pc: ffff1000, x17: 1, all others: 0
Lui { rd: 21, imm: 1183318016 }
pc: ffff1004, x17: 1, x21: 46880000, all others: 0
Addi { rd: 21, rs1: 21, imm: 1584 }
pc: ffff1008, x17: 1, x21: 46880630, all others: 0
X { rd: 21 }
pc: ffff100c, x17: 1, x21: 46880630, all others: 0
Addi { rd: 5, rs1: 0, imm: 3841 }
pc: ffff1010, x5: f01, x17: 1, x21: 46880630, all others: 0
Lw { rd: 6, rs1: 5, imm: 0 }
pc: ffff1014, x5: f01, x17: 1, x21: 46880630, all others: 0
Lui { rd: 5, imm: 1598504960 }
pc: ffff1018, x5: 5f474000, x17: 1, x21: 46880630, all others: 0
Addi { rd: 5, rs1: 5, imm: 3142 }
pc: ffff101c, x5: 5f474c46, x17: 1, x21: 46880630, all others: 0
Addi { rd: 10, rs1: 0, imm: 0 }
pc: ffff1020, x5: 5f474c46, x17: 1, x21: 46880630, all others: 0
Beq { rs1: 5, rs2: 6, imm: 4 }
pc: ffff1024, x5: 5f474c46, x17: 1, x21: 46880630, all others: 0
Addi { rd: 10, rs1: 10, imm: 1 }
pc: ffff1028, x5: 5f474c46, x10: 1, x17: 1, x21: 46880630, all others: 0
Lui { rd: 21, imm: 3670048768 }
pc: ffff102c, x5: 5f474c46, x10: 1, x17: 1, x21: dac08000, all others: 0
Addi { rd: 21, rs1: 21, imm: 2588 }
pc: ffff1030, x5: 5f474c46, x10: 1, x17: 1, x21: dac08a1c, all others: 0
X { rd: 21 }
pc: ffff1034, x5: 5f474c46, x10: 1, x17: 1, x21: dac08a1c, all others: 0
Jal { rd: 0, imm: 4294967116 }
pc: ffff0f80, x5: 5f474c46, x10: 1, x17: 1, x21: dac08a1c, all others: 0
```
- If we study it we can see that the line `0x8d2a6aab` refers to a function called `X` that uses as destination register `x21`


But what the instruction `X` represents?

We can try to convert `0x8d2a6aab` in binary using this [link](https://www.rapidtables.com/convert/number/hex-to-binary.html) to see what it represents, it must be a 32 bit string, and the  last 7 bits must be the operation code...

So maybe we can understand what instruction it is effectively.

The result is:
![[Pasted image 20240518123021.png]]

The last 7 bits are: `0101011`

## 01011011 operation code searching
**==If we open the official documentation of CRISP-V we cannot find an instruction with this operation code. So maybe it is a custom instruction.==**

In fact, if we search for it into the file `ace/instruction.py` we can see that:
![[Pasted image 20240518123320.png]]
- it has the same operation code of the **==custom privilege operation==**


So maybe it is used to change the privileges into the machine ones, we have to try it.



## 0x8d2a6aab operation executing

We know from the documentation of ace, and so from the file `ace/README.md` that:
![[Pasted image 20240518123633.png]]
- we can use directly hex values in our CRISP-V "code"
	- so we can run directly this instruction without decoding it


So what we want to do is to force the execution of `0x8d2a6aab` in order to see if it can be used to gain the machine privileges and so to print the memory non accessible without these privileges.

So we can start to write:
```C
_start:
    li t1, 0xffff0000   
    li t2, 0xffffffff
    lui x21,1183318016
    addi x21,x21,1584
    0x8d2a6aab 

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
    ebreak
```
- in order to see if this approach works
- we use the print with format 0 to print everything in `utf-8`, if the flag is here we can notice easily it
- we avoid here to print 0s, and we use the same approach we used for the flag 2

We store this "code" into a file called `part3.txt`

>NOTE: we used also the instructions `lui x21,1183318016` and `addi x21,x21,1584` because the instruction `0x8d2a6aab` uses the register x21, so it has sense to reproduce everything as it is in the remote firmware to test


So let's use the same approach as before:
```shell
python3 main.py a --input part3.txt | base64
```
- and we obtain `NwP//xMDAwC38///k4Pz/7cKiEaTigpjq2oqjWPAYwIDTwMAYxYPABMDEwBv8B//iwIPABMDEwBv8F/+cwAQAA==`

We use it on the remote server and the result is:
![[Pasted image 20240518124643.png]]

So nothing happens...

==**But maybe, since this instruction is a firmware instruction, we need to gain first the supervisor privileges...**==


So let's try to change the "code" into:
```C
_start:
    li t1, 0xffff0000   
    li t2, 0xffffffff
    privilege
    lui x21,1183318016
    addi x21,x21,1584
    0x8d2a6aab 

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
    ebreak
```
- we added `privilege` before calling the new instructions


The base64 for this new "code" is: `NwP//xMDAwC38///k4Pz/6sgoCS3CohGk4oKY6tqKo1jwGMCA08DAGMWDwATAxMAb/Af/4sCDwATAxMAb/Bf/nMAEAA=`


And booom, it works:
![[Pasted image 20240518125014.png]]

<mark style="background: #BBFABBA6;">The flag for the part 3 is</mark>: `FLG_PT3{chris_domas_h4s_n1c3_t4lk5_r3c0rd3d}`