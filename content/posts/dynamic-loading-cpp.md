---
title: "Live Editing/Dynamically Loading C++"
date: 2023-08-14T16:31:31-04:00
draft: false
---

Dynamic loading of code allows for rapid iteration in the development process. The main idea is to separate out part of your code that needs editing from the platform layer that is required to run the application. The platform layer requires pointers to functions, and those can be dynamically loaded. Following is a proof-of-concept for how this can be achieved with C++ on Windows:

```cpp
// foo.cpp

#include <stdio.h>

extern "C" {
	void __declspec(dllexport) foo()
	{
		printf("Foo\n");
	}
}

// Build using the following command:
//
//     cl.exe /Zi foo.cpp /LD
//
// This will generate a foo.dll with debug symbols.
```

- `extern "C"` is used to disable [C++ name mangling](https://learn.microsoft.com/en-us/cpp/build/reference/decorated-names). This is necessary since Windows needs  the name of the function to locate its memory address in the loaded library.
- [`__declspec(dllexport)`](https://learn.microsoft.com/en-us/cpp/build/exporting-from-a-dll-using-declspec-dllexport) is the decorator needed by Windows to determine which functions can be exported.

```cpp
// main.cpp

#include <windows.h>

typedef void(*foo_t)(void);

struct FooLib {
	HINSTANCE library;
	foo_t function;
};

FooLib Load(const char* dllName)
{
	FooLib lib = {};
	HINSTANCE dll = LoadLibrary(dllName);
	if (dll) {
		lib.library = dll;

		foo_t func = (foo_t)GetProcAddress(dll, "foo");
		if (func) {
			lib.function = func;	
		}
	}

	return lib;
}

int main()
{

	while (true) {
		// Copy the DLL generated into another DLL and load that
		// because otherwise the compiler cannot overwrite
		// the DLL since it's being used by the program.
		CopyFile("foo.dll", "foo_.dll", false);
		
		FooLib lib = Load("foo_.dll");
		foo_t func = lib.function();
		
		func();
		
		FreeLibrary(lib.library);
	}
}

// Build using the following command:
//
//     cl.exe /Zi main.cpp
//
// This will generate a main.exe with debug symbols.
```

- We start by defining a type (`foo_t`) for the function.
- Our `Load()` function uses [`LoadLibrary`](https://learn.microsoft.com/en-us/windows/win32/api/libloaderapi/nf-libloaderapi-loadlibraryw) to find and load the DLL , then it uses the [`GetProcAddress`](https://learn.microsoft.com/en-us/windows/win32/api/libloaderapi/nf-libloaderapi-getprocaddress) to find the function with the name `"foo"` inside the library and returns an address to it.
- We can use that function address and use it like any other normal function.
- Finally, we use [`FreeLibrary`](https://learn.microsoft.com/en-us/windows/win32/api/libloaderapi/nf-libloaderapi-freelibrary) to release the DLL.

With the setup shown above, the `foo` function inside `foo.cpp` can be edited while the `main.exe` executable is running, and it will load and execute it dynamically. Note that this is far from an ideal setup since it is loading and freeing the library in every single iteration of the main loop, but that can easily be changed to fit the application's needs.

## Further Reading

- [MSDN article on using run-time dynamic loading](https://learn.microsoft.com/en-us/windows/win32/dlls/using-run-time-dynamic-linking)
- [Handmade Hero - Episode 021 - Loading Game Code Dynamically](https://www.youtube.com/watch?v=WMSBRk5WG58)
- [Tsoding Daily - Hot Code Reloading in C](https://www.youtube.com/watch?v=Y57ruDOwH1g)