# [Program Library HOWTO](http://tldp.org/HOWTO/Program-Library-HOWTO/)

## 1. Introduction

Program libraries can be divided into three types: static libraries, shared libraries, and dynamically loaded (DL) libraries.

**Static libraries** are installed into a program executable before the program can be run.

**Shared libraries** are loaded at program start-up and shared between programs.

**Dynamically loaded (DL) libraries** can be loaded and used at any time while a program is running. DL libraries aren't really a different kind of library format (both static and shared libraries can be used as DL libraries).

Most developers who are developing libraries **should create shared libraries**, since these allow users to update their libraries separately from the applications that use the libraries. **Dynamically loaded (DL) libraries** are useful, but they require a little more work to use and **many programs don't need the flexibility they offer**. Conversely, **static libraries** make upgrading libraries far more troublesome, so for general-purpose use they're **hard to recommend**. Still, each have their advantages.

Some people use the term dynamically linked libraries (DLLs) to refer to shared libraries, some use the term DLL to mean any library that is used as a DL library, and some use the term DLL to mean a library meeting either condition.

If you're building an application that should port to many systems, you might consider using [GNU libtool](http://www.gnu.org/software/libtool/libtool.html) to build and install libraries instead of using the Linux tools directly.

## 2. Static Libraries

Static libraries are simply a collection of ordinary object files; conventionally, static libraries end with the ``.a'' suffix. This collection is created using the `ar` (archiver) program. Static libraries aren't used as often as they once were, because of the advantages of shared libraries (described below). Still, they're sometimes created, they existed first historically, and they're simpler to explain.

Static libraries permit users to link to programs without having to recompile its code, saving recompilation time. Note that recompilation time is less important given today's faster compilers, so this reason is not as strong as it once was. Static libraries are often useful for developers if they wish to permit programmers to link to their library, but don't want to give the library source code (which is an advantage to the library vendor, but obviously not an advantage to the programmer trying to use the library). In theory, code in static ELF libraries that is linked into an executable should run slightly faster (by 1-5%) than a shared library or a dynamically loaded library, but in practice this rarely seems to be the case due to other confounding factors.

To create a static library, or to add additional object files to an existing static library, use a command like this:

    ar rcs my_library.a file1.o file2.o

This sample command adds the object files `file1.o` and `file2.o` to the static library `my_library.a`, creating `my_library.a` if it doesn't already exist.

Be careful about the order of the parameters when using `gcc`; the `-l` option is a linker option, and thus needs to be placed **AFTER** the name of the file to be compiled. This is quite different from the normal option syntax. If you place the `-l` option before the filename, it may fail to link at all, and you can end up with mysterious errors.

You can also use the linker `ld(1)` directly, using its `-l` and `-L` options; however, in most cases it's better to use `gcc(1)` since the interface of `ld(1)` is more likely to change.

## 3. Shared Libraries

Shared libraries are libraries that are **loaded by programs when they start**. When a shared library is installed properly, all programs that start afterwards automatically use the **new** shared library. It's actually much more flexible and sophisticated than this, because the approach used by Linux permits you to: 

* update libraries and still support programs that want to use **older, non-backward-compatible versions** of those libraries;

* **override** specific **libraries** or even specific **functions** in a library when executing a particular program.

* do all this while programs are **running** using existing libraries.

### 3.1. Conventions

For shared libraries to support all of these desired properties, a number of conventions and guidelines must be followed. You need to understand the difference between a library's names, in particular its \`\`**soname**'' and \`\`**real name**'' (and how they interact). You also need to understand where they should be placed in the filesystem.

#### 3.1.1. Shared Library Names

Every shared library has a special name called the \`\`**soname**''. The soname has the prefix \`\`**lib**'', the name of the library, the phrase \`\`**.so**'', followed by a **period** and a **version number** that is incremented whenever the interface changes (as a special exception, the **lowest-level C libraries** don't start with \`\`**lib**''). A **fully-qualified soname** includes as a prefix the directory it's in; on a working system a fully-qualified soname is simply a **symbolic link** to the shared library's \`\`**real name**''.

Every shared library also has a \`\`**real name**'', which is the filename containing the actual library code. The real name **adds to** the soname a **period**, a **minor number**, another **period**, and the **release number**. The **last period** and release number are **optional**. The minor number and release number support **configuration control** by letting you know exactly what version(s) of the library are installed. Note that these numbers might not be the same as the numbers used to describe the library in documentation, although that does make things easier.

In addition, there's the name that the compiler uses when requesting a library, (I'll call it the \`\`**linker name**''), which is simply the soname **without any version number**.

The key to managing shared libraries is the **separation** of these names. Programs, when they **internally** list the shared libraries they need, should only list the **soname** they need. Conversely, when you create a shared library, you only create the library with a specific filename (with more detailed version information). When you **install** a new version of a library, you install it in one of a few special directories and then run the program **ldconfig(8)**. ldconfig examines the existing files and **creates the sonames as symbolic links** to the **real names**, as well as setting up the cache file **/etc/ld.so.cache** (described in a moment).

`ldconfig` doesn't set up the **linker names**; typically this is done during **library installation**, and the **linker name** is simply created as a **symbolic link** to the **\`\`latest'' soname or the latest real name**. I would **recommend** having the **linker name** be a symbolic link to the soname, since in most cases if you update the library you'd like to automatically use it when linking. I asked H. J. Lu why ldconfig doesn't automatically set up the linker names. His explanation was basically that you might want to run code using the latest version of a library, but might instead want development to link against an old (possibly incompatible) library. Therefore, **ldconfig makes no assumptions about what you want programs to link to**, so **installers** must specifically modify symbolic links to update what the **linker** will use for a library.

Thus, `/usr/lib/libreadline.so.3` is a fully-qualified **soname**, which ldconfig would set to be a symbolic link to some **realname** like `/usr/lib/libreadline.so.3.0`. There should also be a **linker name**, `/usr/lib/libreadline.so` which could be a symbolic link referring to `/usr/lib/libreadline.so.3`.

#### 3.1.2. Filesystem Placement

The GNU standards recommend installing by default all libraries in `/usr/local/lib` when distributing source code (and all commands should go into `/usr/local/bin`).

According to the FHS, most libraries should be installed in `/usr/lib`, but libraries required for startup should be in `/lib` and libraries that are not part of the system should be in `/usr/local/lib`.

The **GNU** standards recommend the default for **developers** of source code, while the **FHS** recommends the default for **distributors** (who selectively override the source code defaults, usually via the system's package management system). In practice this works nicely: the \`\`latest'' (possibly buggy!) source code that you download automatically installs itself in the \`\`local'' directory (`/usr/local`), and once that code has matured the package managers can trivially override the default to place the code in the standard place for distributions. Note that if your library calls programs that can only be called via libraries, you should place those programs in `/usr/local/libexe`c (which becomes `/usr/libexec` in a distribution). One complication is that Red Hat-derived systems don't include `/usr/local/lib` by default in their search for libraries; see the discussion below about `/etc/ld.so.conf`. Other standard library locations include `/usr/X11R6/lib` for X-windows. Note that `/lib/security` is used for PAM modules, but those are usually loaded as DL libraries (also discussed below).

### 3.2. How Libraries are Used

On GNU glibc-based systems, including all Linux systems, **starting up** an ELF binary executable automatically **causes** the program **loader** to be loaded and run. On Linux systems, this loader is named `/lib/ld-linux.so.X` (where X is a version number). This loader, in turn, **finds** and **loads** all other shared libraries used by the program.

The list of **directories to be searched** is stored in the file `/etc/ld.so.conf`. Many Red Hat-derived distributions don't normally include `/usr/local/lib` in the file `/etc/ld.so.conf`. I consider this a bug, and adding `/usr/local/lib` to `/etc/ld.so.conf` is a common \`\`fix'' required to run many programs on Red Hat-derived systems.

If you want to just **override a few functions in a library**, but keep the rest of the library, you can enter the names of overriding libraries (`.o` files) in `/etc/ld.so.preload`; these \`\`preloading'' libraries will take precedence over the standard set. This preloading file is typically used for emergency patches; a distribution usually won't include such a file when delivered.

Searching all of these directories at program start-up would be grossly inefficient, so a **caching** arrangement is actually used. The program `ldconfig(8)` by default reads in the file `/etc/ld.so.conf`, sets up the appropriate **symbolic links** in the dynamic link directories (so they'll follow the standard conventions), and then writes a cache to `/etc/ld.so.cache` that's then used by other programs. This greatly speeds up access to libraries. The implication is that `ldconfig` must be run whenever a DLL is **added**, when a DLL is **removed**, or when the set of DLL **directories changes**; running `ldconfig` is often one of the steps performed by package managers when installing a library. On start-up, then, the dynamic loader actually uses the file `/etc/ld.so.cache` and then loads the libraries it needs. 

### 3.3. Environment Variables

#### LD_LIBRARY_PATH
