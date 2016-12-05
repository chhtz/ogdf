[OGDF](README.md) » Build Guide

# Build Guide {#build}

The OGDF build configuration is generated using [CMake](http://www.cmake.org/).

## Requirements

 * CMake 3.1+
 * C++11 compliant compiler
   * gcc 4.8+
   * clang 3.5+
   * Microsoft Visual C++ 2015+
 * GNU Make (in most cases)
 * Doxygen (optional)

## Build Configuration

CMake is a meta-build system that will generate your actual build system.
The most common build systems include Unix Makefiles, Visual Studio project files, and XCode project files.
We refer to [Running CMake](http://www.cmake.org/runningcmake/) for extensive information on using CMake.

Note that CMake allows you to place the generated build system in a separate folder, thus allowing several parallel build configurations (called out-of-source builds). We recommend following this approach.

## Unix Examples

All of the examples cover unix systems. On windows, CMake provides a self-explanatory graphical user interface.

We assume that the directory `~/OGDF` contains a clone of this repository.
In each of the examples we are initially located in the home directory `~`.

CMake will present you with several options for configuring your build.
All of these options are explained by the CMake interface.

### Default Configuration

This will generate the default build system inside the OGDF directory.
Such a configuration is sufficient if you want to compile the Library with a single configuration only.

```
$ cd OGDF
$ cmake .
-- The CXX compiler identification is GNU 6.2.0
-- Check for working CXX compiler: /usr/bin/c++
-- Check for working CXX compiler: /usr/bin/c++ -- works
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Found LIBDW: /usr/lib/x86_64-linux-gnu/libdw.so
-- Found LIBBFD: /usr/lib/x86_64-linux-gnu/libbfd.so
-- Found LIBUNWIND: /usr/lib/x86_64-linux-gnu/libunwind.so
-- Found Doxygen: /usr/bin/doxygen (found version "1.8.12")
-- Configuring done
-- Generating done
-- Build files have been written to: ~/OGDF
```

Assuming your system supports Unix Makefiles you could now start the actual build:
Note that invoking make with the `-j` flag will execute jobs in parallel thus speeding up your build.

```
$ make -j8
Scanning dependencies of target COIN
Scanning dependencies of target OGDF
[  0%] Building CXX object CMakeFiles/COIN.dir/src/coin/Clp/ClpConstraintLinear.cpp.o
[  0%] Building CXX object CMakeFiles/COIN.dir/src/coin/Clp/ClpCholeskyBase.cpp.o
[  0%] Building CXX object CMakeFiles/COIN.dir/src/coin/Clp/ClpConstraint.cpp.o
[  1%] Building CXX object CMakeFiles/COIN.dir/src/coin/Clp/ClpCholeskyTaucs.cpp.o
[  1%] Building CXX object CMakeFiles/COIN.dir/src/coin/Clp/ClpCholeskyDense.cpp.o
[  1%] Building CXX object CMakeFiles/COIN.dir/src/coin/Clp/ClpConstraintQuadratic.cpp.o
[  1%] Building CXX object CMakeFiles/COIN.dir/src/coin/Clp/ClpDualRowDantzig.cpp.o
[  1%] Building CXX object CMakeFiles/COIN.dir/src/coin/Clp/ClpDualRowPivot.cpp.o
[  2%] Building CXX object CMakeFiles/OGDF.dir/src/ogdf/augmentation/DfsMakeBiconnected.cpp.o
...
```

### Out-of-Source Build

An out-of-source build does not modify the source directory.
Thus, generating a second out-of-source build will not create any conflicts with the previous one.

```
$ mkdir my-out-of-source-build
$ cd my-out-of-source-build
$ cmake ../OGDF
...
-- Configuring done
-- Generating done
-- Build files have been written to: ~/my-out-of-source-build
```

Note that the documentation is always built in-source.

### Custom Configuration

Creating multiple build configurations is of no use if the configurations do not differ.
Run `ccmake` instead of `cmake` to modify the build configuration before generating the build system.

This includes the specification of the linear program solver,
whether to include OGDF specific assertions (and how to handle them),
additional include directories, and the likes.
Running `ccmake` also allows you to specify the compiler, linker and the respective flags.

```
$ mkdir ogdf-release ogdf-debug
$ cd ogdf-release
$ cmake ../OGDF # default configuration
$ cd ../ogdf-debug
$ cmake ../OGDF
$ ccmake .

 BUILD_SHARED_LIBS               >OFF
 CMAKE_BUILD_TYPE                 Release
 CMAKE_INSTALL_PREFIX             /usr/local
 COIN_EXTERNAL_SOLVER_INCLUDE_D
 COIN_EXTERNAL_SOLVER_LIBRARIES
 COIN_SOLVER                      CLP
 OGDF_MEMORY_MANAGER              POOL_TS
 OGDF_SEPARATE_TESTS              OFF
 OGDF_USE_ASSERT_EXCEPTIONS       OFF
 OGDF_USE_ASSERT_EXCEPTIONS_WIT   OFF

BUILD_SHARED_LIBS: Whether to build shared libraries instead of static ones.
Press [enter] to edit option
Press [c] to configure
Press [h] for help           Press [q] to quit without generating
Press [t] to toggle advanced mode (Currently Off)

....

 BUILD_SHARED_LIBS                OFF
 CMAKE_BUILD_TYPE                 Debug
 CMAKE_INSTALL_PREFIX             /usr/local
 COIN_EXTERNAL_SOLVER_INCLUDE_D   /opt/gurobi/linux64/include
 COIN_EXTERNAL_SOLVER_LIBRARIES   /opt/gurobi/linux64/lib/libgurobi70.so
 COIN_SOLVER                      GRB
 OGDF_MEMORY_MANAGER             >MALLOC_TS
 OGDF_SEPARATE_TESTS              ON
 OGDF_USE_ASSERT_EXCEPTIONS_WIT   ON_LIBDW

OGDF_MEMORY_MANAGER: Memory manager to be used.
Press [enter] to edit option
Press [c] to configure       Press [g] to generate and exit
Press [h] for help           Press [q] to quit without generating
Press [t] to toggle advanced mode (Currently Off)
```

#### On assertions

OGDF code has a lot of extra assertions in its code, partly adding running time.
These assertions are only activated in debug mode and if the advanced option
`OGDF_DEBUG` is activated (it is by default).
The assertions (plus a backtrace of the calls) might be very valuable and
helpful for debugging (user) code.

There are two modes how failed assertions are handled.

If `OGDF_USE_ASSERT_EXCEPTIONS` is turned off (default), an `assert()`
call exits the program.
Note that enabling this might hide failed assertions when `catch (...)` is used.
You can get a call stack trace if the program was invoked with a debugger
(which is part of many IDEs).

If turned on, an `ogdf::AssertionFailed` exception is thrown instead.
The informational string returned by the `what()` method contains the
failed expression, its source code file, line, and function.
By also setting the `OGDF_USE_ASSERT_EXCEPTIONS_WITH_STACK_TRACE` variable to
`ON_LIBDW` (if libdw is installed, recommended), `ON_LIBBFD` (if libbfd
from binutils is installed), or `ON_LIBUNWIND` (if libunwind is installed),
a stack trace will be part of the exception's `what()` return string.
As far as we know, this feature only works on Linux.