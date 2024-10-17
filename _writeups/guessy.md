---
title: Hackropole - Guessy
author: Garfield1002
date: 2024-10-1
category: Reverse Hackropole Writeup
layout: post
---

> [challenge link](https://hackropole.fr/fr/challenges/reverse/fcsc2023-reverse-aaarg/)

This is a writeup for a challenge that really needs no writeup.

Instead, I want to take advantage of this simple binary to show you different reverse engineering techniques.

## The program

When we load the program with Ghidra, we get this `main` function (if you can't find main checkout [this tutorial]({% link _reverse/finding_main.md %})) :

```c
int main(int argc,char **argv,char **envp)
{
  int iVar1;
  ulong uVar2;
  char *local_10;

  iVar1 = 1;
  if (1 < argc) {
    uVar2 = strtoul(argv[1],&local_10,10);
    iVar1 = 1;
    if ((*local_10 == '\0') && (iVar1 = 2, uVar2 == (long)-argc)) {
      uVar2 = 0;
      do {
        putc((int)(char)(&DAT_00402010)[uVar2],stdout);
        uVar2 = uVar2 + 4;
      } while (uVar2 < 0x116);
      putc(10,stdout);
      iVar1 = 0;
    }
  }
  return iVar1;
}
```

## The _normal solution_

Let's look at what this function does, it takes the first argument, converts it to a long with `strtoul` and then checks that it is equal to minus the number of arguments.

With those checks past, it prints characters from an address on screen with `putc`.

Let's run it with this knowledge:

```
$ ./aaarg -2
FCSC{...}
```

Welp theres the flag.

## But I don't want to run the binary

We can also look at the bytes being printed in Ghidra, the loop prints every 4th byte (`uVar2 = uVar2 + 4;`). It does so while `uVar2` is lesser than 0x116.

So we can take the 0x116 bytes after `DAT_00402010` and print every 4th byte ourselves.

To do so, I retype (Ctrl+L) `DAT_00402010` to a `char[0x116]`.
Then I copy a python list of it:

![](./image.png)

We can print said list with a simple script:

```py
l = [ 0x46, 0xe2, 0x80, 0x8d, 0x43, 0xe2, 0x80, ... ]
print(''.join([chr(x) for x in l[::4]]))
```

## Could we have done it otherwise ?

> This is overkill, the point is to show off different techniques

If we look at the loop, we realize that it does not depend on the user input..

We can just jump to the printing routine and start the program at `0x004011d5`.

![alt text](./image-1.png)

In GDB we'll break the program at its entrypoint (`0x401050` we know this from ghidra, but in GBB we can also run `info file`), then set the program counter to the desired address and continue execution

```
(gdb) br *0x401050
(gdb) r
(gdb) set $rip=0x004011d5
(gdb) c
Continuing.
FCSC{...}
```
### Why did we break at the entrypoint ?

Why do we need a breakpoint at the binary's entrypoint ? Shouldn't it be the first instruction executed ?

Our binary is dynamically linked:
```
$ file ./aaarg
./aaarg: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=f5b07c01242cc5987bed7730c2762ae0491b5ddc, stripped
```

The code before our entrypoint handles the dynamic linking. If we skip this code, our binary is not going to be able to interact with the libc and `putc` will crash.

## Can we overkill the chall ?

Wouldn't it be nice if there was an automated way to find an input value such that the instruction pointer made its way to a desired address ? Symbolic execution is the way to go.

We will be using `angr` as our symbolic execution engine.

We need to tell `angr` what address we want to get to, in our case its `0x004011d5`.

```py
import angr
import claripy

FIND  = 0x004011d5

# We guess a value
ARG_LEN = 10

p = angr.Project('aaarg')

# We create a Symbolic Bit Vector for our argument
argv1 = claripy.BVS("argv1", ARG_LEN * 8)

# We initialize our engine with our arguments
state = p.factory.entry_state(args=["aaarg", argv1])
ex = p.factory.simgr(state)

# We ask the engine to find an execution path that goes to FIND
# execution stops there so we don't see our flag
ex.explore(find=FIND)

assert len(ex.found) > 0

# Prints the value of argv1
# Since we know it is  a null terminated string, we can 
solution = ex.found[0].solver.eval(argv1, cast_to=bytes)
solution = solution[: solution.find(b"\x00")]
print(solution.decode())

#> -2
```

### Why look at the binary ?

We still had to do a lot of work... Open the binary, find the desired address.. We had to execute the binary with the provided input...

Lets be truly lazy.

We can simply tell `angr` to find an execution path where `FCSC` and `}` appear in stdout:

```py
import angr
import claripy

# We guess a value
ARG_LEN = 10

p = angr.Project('aaarg')

# We create a Symbolic Bit Vector for our argument
argv1 = claripy.BVS("argv1", ARG_LEN * 8)

# We initialize our engine with our arguments
state = p.factory.entry_state(args=["aaarg", argv1])
ex = p.factory.simgr(state)

# We ask the engine to find an execution path where FCSC appears in stdout
ex.explore(find=lambda s: b"FCSC" in s.posix.dumps(1) and b"}" in s.posix.dumps(1))

assert len(ex.found) > 0

# Prints the value of stdout
print(ex.found[0].posix.dumps(1).decode())

#> FCSC {...}
```
