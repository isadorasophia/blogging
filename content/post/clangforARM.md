+++
date = "2016-09-07T16:09:58+05:30"
draft = true
title = "Compiling Clang/LLVM for ARM support"
description = "How I was able to build applications for ARM with OpenMP support, using Clang"
+++

Recently I have to face up the challenge to cross compile applications for ARM-v7 using Clang and LLVM, along with OpenMP support.
I figured it wouldn't be **that** hard, since I have already used gcc utils for a while and had no problem with that. First, I would have to compile LLVM from source in order to use it with my current project.

I proceeded cloning the LLVM repository, along with some compiler supported routines for OpenMP and Clang.

    git clone https://github.com/clang-omp/llvm
    git clone https://github.com/clang-omp/compiler-rt llvm/projects/compiler-rt
    git clone -b clang-omp https://github.com/clang-omp/clang llvm/tools/clang

Since autoconf was kinda messy, I approached it using CMake. After some research in the documentation-land, I proceeded with the following commands in order to compile Clang:

    mkdir build && cd build

    cmake -DLLVM_TARGETS_TO_BUILD="ARM" /path/to/llvm/source

and then simply:

    cmake --build .

Alright. Bin files generated. I linked the proper variables and proceeded to the next step of compilation.

Ok, now what I should do with it? In order to keep everything clean and pretty, I created a simple script to export the proper variables, as follows:

    export OMP_CLANG=path/to/build/dir

    # executables
    export PATH=$OMP_CLANG/bin:$PATH

    # directories
    export LIBRARY_PATH=$OMP_CLANG/lib:$LIBRARY_PATH
    export LD_LIBRARY_PATH=$OMP_CLANG/lib:$LD_LIBRARY_PATH

After some more extensive search thourgh the brave LLVM [documentation], I proceeded compiling my sample program with the following options:

    clang++ -target armv7-none-linux-gnueabihf toy.cpp -o toy

The target that I used was a triplet described in the LLVM documentation. Ok! But things weren't done yet. I still had to compile my project using IOmp, Intel OpenMP run-time library. I cloned the repository and proceeded with CMake, once again.

    mkdir build && cd build

    cmake ../ -DLIBOMP_ARCH=arm -DCMAKE_C_COMPILER=arm-linux-gnueabihf-gcc -DCMAKE_CXX_COMPILE=arm-linux-gnueabihf-g++

And that should be it! Right? Well, no. For some reason, LLVM kept ignoring my CXX flag and assigning g++ for me. So, when it compiles, it run all the C files correctly (with my gnueabihf-gcc, naturally), but the C++ incorrectly!
    
What it turns out is that the CMake was basically ignoring the "CMAKE_C_COMPILER" flag. After I exported variables C and CXX as above:

    export C=arm-linux-gnueabihf-gcc
    export CXX=arm-linux-gnueabihf-g++

I was able to successfully compile the runtime. With everything set, I needed some more instructions to build my program, in order to link with the Intel run-time library:
    
    export OMP_INTEL=path/to/build/dir/src

    export LIBRARY_PATH=$OMP_INTEL:$LIBRARY_PATH
    export LD_LIBRARY_PATH=$OMP_INTEL:$LD_LIBRARY_PATH

And then, I finally built it as:

    clang++ -target armv7-none-linux-gnueabihf -L$OMP_INTEL -fopenmp toy.cpp -o toy

Easy, right?

[documentation]: http://llvm.org/docs/
