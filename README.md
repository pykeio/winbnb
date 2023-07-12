# winbnb
A non-hacky Windows distribution of [`bitsandbytes`](https://github.com/TimDettmers/bitsandbytes). Inspired by [`ShanGor/bitsandbytes-windows`](https://github.com/ShanGor/bitsandbytes-windows), updated for `bitsandbytes` 0.40.0-post4 with 4-bit inference, and less hacky specifically in the CUDA loading parts.

For most purposes, you should probably just use WSL 2, but this package is intended for usage in memory-constrained environments where WSL taking an extra few GB of RAM on top of whatever absurd amount CUDA itself allocates is undesirable.

## Usage
Currently we only distribute prebuilt binaries for CUDA 11.8. If you have CUDA 11.8, great! Head over to `Releases` to pick up our prebuilt wheel.

Otherwise, head to Building below.

## Building
You need your choice of CUDA Toolkit > 11 and Visual Studio Community/Build Tools 2022.

In PowerShell:
```ps
mkdir dependencies
cd dependencies
# use wget if you have that (choco install wget) or just download it manually
wget https://github.com/GerHobbelt/pthread-win32/archive/refs/tags/version-3.1.0-release.zip
expand-archive version-3.1.0-release.zip .
rm version-3.1.0-release.zip
mv pthread-win32-version-3.1.0-release pthread-win32
```

In x64 Native Tools Command Prompt for VS 2022 (aka hell):
```
# configure these accordingly
set WINBNB=E:\Dev\winbnb       # winbnb clone path
set PYTHON_HOME=C:\Python3.10
set CUDA_HOME=C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v11.8\bin
set CUDA_VERSION=118           # e.g. 11.8 = 118, 12.0 = 120
set LIB=%LIB%;%CUDA_HOME%/lib/x64;%PYTHON_HOME%/lib;%WINBNB%/dependencies/pthread-win32

cd %WINBNB%\dependencies\pthread-win32
nmake VC-static
cd %WINBNB%
mkdir build
nvcc -gencode arch=compute_75,code=sm_75 -gencode arch=compute_80,code=sm_80 -gencode arch=compute_86,code=sm_86 -gencode arch=compute_89,code=sm_89 -gencode arch=compute_90,code=sm_90 -Xcompiler /DYNAMICBASE --use_fast_math -Xptxas=-v -dc %WINBNB%/csrc/ops.cu %WINBNB%/csrc/kernels.cu -I %CUDA_HOME%/include -I %WINBNB%/csrc -I %PYTHON_HOME%/include -I %WINBNB%/include -I %WINBNB%/dependencies/pthread-win32 -L %CUDA_HOME%/lib/x64 -lcudart -lcublas -lcublasLt -lcusparse -lpthreadVC3 -L %PYTHON_HOME%/lib -L %WINBNB%/dependencies/pthread-win32 --output-directory %WINBNB%/build
nvcc -gencode arch=compute_75,code=sm_75 -gencode arch=compute_80,code=sm_80 -gencode arch=compute_86,code=sm_86 -gencode arch=compute_89,code=sm_89 -gencode arch=compute_90,code=sm_90 -Xcompiler /DYNAMICBASE -dlink %WINBNB%/build/ops.obj %WINBNB%/build/kernels.obj -o %WINBNB%/build/link.obj
cl /EHsc /DBUILD_CUDA=1 /std:c++14 /LD /I %CUDA_HOME%/include /I %WINBNB%/csrc /I %PYTHON_HOME%/include /I %WINBNB%/include /I %WINBNB%/dependencies/pthread-win32 %WINBNB%/build/kernels.obj %WINBNB%/build/ops.obj %WINBNB%/build/link.obj ./csrc/common.cpp ./csrc/cpu_ops.cpp ./csrc/pythonInterface.cpp libpthreadVC3.lib cudart.lib cublas.lib cublasLt.lib cusparse.lib /link /out:./bitsandbytes/bitsandbytes_cuda%CUDA_VERSION%.dll
```
