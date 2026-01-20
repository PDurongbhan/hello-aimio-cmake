# Hello COM_MEAS with AimIO (Windows, CMake, conda)

This repository is a **minimal, working example** of building and linking
[AimIO](https://github.com/Numerics88/AimIO) to perform joint centre of mass calculation on **Windows** using:

* MSVC (Visual Studio, x64)
* CMake (>= 3.20)
* Ninja
* conda-forge / numerics88 packages (Boost, n88util)

The goal of this repo is **reproducibility**: once this works, you should not
have to rediscover the same toolchain and dependency issues again.

This project was developed and tested on Windows 10 using a conda environment.

---

## What this repo contains

* `main.cpp`
  Minimal program that includes AimIO headers and links successfully.

* `CMakeLists.txt`
  Consumer-side CMake that finds AimIO, Boost, and n88util via `find_package`
  and builds an executable.

This repo **does not** vendor AimIO itself. AimIO is built separately and
installed into a conda environment.

---

## Prerequisites

### 1. Visual Studio (MSVC x64)

Install **Visual Studio Community** with:

* Desktop development with C++
* MSVC v143 toolset (or newer)

You must use the **x64 toolchain**.
Do NOT use x86.

### 2. conda (Anaconda or Miniconda)

Ensure `conda` is available in your shell.

### 3. Git

Git for Windows (with Git Credential Manager enabled).

---

## Create the conda environment

Example environment (name used here: `aimio-dev`):

```
conda create -n aimio-dev -y -c conda-forge cmake ninja boost boost-cpp
conda activate aimio-dev
```

Install `n88util` and `aimio` dependencies from numerics88:

```
conda install -y -c numerics88 n88util
```

Note:

* Boost headers live in: <env>/Library/include
* Some packages install CMake configs into: <env>/Library/CMake <env>/CMake

This matters on Windows.

---

## Build and install AimIO (from source)

AimIO must be built from source and installed into the conda environment.

Example directory layout:

```
C:\COMMS\
  external\
    AimIO\
```

### Open the correct shell

Use **Developer PowerShell for VS** (NOT PowerShell x86).

Confirm x64 compiler:

```
where.exe cl
```

You should see a path containing:
Hostx64\x64\cl.exe

### Configure AimIO

From the AimIO source directory:

```
cmake -S . -B build -G Ninja ^
  -Dn88util_DIR=%CONDA_PREFIX%\Library\CMake ^
  -DBoost_DIR=%CONDA_PREFIX%\Library\lib\cmake\Boost-1.78.0 ^
  -DCMAKE_INSTALL_PREFIX=%CONDA_PREFIX% ^
  -DN88_BUILD_AIX=OFF ^
  -DN88_BUILD_CTHEADER=OFF
```

### Build and install

```
cmake --build build
cmake --install build
```

Verify install:

```
<env>/include/AimIO/AimIO.h
<env>/CMake/AimIOConfig.cmake
```

---

## Build this project

From the root of this repository:

```
conda activate aimio-dev

cmake -S . -B build -G Ninja ^
  -Dn88util_DIR=%CONDA_PREFIX%\Library\CMake ^
  -DBoost_DIR=%CONDA_PREFIX%\Library\lib\cmake\Boost-1.78.0 ^
  -DAimIO_DIR=%CONDA_PREFIX%\CMake ^
  -DCMAKE_PREFIX_PATH=%CONDA_PREFIX%;%CONDA_PREFIX%\Library

cmake --build build
build\hello.exe
```

Expected output:

```
Hello from modern C++ + CMake
AimIO include works.
```

---

## Important Windows notes

1. **x64 vs x86 matters**
   Mixing x86 and x64 will cause Boost and AimIO link failures.

2. **Boost headers are NOT in `<env>/include`**
   They are in: <env>/Library/include

   This is why the CMakeLists explicitly adds both include paths.

3. **AimIO does not automatically pull transitive dependencies**
   Consumer projects must explicitly call:

   ```
   find_package(n88util CONFIG REQUIRED)
   find_package(Boost CONFIG REQUIRED COMPONENTS filesystem system)
   ```

   before `find_package(AimIO)`.

4. **Debug vs Release**
   Be consistent. If AimIO is installed as Debug, your app must also be Debug.

---

## CMakeLists.txt (consumer)

The current working CMakeLists.txt for this project is:

```
cmake_minimum_required(VERSION 3.20)
project(HelloAimIO LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

find_package(n88util CONFIG REQUIRED)
find_package(Boost CONFIG REQUIRED COMPONENTS filesystem system)
find_package(AimIO CONFIG REQUIRED)

add_executable(hello main.cpp)
target_link_libraries(hello PRIVATE AimIO::AimIO)

target_include_directories(hello PRIVATE
    $ENV{CONDA_PREFIX}/include
    $ENV{CONDA_PREFIX}/Library/include
)
```

---

## Why this repo exists

Building C++ libraries on Windows with conda, MSVC, Boost, and CMake is
not trivial. This repository serves as a **known-good reference** for:

* Toolchain setup
* Dependency discovery
* AimIO integration
* Avoiding common Windows pitfalls

If this saves someone even one afternoon, it has done its job.
