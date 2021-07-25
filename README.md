Obfuscation
===========

## Install clang with visual studio [OPTIONAL]
1. open visual studio 2019 go to Tools > Get Tools and Features > individual components and type clang in searchhttps://i.ibb.co/qmpkmPs/visualclang.png install these
2. now you can compile projects with clang
https://i.ibb.co/YNC0PLk/setclang.png
   
## Compile llvm-obfuscator
1. Install [Git](https://git-scm.com/download/win)
2. Install [mingw-w64](http://mingw-w64.org/doku.php/download)
3. Install [Microsoft BuildTools](https://visualstudio.microsoft.com/downloads/?utm_medium=microsoft&utm_source=docs.microsoft.com&utm_campaign=navigation+cta&utm_content=download+vs2019)
4. Add mingw to environment path variables
5. Clone LLVM from [this](https://github.com/klezVirus/obfuscator) repo and compile
```bat
git clone -b llvm-9.0.1 https://github.com/klezVirus/obfuscator
cd obfuscator
mkdir build
cd build
"C:\Program Files (x86)\Microsoft Visual Studio\2019\BuildTools\Common7\Tools\VsDevCmd.bat"
cmake.exe -DCMAKE_BUILD_TYPE=Release -G "MinGW Makefiles" ..
mingw32-make.exe -j7
```

## Integrate with Visual Studio 2019

#### NW: This step is completely avoidable if you just want to use [Inceptor](https://github.com/klezVirus/inceptor)

1. Change the windows standard binaries for clang

```bat
cd obfuscator\build\bin
xcopy /S /Y * "C:\Program Files (x86)\Microsoft Visual Studio\2019\Community\VC\Tools\Llvm\x64\bin\"
```

2. In VS2019, select clang-cl as the main build 
3. Add LLVM-Obfuscator arguments to the compiler command line

```
-D__CUDACC__ -D_ALLOW_COMPILER_AND_STL_VERSION_MISMATCH -mllvm -bcf -mllvm -bcf_prob=73 -mllvm -bcf_loop=1 -mllvm -sub -mllvm -sub_loop=5 -mllvm -fla -mllvm -split_num=5 -mllvm -aesSeed=DEADBEEFDEADCODEDEADBEEFDEADCODE
```

## Fast clang compilation

If used from command line, the Clang-CL compiler can be speed up by using the Microsoft MPCL routine, which stands for
"Multi-Process CL". The tool will use multiple process to perform compilation, with sometimes a
considerable performance improvement.

```
mpcl.exe clang-cl.exe /MP file1.c file2.c file3.c <-- creates 1 process per file
```

## Porting to LLVM-obfuscator 11.0

It is also possible to use version 11 of LLVM. I personally had some issues on some Windows VM,
but the same procedure explained here should work fine for LLVM v11 as well.

You can find LLVM v11 [here](https://github.com/liLeiBest/obfuscator/tree/llvm-11.0.0)

## Using Ninja as build system (Untested)

In order to use ninja, the only necessary thing is to download it, compiler llvm-obfuscator as 
already shown, then put the ninja binary in the same directory as clang.

This PowerShell script is an example of how you can install it from the CLI.

```powershell
iwr "https://github.com/ninja-build/ninja/releases/download/v1.10.2/ninja-win.zip" -O "ninja-win.zip"
git clone https://github.com/crvvdev/obfuscator
# This should be set to where you've installed MinGW
[System.Environment]::SetEnvironmentVariable("CMAKE_MAKE_PROGRAM", "C:\Program Files\mingw-w64\x86_64-8.1.0-posix-seh-rt_v6-rev0\mingw64\bin\mingw32-make.exe")
cd obfuscator
mkdir build
cd build
# Extract ninja binary to `build` directory
Expand-Archive -LiteralPath .\ninja-win.zip -DestinationPath .
if((Get-Command cmake.exe -ea silentlyconinue) -eq $null)
{
    # If cmake is not in path, start VS developer console
    Start-Process "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Visual Studio 2019\Visual Studio Tools\Developer Command Prompt for VS 2019.lnk"
    Write-Host "[!] In VS Developer console, run the following command"
    Write-Host "    cmake ../llvm -G Ninja -DLLVM_ENABLE_PROJECTS="clang;libcxx" -DCMAKE_BUILD_TYPE=Release -DLLVM_TARGETS_TO_BUILD=X86 -DLLVM_INCLUDE_EXAMPLES=OFF -DLLVM_BUILD_TESTS=OFF -DLLVM_INCLUDE_TESTS=OFF -DCMAKE_CXX_FLAGS=/std:c++14 -DBUILD_SHARED_LIBS=OFF -DLLVM_PARALLEL_COMPILE_JOBS=4 -DLLVM_PARALLEL_LINK_JOBS=4"
}
else
{
    # If cmake is in your path, you might compile
    cmake ../llvm -G Ninja -DLLVM_ENABLE_PROJECTS="clang;libcxx" -DCMAKE_BUILD_TYPE=Release -DLLVM_TARGETS_TO_BUILD=X86 -DLLVM_INCLUDE_EXAMPLES=OFF -DLLVM_BUILD_TESTS=OFF -DLLVM_INCLUDE_TESTS=OFF -DCMAKE_CXX_FLAGS=/std:c++14 -DBUILD_SHARED_LIBS=OFF -DLLVM_PARALLEL_COMPILE_JOBS=4 -DLLVM_PARALLEL_LINK_JOBS=4
}
```

# Credits

Thanks to discord user frostiest#6102