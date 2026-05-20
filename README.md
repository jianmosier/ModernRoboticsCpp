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

ModernRoboticsCpp requires CMake 3.14 or newer and a C++11 compiler.
Eigen is used as a header-only dependency. CMake will first try to find an
installed Eigen3 package, then an `EIGEN3_INCLUDE_DIR` path, and finally
download Eigen 3.4.0 automatically when `MODERN_ROBOTICS_FETCH_DEPENDENCIES`
is enabled.

### Windows

With Visual Studio 2022:

```console
cmake -S . -B build-win -G "Visual Studio 17 2022" -A x64
cmake --build build-win --config Release
cmake --install build-win --config Release --prefix _install
```

To use an existing Eigen checkout instead of downloading it:

```console
cmake -S . -B build-win -DEIGEN3_INCLUDE_DIR=C:\path\to\eigen
```

### macOS and Linux

Install Eigen with your package manager if you do not want CMake to download it:

```console
brew install eigen
sudo apt-get install libeigen3-dev
```

Then configure, build, and install:

```console
cmake -S . -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build --parallel
cmake --install build --prefix _install
```

## Using the library

The public header is:

```cpp
#include <modern_robotics.h>
```

The functions live in the `mr` namespace. A minimal example:

```cpp
#include <Eigen/Dense>
#include <iostream>
#include <modern_robotics.h>

int main() {
  Eigen::Vector3d omega(1.0, 2.0, 3.0);
  std::cout << mr::VecToso3(omega) << '\n';
  return 0;
}
```

If ModernRoboticsCpp is part of your source tree, add it directly:

```cmake
add_subdirectory(path/to/ModernRoboticsCpp)
target_link_libraries(my_app PRIVATE ModernRoboticsCpp::ModernRoboticsCpp)
```

If you installed the library, point CMake at the install prefix and use the
exported package. The consuming project still needs Eigen available through an
installed Eigen3 package or `EIGEN3_INCLUDE_DIR`.

```console
cmake -S . -B build -DCMAKE_PREFIX_PATH=C:\path\to\ModernRoboticsCpp\_install
```

```cmake
find_package(ModernRoboticsCpp CONFIG REQUIRED)
target_link_libraries(my_app PRIVATE ModernRoboticsCpp::ModernRoboticsCpp)
```

When using the shared library on Windows, make sure `ModernRoboticsCpp.dll` is
next to your executable or available through `PATH`.

## Testing the library

Tests are enabled by default when this repository is the top-level CMake project.
GoogleTest is downloaded automatically during configure.

```console
ctest --test-dir build-win -C Release --output-on-failure
```

For single-config generators such as Makefiles or Ninja:

```console
ctest --test-dir build --output-on-failure
```
