# Lab: Understanding Compilation, Object Files, and Linking in C

## 🎯 Objective

Demonstrate how a C source file is transformed into machine code through:

- Compilation (creating object files)
- Inspection of symbols and low-level code
- Linking into an executable

## 🧰 Requirements

- Linux environment (Kali, Ubuntu, WSL, etc.)
- Tools: `gcc`, `file`, `nm`, `objdump`, `strings`, `hexdump`

## Install GCC (If Not Installed)

First, check if GCC (GNU Compiler Collection) is installed by running:

```bash
gcc --version
```

```output
gcc (Debian 14.2.0-12) 14.2.0
Copyright (C) 2024 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
```

If GCC is not installed, install it with:

```bash
sudo apt update && sudo apt install gcc -y
```

---

## 1. Write a Simple C Program

Use any text editor to create a C file. For example, using **nano**:

```bash
nano hello.c
```

Then, add the following simple C program:

```c
#include <stdio.h>

void greet() {
    printf("Hello from the greet function!\n");
}

int main() {
    greet();
    return 0;
}
```

Save the file with **CTRL + X**, then **Y**, and press **Enter**.

## 2. Compile into Object File

Compile the source into an object file (machine code with symbols):

- Produces hello.o
- Not executable
- Has unresolved references (e.g. to puts)
- readelf -h hello.o → Type: REL (Relocatable file)
- It contains machine code and data, but still with unresolved symbols and relocation entries.
- It cannot be executed directly. Instead, it must be linked with other object files and libraries by the linker

Run the following command to compile the program:

```bash
gcc -c hello.c -o hello.o
```

- `gcc` → Calls the compiler.
- `hello.c` → The C source file.
- `-o hello` → Output file named `hello` (a non-executable object file.).

```bash
readelf -h hello.o
```

```output
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00
  Class:                             ELF64
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              REL (Relocatable file)
  Machine:                           Advanced Micro Devices X86-64
  Version:                           0x1
  Entry point address:               0x0
  Start of program headers:          0 (bytes into file)
  Start of section headers:          688 (bytes into file)
  Flags:                             0x0
  Size of this header:               64 (bytes)
  Size of program headers:           0 (bytes)
  Number of program headers:         0
  Size of section headers:           64 (bytes)
  Number of section headers:         13
  Section header string table index: 12
```

```bash
 file hello.o
```

```output
hello.o: ELF 64-bit LSB relocatable, x86-64, version 1 (SYSV), not stripped
```

```bash
./hello.o    # error: not executable
```

```output
zsh: exec format error: ./hello.o
```

## 3. Compile the object file into an executable

Compile the object file into an executable:

- Produces hello (ELF executable)
- Contains all actual memory addresses and a start point
- readelf -h hello → Type: EXEC (Executable file)

```bash
gcc hello.o -o hello
```

or

Compile the source file directly into an executable:

```bash
gcc hello.c -o hello
```

- `hello.o` → The object file.
- `-o hello` → Output file named `hello` (an executable).

```bash
readelf -h hello
```

```output
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00
  Class:                             ELF64
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              DYN (Position-Independent Executable file)
  Machine:                           Advanced Micro Devices X86-64
  Version:                           0x1
  Entry point address:               0x1050
  Start of program headers:          64 (bytes into file)
  Start of section headers:          14000 (bytes into file)
  Flags:                             0x0
  Size of this header:               64 (bytes)
  Size of program headers:           56 (bytes)
  Number of program headers:         14
  Size of section headers:           64 (bytes)
  Number of section headers:         31
  Section header string table index: 30
```

Execute the compiled file with:

```bash
./hello
```

Expected output:

``` output
Hello from the greet function!
```

## 4. Running Without `./` (Optional)

To run the compiled file without `./`, move it to `/usr/local/bin`:

```bash
sudo mv hello /usr/local/bin/
```

Now you can run it with:

```bash
hello
```

## Cross-Compiling (Linux to Windows)

To compile a Windows PE executable on Linux, install mingw-w64:

```bash
sudo apt install mingw-w64 -y  # For 64-bit
```

```bash
sudo apt install gcc-multilib libc6-dev-i386 -y  # For 32-bit
```

- gcc-multilib → Enables 32-bit compilation support.
- libc6-dev-i386 → Installs 32-bit development libraries.

Then, compile the program 64-bit Windows:

```bash
x86_64-w64-mingw32-gcc hello.c -o hello_x64.exe
```

Compile for 32-bit Windows:

```bash
i686-w64-mingw32-gcc hello.c -o hello_x86.exe
```

## check the file type of the compiled binary

```bash
file hello_x64.exe
```

```output
hello_x64.exe: PE32+ executable (console) x86-64, for MS Windows, 19 sections
```

```bash
xxd hello_x64.exe | head
```

```output
00000000: 4d5a 9000 0300 0000 0400 0000 ffff 0000  MZ..............
00000010: b800 0000 0000 0000 4000 0000 0000 0000  ........@.......
00000020: 0000 0000 0000 0000 0000 0000 0000 0000  ................
00000030: 0000 0000 0000 0000 0000 0000 8000 0000  ................
00000040: 0e1f ba0e 00b4 09cd 21b8 014c cd21 5468  ........!..L.!Th
00000050: 6973 2070 726f 6772 616d 2063 616e 6e6f  is program canno
00000060: 7420 6265 2072 756e 2069 6e20 444f 5320  t be run in DOS
00000070: 6d6f 6465 2e0d 0d0a 2400 0000 0000 0000  mode....$.......
00000080: 5045 0000 6486 1300 42b5 de67 001c 0300  PE..d...B..g....
00000090: 4d07 0000 f000 2600 0b02 022b 006e 0000  M.....&....+.n..
```

```bash
file hello_x86.exe
```

Expected output:

``` output
hello_x86.exe: PE32 executable (console) Intel 80386, for MS Windows
```

```bash
file hello_x86.exe

```output
hello_x86.exe: PE32 executable (console) Intel 80386, for MS Windows, 17 sections
```

```bash
xxd hello_x86.exe| head
```

```output
00000000: 4d5a 9000 0300 0000 0400 0000 ffff 0000  MZ..............
00000010: b800 0000 0000 0000 4000 0000 0000 0000  ........@.......
00000020: 0000 0000 0000 0000 0000 0000 0000 0000  ................
00000030: 0000 0000 0000 0000 0000 0000 8000 0000  ................
00000040: 0e1f ba0e 00b4 09cd 21b8 014c cd21 5468  ........!..L.!Th
00000050: 6973 2070 726f 6772 616d 2063 616e 6e6f  is program canno
00000060: 7420 6265 2072 756e 2069 6e20 444f 5320  t be run in DOS
00000070: 6d6f 6465 2e0d 0d0a 2400 0000 0000 0000  mode....$.......
00000080: 5045 0000 4c01 1100 c9b5 de67 00f4 0200  PE..L......g....
00000090: d306 0000 e000 0601 0b01 022b 0072 0000  ...........+.r..
```

## Summary

| Target                  | Command                                         |
| :---------------------- | :---------------------------------------------- |
| Linux x86_64 (64-bit)   | gcc hello.c -o hello_x64                        |
| Linux x86 (32-bit)      | gcc -m32 hello.c -o hello_x86                   |
| Windows x86_64 (64-bit) | x86_64-w64-mingw32-gcc hello.c -o hello_x64.exe |
| Windows x86 (32-bit)    | i686-w64-mingw32-gcc hello.c -o hello_x86.exe   |
