---
title: Calling a C function from within a binary on OS X/iOS
tags: c, osx, code injection, binary, hopper
description: code injection with function pointers
---

*If you like this and other posts of mine, then reach out to me at
edgar.factorial@gmail.com, I'm looking for remote contract work.*

Let's say we have a C binary given to us and there's a C function in
it that we want to call. For example, say we have this code, assume it
is called `inject_me.c`:

```c
#include <stdio.h>

int test_function(char *some_word)
{
  return printf("Did say: %s\n", some_word);
}

int main (int argc, char **argv)
{
  return test_function("Hello World");
}
```

and we want to call the function `test_function` by ourselves. To do
that we need to find the location of the function in the binary. We
can do that by disassembling the binary, I used to use command line
tools for that but now I'm using `hopper`. (Hopper is an AMAZING
program). Once we disassemble it, we find the function in memory: 

![](/images/found_test_function.png)

Notice that I highlighted the actual location of: `0x100000f20`. Now
we can inject it with this code, assume it is called `inject.c`

```c
#include <stdio.h>
#include <string.h>
#include <mach-o/dyld.h>

#define SPOT 0x100000f20

typedef int pull_it(char *);

static pull_it *pulled = NULL;

__attribute__((constructor))
void example_injection()
{
  char path[1024];
  uint32_t size = sizeof(path);
  _NSGetExecutablePath(path, &size);
  
  for (uint32_t i = 0; i < _dyld_image_count(); i++) {
    if (strcmp(_dyld_get_image_name(i), path) == 0) {
      intptr_t slide = _dyld_get_image_vmaddr_slide(i);

      pulled = (pull_it*)(intptr_t)(slide + SPOT);
      printf("slide: %lu\n", slide);
      pulled("Please work\n");
    }
  }

  printf("This ran before the actual program\n");
}
```
1. The `__attribute__((constructor))` calls the wrapped function
   before `main` goes off.

2. The slide is needed because: (I found this on some Apple mailing
   list and don't remember from where anymore)

```
Q: Can someone enlighten me as to what the virtual memory slide amount
is ? (this parameter is returned by _dyld_get_image_vmaddr_slide for
instance)

A: The shared libraries are prebound to an initial base address but
when the shared library gets loaded dyld can "slide" the library to a
new base address, this is where the virtual memory slide comes from.
```

Build both with this `Makefile`

```Makefile
osx_clang := $(shell xcrun --sdk macosx --find clang)
osx_sdk := $(shell xcrun --sdk macosx --show-sdk-path)
c_flags := -std=c11

inject_me:inject_me.c
	@${osx_clang} -isysroot ${osx_sdk} ${c_flags} $< -o $@

code_injection:inject_me
	@${osx_clang} -isysroot ${osx_sdk} ${c_flags} inject.c -dynamiclib -o $@.dylib
```

Now we can invoke it like so:

```shell
$ make code_injection
$ DYLD_INSERT_LIBRARIES=code_injection.dylib ./inject_me
slide: 125042688
Did say: Please work

This ran before the actual program
Did say: Hello World
```

Yay, it works. BTW, this works for jailbroken iOS as well.
