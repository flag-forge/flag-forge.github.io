---
title: Finding Main
author: Garfield1002
date: 2024-10-17
category: Reverse Ghidra
layout: post
---

## TLDR

`main` is the first function pointer in the call to `__libc_start_main` in the `entry` function.

![](./image-1.png)

## Details

For some reason, Ghidra (Version 11.0) rarely shows me the main function.

Here's a quick tutorial on finding it.

> This tutorial assumes you are reversing a plain, C program with minimal / no obfuscation

Open the `entry` function:

![](./image.png)

It should look something like this:

```c

void processEntry entry(undefined8 param_1,undefined8 param_2)
{
  undefined auStack_8 [8];

  __libc_start_main(FUN_00401190,param_2,&stack0x00000008,FUN_00401220,FUN_00401280,param_1,
                    auStack_8);
  do {
                    /* WARNING: Do nothing block with infinite loop */
  } while( true );
}
```

`__libc_start_main` is defined in `glibc` in `csu/libc-start.c`.

The initial comment states:
> Perform initialization and invoke main.

If we take a look at the prototype of `__libc_start_main` we get this:

```
STATIC int LIBC_START_MAIN (int (*main) (int, char **, char **
					 MAIN_AUXVEC_DECL),
			    int argc,
			    char **argv,
#ifdef LIBC_START_MAIN_AUXVEC_ARG
			    ElfW(auxv_t) *auxvec,
#endif
			    __typeof (main) init,
			    void (*fini) (void),
			    void (*rtld_fini) (void),
			    void *stack_end)
```

The `main` function is therefore going to be the function at the address of the first argument.

In our case the main function is `FUN_00401190`.

While we are at it, we can immediately change its prototype (right click > Edit function signature) to that of main ([source](https://en.cppreference.com/w/c/language/main_function)):

```c
int main(int argc, char** argv, char** envp)
```
