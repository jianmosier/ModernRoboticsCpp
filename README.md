# Modern Robotics:  Mechanics, Planning, and Control
# C++ Library

This repository contains the code library accompanying [_Modern Robotics:
Mechanics, Planning, and Control_](http://modernrobotics.org) (Kevin Lynch
and Frank Park, Cambridge University Press 2017). The
[user manual](https://github.com/NxRLab/ModernRobotics/blob/master/doc/MRlib.pdf) is in the doc directory of [main repository](https://github.com/NxRLab/ModernRobotics/).

The functions are available in:

* C++
* [Python](https://github.com/NxRLab/ModernRobotics/tree/master/packages/Python)
* [MATLAB](https://github.com/NxRLab/ModernRobotics/tree/master/packages/Matlab)
* [Mathematica](https://github.com/NxRLab/ModernRobotics/tree/master/packages/Mathematica)

Each function has a commented section above it explaining the inputs required for its use as well as an example of how it can be used and what the output will be. This repository also contains a pdf document that provides an overview of the available functions using MATLAB syntax. Functions are organized according to the chapter in which they are introduced in the book. Basic functions, such as functions to calculate the magnitude of a vector, normalize a vector, test if the value is near zero, and perform matrix operations such as multiplication and inverses, are not documented here.

The primary purpose of the provided software is to be easy to read and educational, reinforcing the concepts in the book. The code is optimized neither for efficiency nor robustness.

## Installation

### 1. Install dependencies.
* CMake 3.14 or newer
* A C++11 compiler
* Eigen3
* GoogleTest is downloaded automatically when tests are enabled.

If Eigen3 is not found, CMake can download Eigen 3.4.0 automatically.

* On Windows: Visual Studio 2022 or Visual Studio Build Tools 2022 are supported.
  The Visual Studio IDE is not required; you can build from a Visual Studio Code
  terminal with MSVC Build Tools and Ninja.

* On Mac
```console
foo@bar:~$ brew install eigen
```

* On Linux
```console
foo@bar:~$ sudo apt-get install libeigen3-dev
```

### 2. Prepare build.
```console
foo@bar:~$ mkdir build && cd build
```

By default cmake will install our build into the system directories.
To define a custom install directory we simply pass it to cmake:
```console
foo@bar:build $ cmake .. -DCMAKE_INSTALL_PREFIX=../_install
```

On Windows with Visual Studio:
```console
foo@bar:build $ cmake .. -G "Visual Studio 17 2022" -A x64 -DCMAKE_INSTALL_PREFIX=../_install
```

On Windows with Ninja, for example from a Visual Studio Code terminal:
```console
foo@bar:build $ cmake .. -G Ninja -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=../_install
```

Or just configure with defaults:
```console
foo@bar:build $ cmake ..
```

To use an existing Eigen checkout, set `EIGEN3_INCLUDE_DIR` to the folder
containing `Eigen/Dense`:
```console
foo@bar:build $ cmake .. -DEIGEN3_INCLUDE_DIR=C:\path\to\eigen
```

Building and installing library:
```console
foo@bar:build $ cmake --build . --config Release
foo@bar:build $ cmake --install . --config Release
```

With Makefiles, this also works:
```console
foo@bar:build $ make all && make install
```

## Using the library

Include the public header and use the `mr` namespace:

```cpp
#include <modern_robotics.h>

Eigen::Matrix3d m = mr::VecToso3(Eigen::Vector3d(1, 2, 3));
```

In CMake:

```cmake
find_package(ModernRoboticsCpp CONFIG REQUIRED)
target_link_libraries(my_app PRIVATE ModernRoboticsCpp::ModernRoboticsCpp)
```

On Windows, make sure `ModernRoboticsCpp.dll` is next to your executable or in `PATH`.

## Testing the library

```console
foo@bar:build $ ./lib_test
```

On Windows with the Visual Studio generator:
```console
foo@bar:build $ .\Release\lib_test.exe
```

Or run all discovered tests with CTest:
```console
foo@bar:build $ ctest -C Release --output-on-failure
```
