# Linux eBPF installation guide

This guide provides an overview of the installation of libbpf and bpftool, whiwh are tools for eBPF.
We also need clang/llvm to compile eBPF code.

## Installing dependencies for bpftool

First of all, make sure your system is already updated with the following command:
```
sudo apt-get update -y
```

Now, we have to install some packages needed by bpftool.
We need `libelf-dev`, `zlib`, `libbfd-dev`, `libcap-dev`, `libbpf-dev` and `llvm` package.

```
sudo apt-get install -y zlib1g-dev libelf-dev libbfd-dev libcap-dev libbpf-dev llvm
```

On Ubuntu distributions, clang is basically installed but if it's not, you can install it with
```
sudo apt install clang
```
## Installing bpftool and libbpf

Now, you can install bpftool without any stress.
You just have to clone the bpftool repository, compile, and install it with :

```
git clone --recurse-submodules https://github.com/libbpf/bpftool.git
cd bpftool/src
sudo make install
```

Your installation should be complete, and you can now compile BPF code. 

## Problem with the location of shared library

If you try to execute this code, you will see this issue :
```
sudo ./simple
./simple: error while loading shared libraries: libbpf.so.1: cannot open shared object file: No such file or directory
```

If this issue occurs, you just need to tell the opearting system where it can locate it at runtime.

To do so, we will need to do those steps:

1. Find where the library is placed if you dont know it.
```sudo find / -name libbpf.so.1
```
2. Check for the existence of the dynamic library path environment variable(`LD_LIBRARY_PATH`)
```
echo $LD_LIBRARY_PATH
```
If there is nothing to be displayed, add a default path value (or not if you wish to)
```
LD_LIBRARY_PATH=/my_path
```
where my_path is the path for the file libbpf.so.1 that you find with the previous command

3. We add the desired path, export it and try the application.
```
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/my_path
```

Now you can run 
```
sudo ldconfig my_path
```

With that if you try to execute your code, it will work.

Congratulation, now you can make and execute instructions and code to use eBPF technology. 
