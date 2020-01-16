# cmake documentation

## [cmake-buildsystem(7)](https://cmake.org/cmake/help/v3.16/manual/cmake-buildsystem.7.html)

### Introduction

A CMake-based buildsystem is organized as a set of high-level **logical targets**. Each target corresponds to an **executable** or **library**, or is a custom target containing **custom commands**. **Dependencies** between the targets are expressed in the buildsystem to determine the **build order** and the rules for regeneration in response to change.  

### Binary Targets

**Executables** and **libraries** are defined using the `add_executable()` and `add_library()` commands. The resulting binary files have appropriate `PREFIX`, `SUFFIX` and extensions for the platform targeted. **Dependencies** between binary targets are expressed using the `target_link_libraries()` command:

    add_library(archive archive.cpp zip.cpp lzma.cpp)
    add_executable(zipapp zipapp.cpp)
    target_link_libraries(zipapp archive)

`archive` is defined as a `STATIC library` – an archive containing objects compiled from `archive.cpp`, `zip.cpp`, and `lzma.cpp`. `zipapp` is defined as an executable formed by compiling and linking `zipapp.cpp`. When linking the `zipapp` executable, the `archive` static library is linked in.  

### Binary Executables

The `add_executable()` command defines an executable target:

    add_executable(mytool mytool.cpp)

Commands such as `add_custom_command()`, which generates rules to be run at build time can transparently use an `EXECUTABLE` target as a `COMMAND` executable. The buildsystem rules will ensure that the executable is built before attempting to run the command.  

### Binary Library Types

#### Normal Libraries

By default, the `add_library()` command defines a `STATIC` library, unless a type is specified. A type may be specified when using the command:

    add_library(archive SHARED archive.cpp zip.cpp lzma.cpp)
    add_library(archive STATIC archive.cpp zip.cpp lzma.cpp)

The `BUILD_SHARED_LIBS` variable may be enabled to change the behavior of `add_library()` to build shared libraries by default.

In the context of the buildsystem definition as a whole, it is largely **irrelevant** whether particular libraries are `SHARED` or `STATIC` – the **commands**, **dependency** specifications and other **APIs** work similarly **regardless of** the library type. The `MODULE` library type is dissimilar in that it is generally not linked to – it is not used in the right-hand-side of the `target_link_libraries()` command. It is a type which is loaded as a plugin using runtime techniques. If the library does not export any unmanaged symbols (e.g. Windows resource `DLL`, `C++/CLI DLL`), it is required that the library not be a `SHARED` library because CMake expects `SHARED` libraries to export at least one symbol.

    add_library(archive MODULE 7z.cpp)

#### Apple Frameworks¶

A `SHARED` library may be marked with the `FRAMEWORK` target property to create an macOS or iOS Framework Bundle. The `MACOSX_FRAMEWORK_IDENTIFIER` sets `CFBundleIdentifier` key and it uniquely identifies the bundle.

    add_library(MyFramework SHARED MyFramework.cpp)
    set_target_properties(MyFramework PROPERTIES
      FRAMEWORK TRUE
      FRAMEWORK_VERSION A
      MACOSX_FRAMEWORK_IDENTIFIER org.cmake.MyFramework
    )

#### Object Libraries

The `OBJECT` library type defines a **non-archival collection** of object files resulting from compiling the given source files. The object files collection may be **used as source** inputs to other targets:

    add_library(archive OBJECT archive.cpp zip.cpp lzma.cpp)

    add_library(archiveExtras STATIC $<TARGET_OBJECTS:archive> extras.cpp)

    add_executable(test_exe $<TARGET_OBJECTS:archive> test.cpp)

The link (or archiving) step of those other targets will use the object files collection in addition to those from their own sources.

Alternatively, object libraries **may be linked** into other targets:

    add_library(archive OBJECT archive.cpp zip.cpp lzma.cpp)

    add_library(archiveExtras STATIC extras.cpp)
    target_link_libraries(archiveExtras PUBLIC archive)

    add_executable(test_exe test.cpp)
    target_link_libraries(test_exe archive)

The link (or archiving) step of those other targets will use the object files from `OBJECT` libraries that are directly linked. Additionally, usage requirements of the `OBJECT` libraries will be honored when compiling sources in those other targets. Furthermore, those usage requirements will propagate transitively to dependents of those other targets.

Object libraries may not be used as the `TARGET` in a use of the `add_custom_command(TARGET)` command signature. However, the list of objects can be used by `add_custom_command(OUTPUT)` or `file(GENERATE)` by using `$<TARGET_OBJECTS:objlib>`.

### Build Specification and Usage Requirements

The `target_include_directories()`, `target_compile_definitions()` and `target_compile_options()` commands specify the build specifications and the usage requirements of binary targets. The commands **populate** the `INCLUDE_DIRECTORIES`, `COMPILE_DEFINITIONS` and `COMPILE_OPTIONS` target properties respectively, and/or the `INTERFACE_INCLUDE_DIRECTORIES`, `INTERFACE_COMPILE_DEFINITIONS` and `INTERFACE_COMPILE_OPTIONS` target properties.

Each of the commands has a `PRIVATE`, `PUBLIC` and `INTERFACE` mode. The `PRIVATE` mode populates only the `non-INTERFACE_` variant of the target property and the `INTERFACE` mode populates only the `INTERFACE_` variants. The `PUBLIC` mode populates both variants of the respective target property. Each command may be invoked with multiple uses of each keyword:

    target_compile_definitions(archive
      PRIVATE BUILDING_WITH_LZMA
      INTERFACE USING_ARCHIVE_LIB
    )

Note that usage requirements are not designed as a way to make downstreams use particular `COMPILE_OPTIONS` or `COMPILE_DEFINITIONS` etc for convenience only. The contents of the properties **must be requirements**, not merely recommendations or convenience.

See the Creating Relocatable Packages section of the `cmake-packages(7)` manual for discussion of additional care that must be taken when specifying usage requirements while creating packages for redistribution.

#### Target Properties

The contents of the `INCLUDE_DIRECTORIES`, `COMPILE_DEFINITIONS` and `COMPILE_OPTIONS` target properties are used appropriately when compiling the source files of a binary target.

Entries in the `INCLUDE_DIRECTORIES` are added to the compile line with `-I` or `-isystem` prefixes and **in the order of appearance** in the property value.

Entries in the `COMPILE_DEFINITIONS` are prefixed with `-D` or `/D` and added to the compile line in an **unspecified order**. The `DEFINE_SYMBOL` target property is also added as a compile definition as a special convenience case for `SHARED` and `MODULE` library targets.

Entries in the `COMPILE_OPTIONS` are escaped for the shell and added **in the order of appearance** in the property value. Several compile options have special separate handling, such as `POSITION_INDEPENDENT_CODE`.
The contents of the `INTERFACE_INCLUDE_DIRECTORIES`, `INTERFACE_COMPILE_DEFINITIONS` and `INTERFACE_COMPILE_OPTIONS` target properties are Usage Requirements – they specify content which **consumers** must use to correctly compile and link with the target they appear on. For any binary target, the contents of each `INTERFACE_` property on each target specified in a target_link_libraries() command is consumed:

    set(srcs archive.cpp zip.cpp)
    if (LZMA_FOUND)
      list(APPEND srcs lzma.cpp)
    endif()
    add_library(archive SHARED ${srcs})
    if (LZMA_FOUND)
      # The archive library sources are compiled with -DBUILDING_WITH_LZMA
      target_compile_definitions(archive PRIVATE BUILDING_WITH_LZMA)
    endif()
    target_compile_definitions(archive INTERFACE USING_ARCHIVE_LIB)

    add_executable(consumer)
    # Link consumer to archive and consume its usage requirements. The consumer
    # executable sources are compiled with -DUSING_ARCHIVE_LIB.
    target_link_libraries(consumer archive)

Because it is common to require that the source directory and corresponding build directory are added to the `INCLUDE_DIRECTORIES`, the `CMAKE_INCLUDE_CURRENT_DIR` variable can be enabled to conveniently add the corresponding directories to the `INCLUDE_DIRECTORIES` of all targets. The variable `CMAKE_INCLUDE_CURRENT_DIR_IN_INTERFACE` can be enabled to add the corresponding directories to the `INTERFACE_INCLUDE_DIRECTORIES` of all targets. This makes use of targets in multiple different directories convenient through use of the `target_link_libraries()` command.

#### Transitive Usage Requirements

The usage requirements of a target can **transitively propagate** to dependents. The `target_link_libraries()` command has `PRIVATE`, `INTERFACE` and `PUBLIC` keywords to control the propagation.

    add_library(archive archive.cpp)
    target_compile_definitions(archive INTERFACE USING_ARCHIVE_LIB)

    add_library(serialization serialization.cpp)
    target_compile_definitions(serialization INTERFACE USING_SERIALIZATION_LIB)

    add_library(archiveExtras extras.cpp)
    target_link_libraries(archiveExtras PUBLIC archive)
    target_link_libraries(archiveExtras PRIVATE serialization)
    # archiveExtras is compiled with -DUSING_ARCHIVE_LIB
    # and -DUSING_SERIALIZATION_LIB

    add_executable(consumer consumer.cpp)
    # consumer is compiled with -DUSING_ARCHIVE_LIB
    target_link_libraries(consumer archiveExtras)

Because `archive` is a `PUBLIC` dependency of `archiveExtras`, the usage requirements of it are propagated to `consumer` too. Because `serialization` is a `PRIVATE` dependency of `archiveExtras`, the usage requirements of it are not propagated to `consumer`.

Generally, a dependency should be specified in a use of `target_link_libraries()` with the `PRIVATE` keyword if it is used by **only the implementation** of a library, and **not in the header files**. If a dependency is additionally **used in the header files** of a library (e.g. for class inheritance), then it should be specified as a `PUBLIC` dependency. A dependency which is **not used by the implementation** of a library, but only by its headers should be specified as an `INTERFACE` dependency. The `target_link_libraries()` command may be invoked with multiple uses of each keyword:

    target_link_libraries(archiveExtras
      PUBLIC archive
      PRIVATE serialization
    )

Usage requirements are propagated by reading the `INTERFACE_` variants of target properties from dependencies and appending the values to the `non-INTERFACE_` variants of the operand. For example, the `INTERFACE_INCLUDE_DIRECTORIES` of dependencies is read and appended to the `INCLUDE_DIRECTORIES` of the operand. In cases where order is relevant and maintained, and the order resulting from the `target_link_libraries()` calls does not allow correct compilation, use of an appropriate command to set the property directly may update the order.

For example, if the linked libraries for a target must be specified in the order `lib1` `lib2` `lib3` , but the include directories must be specified in the order `lib3` `lib1` `lib2`:

    target_link_libraries(myExe lib1 lib2 lib3)
    target_include_directories(myExe
      PRIVATE $<TARGET_PROPERTY:lib3,INTERFACE_INCLUDE_DIRECTORIES>)

Note that care must be taken when specifying usage requirements for targets which will be exported for installation using the `install(EXPORT)` command. See Creating Packages for more.

#### Compatible Interface Properties

Some target properties are required to be compatible between a target and the interface of each dependency. For example, the `POSITION_INDEPENDENT_CODE` target property may specify a boolean value of whether a target should be compiled as position-independent-code, which has platform-specific consequences. A target may also specify the usage requirement `INTERFACE_POSITION_INDEPENDENT_CODE` to communicate that consumers must be compiled as position-independent-code.

    add_executable(exe1 exe1.cpp)
    set_property(TARGET exe1 PROPERTY POSITION_INDEPENDENT_CODE ON)

    add_library(lib1 SHARED lib1.cpp)
    set_property(TARGET lib1 PROPERTY INTERFACE_POSITION_INDEPENDENT_CODE ON)

    add_executable(exe2 exe2.cpp)
    target_link_libraries(exe2 lib1)

Here, both exe1 and exe2 will be compiled as position-independent-code. lib1 will also be compiled as position-independent-code because that is the default setting for SHARED libraries. If dependencies have conflicting, non-compatible requirements cmake(1) issues a diagnostic:

    add_library(lib1 SHARED lib1.cpp)
    set_property(TARGET lib1 PROPERTY INTERFACE_POSITION_INDEPENDENT_CODE ON)

    add_library(lib2 SHARED lib2.cpp)
    set_property(TARGET lib2 PROPERTY INTERFACE_POSITION_INDEPENDENT_CODE OFF)

    add_executable(exe1 exe1.cpp)
    target_link_libraries(exe1 lib1)
    set_property(TARGET exe1 PROPERTY POSITION_INDEPENDENT_CODE OFF)

    add_executable(exe2 exe2.cpp)
    target_link_libraries(exe2 lib1 lib2)

The `lib1` requirement `INTERFACE_POSITION_INDEPENDENT_CODE` is not “compatible” with the `POSITION_INDEPENDENT_CODE` property of the exe1 target. The library requires that consumers are built as position-independent-code, while the executable specifies to not built as position-independent-code, so a diagnostic is issued.

The lib1 and lib2 requirements are not “compatible”. One of them requires that consumers are built as position-independent-code, while the other requires that consumers are not built as position-independent-code. Because exe2 links to both and they are in conflict, a diagnostic is issued.

To be “compatible”, the `POSITION_INDEPENDENT_CODE` property, if set must be either the same, in a boolean sense, as the `INTERFACE_POSITION_INDEPENDENT_CODE` property of all transitively specified dependencies on which that property is set.

This property of “compatible interface requirement” may be extended to other properties by specifying the property in the content of the `COMPATIBLE_INTERFACE_BOOL` target property. Each specified property must be compatible between the consuming target and the corresponding property with an `INTERFACE_ prefix` from each dependency:

    add_library(lib1Version2 SHARED lib1_v2.cpp)
    set_property(TARGET lib1Version2 PROPERTY INTERFACE_CUSTOM_PROP ON)
    set_property(TARGET lib1Version2 APPEND PROPERTY
      COMPATIBLE_INTERFACE_BOOL CUSTOM_PROP
    )

    add_library(lib1Version3 SHARED lib1_v3.cpp)
    set_property(TARGET lib1Version3 PROPERTY INTERFACE_CUSTOM_PROP OFF)

    add_executable(exe1 exe1.cpp)
    target_link_libraries(exe1 lib1Version2) # CUSTOM_PROP will be ON

    add_executable(exe2 exe2.cpp)
    target_link_libraries(exe2 lib1Version2 lib1Version3) # Diagnostic

Non-boolean properties may also participate in “compatible interface” computations. Properties specified in the `COMPATIBLE_INTERFACE_STRING` property must be either unspecified or compare to the same string among all transitively specified dependencies. This can be useful to ensure that multiple incompatible versions of a library are not linked together through transitive requirements of a target:

    add_library(lib1Version2 SHARED lib1_v2.cpp)
    set_property(TARGET lib1Version2 PROPERTY INTERFACE_LIB_VERSION 2)
    set_property(TARGET lib1Version2 APPEND PROPERTY
      COMPATIBLE_INTERFACE_STRING LIB_VERSION
    )

    add_library(lib1Version3 SHARED lib1_v3.cpp)
    set_property(TARGET lib1Version3 PROPERTY INTERFACE_LIB_VERSION 3)

    add_executable(exe1 exe1.cpp)
    target_link_libraries(exe1 lib1Version2) # LIB_VERSION will be "2"

    add_executable(exe2 exe2.cpp)
    target_link_libraries(exe2 lib1Version2 lib1Version3) # Diagnostic

The `COMPATIBLE_INTERFACE_NUMBER_MAX` target property specifies that content will be evaluated numerically and the maximum number among all specified will be calculated:

    add_library(lib1Version2 SHARED lib1_v2.cpp)
    set_property(TARGET lib1Version2 PROPERTY INTERFACE_CONTAINER_SIZE_REQUIRED 200)
    set_property(TARGET lib1Version2 APPEND PROPERTY
      COMPATIBLE_INTERFACE_NUMBER_MAX CONTAINER_SIZE_REQUIRED
    )

    add_library(lib1Version3 SHARED lib1_v3.cpp)
    set_property(TARGET lib1Version3 PROPERTY INTERFACE_CONTAINER_SIZE_REQUIRED 1000)

    add_executable(exe1 exe1.cpp)
    # CONTAINER_SIZE_REQUIRED will be "200"
    target_link_libraries(exe1 lib1Version2)

    add_executable(exe2 exe2.cpp)
    # CONTAINER_SIZE_REQUIRED will be "1000"
    target_link_libraries(exe2 lib1Version2 lib1Version3)

Similarly, the `COMPATIBLE_INTERFACE_NUMBER_MIN` may be used to calculate the numeric minimum value for a property from dependencies.

Each calculated “compatible” property value may be read in the consumer at generate-time using generator expressions.

Note that for each dependee, the set of properties specified in each compatible interface property must not intersect with the set specified in any of the other properties.

#### Property Origin Debugging
