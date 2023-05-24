---
title: "Anatomy of a C++ program"
date: 2023-05-24T16:19:08-04:00
draft: false
---

I wrote this article to summarize my own understanding of how a C++ program is built and executed - from source code to CPU instructions - and how you can use tools to inspect each step in the pipeline. The primary aim is to concisely yet comprehensively explain the necessary steps without going into too much details about what extra things can be done (i.e. optimizations, etc.)

This article assumes basic familiarity with C++.

C++ is a language that requires compilation - that means it requires conversion of it's source code (intended for humans), defined by the standard, to machine code (intended for CPUs). Compilation is done by compilers. Compilation is a multi-step process and involves stages like preprocessing and linking.

Let's write a very simple C++ program that consists of a `program.cpp` that calls a function to add two integers together and returns the result, and a `sum.h` and `sum.cpp` that contain the declaration and definition for the function respectively.

```c++
// sum.h

#pragma once

static int sum(int a, int b) {
  return a + b;
}
```

```c++
// sum.cpp

#include "sum.h"

int sum(int a, int b) {
  return a + b;
}
```


```c++
// program.cpp

#include "sum.h"

int main()
{
  int a = 1;
  int b = 2;
  int result = sum(a , b);  
  return 0;
}
```

This is stage 1 of the process, where you simply write code that conforms to the C++ Application Programming Interface (API).

Next, we employ a compiler to run the following steps:

1. Preprocessing
2. Assembling
3. Linking

### Preprocessing

The preprocessor stage operates on the preprocessor directives that are present in the source code. In the instance above, we used `#include "sum.h"` to ensure that the definition of `sum` is available for use in our function.

To view the output of the preprocessor stage, we can use the following command:

```powershell
cl program.cpp /P
```

This should generate a file called `program.i` - and it will contain the contents of `sum.h` above the source code for the function we wrote above. This file is a valid C++ source file that requires no further preprocessing.

### Compiling

Since all the code is now collected in one file, the next step is to compile it to assembly for the target instruction set architecture.

To view the output of the compiling stage, we can use the following commands:

```powershell
cl program.cpp /FAs /c
```

This should generate a file (among others) called `program.asm` - and it will contain the generated assembly code for the `program.i` file generated in the previous step. This file contains assembly instructions that all you to see exactly what instructions your code will be executing.


### Linking

In C++, to **declare** something is to acknowledge that it exists somewhere. To **define** it is to give actual definition to the declaration. Since the code for a program may be distributed between several files, it is the linkers job to match the usage of functions/variables in one file to their definitions in another file.

This step can be done at compile time, known as **static linking**, or at runtime, known as **dynamic linking**.

#### Static Linking

Static linking will embed all dependencies into a single executable. To link the function definition within `sum.obj` to `program.obj` that calls it and generate an executable, use the following command:

```powershell
link program.obj sum.obj
```

The output will be a `program.exe` executable that contains your working program!

#### Dynamic Linking

Alternatively, we can compile the `sum.obj` to a DLL and then use it when linking against the `program.obj` file.

```powershell
link powershell.obj sum.dll
```

### Conclusion

While the steps above describe the basic workflow required to get a C++ program from source code to executable, there is a lot left out here, especially since each step of the process can be configured to work differently. I plan on making updates to this article for accuracy and as I get to try things out on different compilers and platforms.