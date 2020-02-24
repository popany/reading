# [Symbols for Windows debugging (WinDbg, KD, CDB, NTSD)](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/symbols)

Symbols for the Windows debuggers (WinDbg, KD, CDB, and NTSD) are available from a public symbol server.

## 1 Introduction to Symbols

### 1.1 [Symbol path for Windows debuggers](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/symbol-path)

The symbol path specifies locations where the Windows debuggers (WinDbg, KD, CDB, NTST) look for symbol files. For more information about symbols and symbol files, see Symbols.

Some compilers (such as Microsoft Visual Studio) put symbol files in the **same directory** as the binary files. The symbol files and the checked binary files contain **path and file name information**. This information frequently enables the debugger to **find the symbol files automatically**. If you are debugging a user-mode process on the computer where the executable was built, and if the symbol files are still in their original location, the debugger can locate the symbol files without you setting the symbol path.

In most other situations, you have to set the symbol path to point to your symbol file locations.

#### Symbol path syntax

The debugger's symbol path is a string that consists of multiple directory paths, separated by semicolons.

Relative paths are supported. However, unless you always start the debugger from the same directory, you should add a drive letter or a network share before each path. Network shares are also supported.

For each directory in the symbol path, the debugger looks in **three directories**. For example, if the symbol path includes the `c:\MyDir` directory, and the debugger is looking for symbol information for a DLL, the debugger first looks in `c:\MyDir\symbols\dll`, then in `c:\MyDir\dll`, and finally in `c:\MyDir`. The debugger then repeats this process for each directory in the symbol path. Finally, the debugger looks in the current directory and then in the current directory with `..\dll` appended to it. (The debugger appends `..\dll` , `..\exe` , or `..\sys` , depending on which binaries it is debugging.)

Symbol files have **date and time stamps**. You do not have to worry that the debugger will use the wrong symbols that it may find first in this sequence. It always looks for the symbols that **match the time stamp** on the binary files that it is debugging. For more information about responses when symbols files are not available, see [Compensating for Symbol-Matching Problems](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/matching-symbol-names).

One way to set the symbol path is by entering the [`.sympath`](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/-sympath--set-symbol-path-) command. For other ways to set the symbol path, see Controlling the Symbol Path later in this topic.

#### Caching symbols locally

We strongly recommend that you always cache your symbols locally. One way to cache symbols locally is to include `cache*`; or `cache*localsymbolcache;*` in your symbol path.

If you include the string `cache*`; in your symbol path, symbols loaded from any element that appears to the right of this string are stored in the default symbol cache directory on the local computer. For example, the following command tells the debugger to get symbols from the network share `\\someshare` and cache the symbols in the **default location** on the local computer.

    .sympath cache*;\\someshare

If you include the string `cache*localsymbolcache`; in your symbol path, symbols loaded from any element that appears to the right of this string are stored in the `localsymbolcache` directory.

For example, the following command tells the debugger to obtain symbols from the network share `\\someshare` and cache the symbols in the `c:\MySymbols` directory.

    .sympath cache*c:\MySymbols;\\someshare

#### Using a symbol server

If you are connected to the Internet or a corporate network, the most efficient way to access symbols is to use a symbol server. You can use a symbol server by using the `srv*`, `srv*symbolstore`, or `srv*localsymbolcache*symbolstore` string in your symbol path.

If you include the string `srv*` in your symbol path, the debugger uses a symbol server to get symbols from the **default symbol store**. For example, the following command tells the debugger to use a symbol server to get symbols from the default symbol store. These symbols are **not cached** on the local computer.

    .sympath srv*

If you include the string `srv*symbolstore` in your symbol path, the debugger uses a symbol server to get symbols from the symbolstore store. For example, the following command tells the debugger to use a symbol server to get symbols from the symbol store at `https://msdl.microsoft.com/download/symbols`. These symbols are **not cached** on the local computer.

    .sympath srv*https://msdl.microsoft.com/download/symbols

If you include the string `srv*localcache*symbolstore` in your symbol path, the debugger uses a symbol server to get symbols from the `symbolstore` store and **caches** them in the `localcache` directory. For example, the following command tells the debugger to use a symbol server to get symbols from the symbol store at `https://msdl.microsoft.com/download/symbols` and cache the symbols in `c:\MyServerSymbols`.

    .sympath srv*c:\MyServerSymbols*https://msdl.microsoft.com/download/symbols

**If you have a directory on your computer where you manually place symbols, do not use that directory as the cache for symbols obtained from a symbol server.** Instead, use two separate directories. For example, you can manually place symbols in `c:\MyRegularSymbols` and then designate `c:\MyServerSymbols` as a cache for symbols obtained from a server. The following example shows how to specify both directories in your symbol path.

    .sympath c:\MyRegularSymbols;srv*c:\MyServerSymbols*https://msdl.microsoft.com/download/symbols

For more information about symbol servers, see [Symbol Stores and Symbol Servers](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/symbol-stores-and-symbol-servers).

#### Combining cache* and srv*

If you include the string `cache*`; in your symbol path, symbols loaded from any element that appears to the right of this string are stored in the default symbol cache directory on the local computer. For example, the following command tells the debugger to use a symbol server to get symbols from the store at `https://msdl.microsoft.com/download/symbols` and cache them in the default symbol cache directory.

    .sympath cache*;srv*https://msdl.microsoft.com/download/symbols

If you include the string `cache*localsymbolcache`; in your symbol path, symbols loaded from any element that appears to the right of this string are stored in the `localsymbolcache` directory.

For example, the following command tells the debugger to use a symbol server to get symbols from the store at `https://msdl.microsoft.com/download/symbols` and cache the symbols in the `c:\MySymbols` directory.

    .sympath cache*c:\MySymbols;srv*https://msdl.microsoft.com/download/symbols

#### Using AgeStore to reduce the cache size

#### Lazy symbol loading

#### Azure DevOps Services Artifacts

### 1.2 [Symbols and Symbol Files](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/symbols-and-symbol-files)

When applications, libraries, drivers, or operating systems are linked, the **linker** that creates the `.exe` and `.dll` files also creates a number of additional files known as `symbol files`.

Symbol files hold a variety of data which are not actually needed when running the binaries, but which could be very useful in the debugging process.

Typically, symbol files might contain:

* Global variables
* Local variables
* Function names and the addresses of their entry points
* Frame pointer omission (FPO) records
* Source-line numbers

Each of these items is called, individually, a symbol. For example, a single symbol file `Myprogram.pdb` might contain several hundred symbols, including global variables and function names and hundreds of local variables. Often, software companies release two versions of each symbol file: a **full symbol** file containing both public symbols and private symbols, and a **reduced (stripped) file** containing only public symbols. For details, see [Public and Private Symbols](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/public-and-private-symbols).

When debugging, you must make sure that the debugger can access the **symbol files that are associated with the target** you are debugging. Both live debugging and debugging crash dump files require symbols. You must obtain the proper symbols for the code that you wish to debug, and load these symbols into the debugger.

#### Windows Symbols

Windows keeps its symbols in files with the extension `.pdb`.

The compiler and the linker control the symbol format. The Visual C++ linker, places all symbols into `.pdb` files.

The Windows operating system was built in two versions. The **free build** (or **retail build**) has relatively small binaries, and the **checked build** (or **debug build**) has larger binaries, with more debugging symbols in the code itself. Checked builds were available on older versions of Windows before Windows 10, version `1803`. Each of these builds had its own symbol files. When debugging a target on Windows, you must use the symbol files that **match the build** of Windows on the target.

### 1.3 [Public and Private Symbols](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/public-and-private-symbols)

When a full-sized `.pdb` or `.dbg` symbol file is built by a linker, it contains two distinct collections of information: the **private symbol data** and a **public symbol table**. These collections differ in the list of items they contain and the information they store about each item.

The private symbol data includes the following items:

* Functions
* Global variables
* Local variables
* Information about user-defined structures, classes, and data types
* The name of the source file and the line number in that file corresponding to each binary instruction

The public symbol table contains fewer items:

* Functions (except for functions declared static)
* Global variables specified as extern (and any other global variables visible across multiple object files)

As a general rule, the public symbol table contains exactly those items that are accessible from one source file to another. Items visible in only one object file--such as static functions, variables that are global only within a single source file, and local variables--are not included in the public symbol table.

#### Full Symbol Files and Stripped Symbol Files

A full symbol file contains both the private symbol data and the public symbol table. This kind of file is sometimes referred to as a private symbol file, but this name is misleading, for such a file contains both private and public symbols.

A stripped symbol file is a smaller file that contains only the public symbol table - or, in some cases, only a subset of the public symbol table. This file is sometimes referred to as a public symbol file.

#### Creating Full and Stripped Symbol Files

#### Viewing Public and Private Symbols in the Debugger

## 2 Accessing Symbols for Debugging

### 2.1 [Accessing Symbols for Debugging](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/accessing-symbols-for-debugging)

### 2.2 [Installing Windows Symbol Files](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/installing-windows-symbol-files)

### 2.3 Symbol Stores and Symbol Servers

### 2.4 [Deferred Symbol Loading](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/deferred-symbol-loading)

### 2.5 [Avoiding debugger searches for un-needed symbols](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/avoiding-debugger-searches-for-unneeded-symbols)

## 3 SymStore

## 4 [How the Debugger Recognizes Symbols](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/how-the-debugger-recognizes-symbols)

## 5 [Symbol Problems While Debugging](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/symbol-problems-while-debugging)

**Invalid** or **missing** symbols are one of the most common causes of debugger problems. When you see some sort of problem, you need to find out if you have a symbol issue.

In some cases, the solution involves acquiring the correct symbol files. In other cases, you simply need to reconfigure the debugger to recognize symbol files you already have. But if you are not able to get the correct symbol files, you will need to **work around this problem** and debug the target in a more limited manner.

### 5.1 [Verifying Symbols](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/verifying-symbols)

Symbol problems can show up in a variety of ways. Perhaps a **stack trace shows incorrect information** or **fails to identify the names of the functions** in the stack. Or perhaps a debugger command **failed to understand the name of** a module, function, variable, structure, or data type.

If you suspect that the debugger is not loading symbols correctly, there are several steps you can take to **investigate this problem**.

First, use the [`lm` (List Loaded Modules)](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/lm--list-loaded-modules-) command to display the list of loaded modules with symbol information. The most useful form of this command is the following:

    0:000> lml 

Pay particular attention to any notes or abbreviations you may see in these displays. For an interpretation of these, see [Symbol Status Abbreviations](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/symbol-status-abbreviations).

If you don't see the proper symbol files, the first thing to do is to check the symbol path:

    0:000> .sympath

If your symbol path is wrong, fix it. If you are using the kernel debugger make sure your local `%WINDIR%` is not on your symbol path.

Then reload symbols using the [`.reload` (Reload Module)](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/-reload--reload-module-) command:

    0:000> .reload ModuleName 

If your symbol path is correct, you should activate noisy mode so you can see which symbol files dbghelp is loading. Then reload your module. See [Setting Symbol Options](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/symbol-options) for information about how to activate noisy mode.
Here is an example of a "noisy" reload of the Microsoft Windows symbols:

    kd> !sym noisy
    kd> .reload nt
    1: Kernel Version 2081 MP Checked
    2: Kernel base = 0x80400000 PsLoadedModuleList = 0x80506fa0
    3: DBGHELP: FindExecutableImageEx-> Looking for D:\MyInstallation\i386\ntkrnlmp.exe...mismatched timestamp
    4: DBGHELP: No image file available for ntkrnlmp.exe
    5: DBGHELP: FindDebugInfoFileEx-> Looking for
    6: d:\MyInstallation\i386\symbols\retail\symbols\exe\ntkrnlmp.dbg... no file
    7: DBGHELP: FindDebugInfoFileEx-> Looking for
    8: d:\MyInstallation\i386\symbols\retail\symbols\exe\ntkrnlmp.pdb... no file
    9: DBGHELP: FindDebugInfoFileEx-> Looking for d:\MyInstallation\i386\symbols\retail\exe\ntkrnlmp.dbg... OK
    10: DBGHELP: LocatePDB-> Looking for d:\MyInstallation\i386\symbols\retail\exe\ntkrnlmp.pdb... OK
    11: *** WARNING: symbols checksum and timestamp is wrong 0x0036a4ea 0x00361a83 for ntkrnlmp.exe

The symbol handler first looks for an image that matches the module it is trying to load (lines three and four). The image itself is not always necessary, but if an incorrect one is present, the symbol handler will often fail. These lines show that the debugger found an image at D:\MyInstallation\i386\ntkrnlmp.exe, but the time-date stamp didn't match. Because the time-date stamp didn't match, the search continues. Next, the debugger looks for a .dbg file and a .pdb file that match the loaded image. These are on lines 6 through 10. Line 11 indicates that even though symbols were loaded, the time-date stamp for the image did not match (that is, the symbols were wrong).

If the symbol-search encountered a catastrophic failure, you would see a message of the form:

    ImgHlpFindDebugInfo(00000000, module.dll, c:\MyDir;c:\SomeDir, 0823345, 0) failed

This could be caused by items such as file system failures, network errors, and corrupt .dbg files.

#### Diagnosing Symbol Loading Errors

When in noisy mode, the debugger may print out error codes when it cannot load a symbol file. The error codes for `.dbg` files are listed in `winerror.h`. The `.pdb` error codes come from another source and the most common errors are printed in plain English text.

Some common error codes for `.dbg` files from `winerror.h` are:

    0xB
    ERROR_BAD_FORMAT

    0x3
    ERROR_PATH_NOT_FOUND

    0x35
    ERROR_BAD_NETPATH

It's possible that the symbol file cannot be loaded because of a networking error. If you see `ERROR_BAD_FORMAT` or `ERROR_BAD_NETPATH` and you are loading symbols from another machine on the network, try copying the symbol file to your host computer and put its path in your symbol path. Then try to reload the symbols.

#### Verifying Your Search Path and Symbols

Let `"c:\MyDir;c:\SomeDir"` represent your symbol path. Where should you look for debug information?
In cases where the binary has been stripped of debug information, such as the **free builds** of Windows, first look for a `.dbg` file in the following locations:

    c:\MyDir\symbols\exe\ntoskrnl.dbg
    c:\SomeDir\symbols\exe\ntoskrnl.dbg
    c:\MyDir\exe\ntoskrnl.dbg
    c:\SomeDir\exe\ntoskrnl.dbg
    c:\MyDir\ntoskrnl.dbg
    c:\SomeDir\ntoskrnl.dbg
    current-working-directory\ntoskrnl.dbg

Next, look for a `.pdb` file in the following locations:

    c:\MyDir\symbols\exe\ntoskrnl.pdb
    c:\MyDir\exe\ntoskrnl.pdb
    c:\MyDir\ntoskrnl.pdb
    c:\SomeDir\symbols\exe\ntoskrnl.pdb
    c:\SomeDir\exe\ntoskrnl.pdb
    c:\SomeDir\ntoskrnl.pdb
    current-working-directory\ntoskrnl.pdb

Note that in the search for the `.dbg` file, the debugger interleaves searching through the `MyDir` and `SomeDir` directories, but in the `.pdb` search it does not.

Windows XP and later versions of Windows do not use any `.dbg` symbol files. See Symbols and Symbol Files for details.

#### Mismatched Builds

One of the most common problems in debugging failures on a machine that is often updated is **mismatched symbols from different builds**. Three common causes of this problem are: **pointing at symbols for the wrong build**, using a **privately built binary without the corresponding symbols**, and using the **uniprocessor hardware abstraction level (HAL) and kernel symbols on a multiprocessor machine**. The first two are simply a matter of matching your binaries and symbols; the third can be corrected by renaming your hal*.dbg and ntkrnlmp.dbg to hal.dbg and ntoskrnl.dbg.

To find out what build of Windows is installed on the target computer, use the [`vertarget` (Show Target Computer Version)](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/vertarget--show-target-computer-version-) command:
dbgcmd

    kd> vertarget 
    Windows XP Kernel Version 2505 UP Free x86 compatible
    Built by: 2505.main.010626-1514
    Kernel base = 0x804d0000 PsLoadedModuleList = 0x80548748
    Debug session time: Mon Jul 02 14:41:11 2001
    System Uptime: 0 days 0:04:53 

#### Testing the Symbols

#### Useful Commands and Extensions

#### Network and Port Problems

#### Questions and Misconceptions

### 5.2 [Matching Symbol Names](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/matching-symbol-names)

### 5.3 [Matching Symbol Names](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/matching-symbol-names)

### 5.4 [Reading Symbols from Paged-Out Headers](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/reading-symbols-from-paged-out-headers)

### 5.5 [Mapping Symbols When the PEB is Paged Out](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/mapping-symbols-when-the-peb-is-paged-out)

### 5.6 [Debugging User-Mode Processes Without Symbols](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/debugging-user-mode-processes-without-symbols)

### 5.7 [Debugging Performance-Optimized Code](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/debugging-performance-optimized-code)

### 5.8 [Offline Symbols for Windows Update](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/symbols-windows-update)










