---
title: Compiling - Linking
date: 2020-07-12 16:18
summary: Compiling source code does not generate an executable without proper linking.
category: Notes
visible: True
tags: compiler
---

Linking refers to the process of creating an executable file from multiple object files. 
During compiling, the compiler only look at one file at a time and assumes that the symbol is defined elsewhere if it is not in the current file.
The linker, on the other hand, will try to look at other files to identify the symbol.

![linking]({static}/images/linking.JPG)  
Figure 1. Linking

Linking can be performed at:

- Compile time, when the source code is translated into machine code.
- load time, when the program is loaded into memory and executed by loader.
- run time, by application programs.

## Static linking

_static linkers_ performs two main tasks:

- _Symbol resolution_, to associate each symbol reference with exactly one symbol definition.
- _Relocation_, code and data sections generated by compilers and assemblers start at address 0. The _linker_ associates _relocates_ these sections by associating a memory location with each symbol definition, and modifying all the references to these symbols accordingly.


### Object files

Object files come in three forms:

- _Relocatable object files_, 
- _Executable object files_, binary code and data that can be copied directly into memory and executed.
- _Shared object files_, a special type of relocatable object file that can be loaded into memory and linked dynamically, at either load time of run time.

#### Relocatable object files (ELF)
The _ELF header_ begins with a 16-byte sequence describe the word size and byte ordering of the system. Followed by the header are:

+ `.text`, machine code
+ `.rodata`, read-only data (constant strings, jump tables, etc.)
+ `.data`, global initialized variable (no local variables).
+ `.bss`, uninitialized global variables, it occupies no actual space in the object file as it is merely a place holder. 
+ `.symtab`, _symbol table_ with information about global variables and functions that are defined or referenced in the program.
- `.rel.text`, a list of locations in the `.text` section that will need to be modified when the linker combines the object file with others.
+ `.rel.data`, relocation information for any global variables that are defined or referenced by the module.
+ check more [here](http://csapp.cs.cmu.edu/2e/ch7-preview.pdf) section 7.4.

#### Symbol and Symbol tables

Each object module has a symbol table about the symbols that are referenced or defined by it. In the context of a linker, there are three types of symbols:

+ _Global symbols_ defined by the module
+ _Global symbols_ defined by other modules and reference by the current module.
+ _local symbols_ exclusively used by the current module. Basically variables and functions defined with `static` attribute.

#### Symbol Resolution

The linker resolves symbol references by associating each reference with exactly one symbol definition from the symbol table of `ELF` files. The compiler also ensure that the static local variables have unique names.

#### Resolve Multiply defined symbols

At compile time, symbols will be classified as 

+ _strong_ if they are initialized global variables or functions.
+ _weak_ if they are uninitialized global variables.

Rules to determine which symbol to use:

1. Multiple strong symbols are not allowed.
2. Given a strong one and multiple weak one, choose the strong one.
3. Given multiple weak symbols, choose any of them.

### Linking with static libraries (`.a` archive files)

_Static library_ is a single file packed with object modules. During linking, the linker copies only referenced modules from the static library to the program.

Linker's algorithm for resolving external references:

1. scan `.o` and `.a` files in the command line order.
2. keep a list of the current unresolved references during the scan.
3. Resolve each unsolved reference in the list against symbols defined in _obj_ of `.o` and `.a` files.
4. report an error if still any unresolved references.

### Cons of static library

+ Duplication in the stored executables
+ Duplication in the running executables
+ Even minor bug fixes require re-link.

## Shared library (DLL, `.so`)

_Shared libraries_ are object files that contain code and data that are loaded and linked into an application __dynamincally__ at either __load-time__ or __run-time__.

![dynamic linking]({static}/images/dynamic_linking_load_time.JPG)  
Figure 2. Dynamic linking

### Path for searching shared library

1. Path specified when compiling object file.
2. Path in `LD_LIBRARY_PATH`.
3. `/etc/ld.so.conf`
4. default path `/lib`, `/usr/lib`.

`ldconfig` should be executed if a configuration file is updated to load the configurations.


## Comparison

### share library can help better sharing resources between processes
As pointed out before, when using static libraries, every program will copy the function it needs from a static library to its code segment, leading to duplication in the stored executables. When using shared libraries, the programs can share the function loaded into the memory to avoid multiple load. 

### Shared library simplifies code updates
There is no need to re-compile the program if one of the library gets updated. Just re-compile the shared library and the update will take effect when the library is loaded.

### Flexibility to load library at run time
Programmers are able to determine which library to load at run time offering more flexibilities and saving the overhead of loading all functions. For example, programmer can specify which library to load according to the type of the file opened.

### Speed
Due to dynamic loading, using shared library is slightly lower than static library. However, the difference is not very significantly, so the shared library is quite widely used.



# References
[_Computer Systems: A Programmer's Perspective - Chapter 7_](http://csapp.cs.cmu.edu/2e/ch7-preview.pdf)  
[_15-213: Intro to Computer Systems: Schedule for Fall 2015 - Lecture 13 Linking_](http://www.cs.cmu.edu/afs/cs/academic/class/15213-f15/www/lectures/13-linking.pdf)  
[_后台开发：核心技术与应用实践_](https://m.douban.com/book/subject/26850616/)