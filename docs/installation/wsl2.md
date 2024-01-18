# WSL2 Installation Guide

!!! warning "Warning"
    This guide is not exhaustive, and making modifications to the WSL kernel may lead to system instability, data corruption, and other potential issues. Proceed with caution and back up your data before attempting any changes.

Installing the eBPF technology along with essential tools such as bcc, bpftool, bpftrace, clang, and llvm on WSL2 can be a complex and time-consuming process. This guide provides a step-by-step overview of the installation process, explaining the purpose of each tool.

## Compiling the WSL Kernel and Obtaining Necessary Headers

On WSL2, certain dependencies and packages required for this installation are not readily available through `apt-get`. the `linux-headers-$(uname -r)` package is not available through standard package managers like `apt-get`. Therefore, manual kernel compilation is necessary to generate the required headers for eBPF.

```
[23:47:05] [~/master_info/eBPF] ❱❱❱ sudo apt-get install linux-headers-$(uname -r)
[sudo] password for dwayne:
Reading package lists... Done
Building dependency tree
Reading state information... Done
E: Unable to locate package linux-headers-6.1.21.2-microsoft-standard-WSL2
E: Couldn't find any package by glob 'linux-headers-6.1.21.2-microsoft-standard-WSL2'
E: Couldn't find any package by regex 'linux-headers-6.1.21.2-microsoft-standard-WSL2'

```

Kernel headers are essential files that contain information about functions, structures, and constants used in the Linux kernel. They are crucial for building and compiling kernel modules, which are required for eBPF.

Our objective is to compile the WSL kernel from [source](https://github.com/microsoft/WSL2-Linux-Kernel) to generate the necessary header files for eBPF. Additionally, while eBPF has been available since kernel version 3.15, I recommend updating your kernel if it is too old to ensure full eBPF functionality. It's essential to be aware that eBPF features and tools may have evolved over time, and some features may require a more recent kernel version to work optimally.


!!! Tip "Tip"
    You can refer to the official [bcc documentation](https://github.com/iovisor/bcc/blob/master/docs/kernel-versions.md) for specific details on which kernel versions support various eBPF features.

In my case, I chose to install the latest available Microsoft Linux kernel, which is [version 6](https://learn.microsoft.com/en-us/community/content/wsl-user-msft-kernel-v6). This ensures that I have access to all the latest eBPF capabilities and toolsets, providing a more robust and up-to-date development environment.


1.  Clone the Microsoft Linux kernel repository from GitHub and switch to the kernel branch corresponding to version 6 (or desired kernel version)

    ```
    git clone https://github.com/microsoft/WSL2-Linux-Kernel.git --depth=1 -b linux-msft-wsl-6.1.y
    ```

2.  Install the necessary packages required for kernel compilation
   
    ```
    sudo apt update && sudo apt install build-essential flex bison libssl-dev libelf-dev
    ```
   
3.  Compile the kernel. This process may take some time
   
    ```
    cd WSL2-Linux-Kernel

    # Copy the WSL-specific configuration
    cp Microsoft/config-wsl .config

    # Compile the kernel with multiple cores for efficiency
    LOCALVERSION= make -j$(nproc)

    # Install kernel modules
    sudo LOCALVERSION= make KCONFIG_CONFIG=Microsoft/config-wsl modules_install -j$(nproc)

    # Install kernel headers
    sudo make headers_install ARCH=x86_64 INSTALL_HDR_PATH=/usr

    # Copy the compiled kernel image to a Windows-accessible location
    cp arch/x86/boot/bzImage /mnt/c/

    ```
       
4. After the installation, verify that the kernel modules are correctly present
   
    ```
    [22:55:39] [~] ❱❱❱ ls /lib/modules
    6.1.21.2-microsoft-standard-WSL2
    ```

## Configuring WSL2 for the New Kernel

1.  Close the Terminal

    Ensure you've closed your current terminal.

2.  Create or Edit the WSL Configuration File

    On your Windows host machine, create or edit the file `%USERPROFILE%\.wslconfig` with the following content

    ```
    [wsl2]
    kernel=C:\\bzImage
    ```

3.  Stop the WSL Instance

    Open PowerShell as Administrator, execute the following command to stop the WSL instance

    ```
    wsl --shutdown
    ```

4.  Check the Kernel Version

    After restarting your WSL instance, you can verify the kernel version with the following command. This command should confirm that you are now using the specified kernel image (`C:\\bzImage`) in your WSL2 environment

    ```
    uname -r
    ```

    !!! Note "Note"
        If you encounter a "No such file or directory" error when trying to access '/sys/kernel/debug/tracing/', you can resolve this issue by running the following command:

    ```
    mount -t debugfs none /sys/kernel/debug/
    ```

    This command mounts the debug filesystem (`debugfs`) to the '/sys/kernel/debug/' directory, enabling proper access to debugging information required for working with eBPF and other tracing tools. Once this command is executed, you should be able to access debugging features without encountering the error.

## Test with bpftrace

Let's proceed with your first attempt using BPF. You can use bpftrace to test and trace events. Bpftrace is a tool that builds a simplified tracing language on top of eBPF and BCC, allowing you to implement complex tracing functions with just a few lines of simple scripts.

1.  Install bpftrace using the following command

    ```
    sudo apt-get install bpftrace
    ```

2.  Listing All Kernel Instruments and Tracepoints

    You can query all kernel instruments and tracepoints using the following command. 
       
    ```
    sudo bpftrace -l
    ```

    This command will provide a list of available tracing points and instruments in the kernel, allowing you to choose the events you want to trace.

3.  Using bpftrace to Trace System Call Execution and Display Input Parameters

    To track the execution of a system call and display its input parameters, you can use the following bpftrace script

    ```
    sudo bpftrace -e 'tracepoint:syscalls:sys_enter_execve {join(args->argv);}'
    ```

    This script will trace the `execve` system call and display its input arguments, providing insights into the executed processes and their parameters.

## bpftool and libbpf

Let's proceed with the installation of bpftool and libbpf, along with the necessary dependencies such as libbfd, clang, llvm, and libcap.

### Installing llvm and clang

First, let's install the latest versions of llvm and clang. Please note that these commands allow you to install **all available versions** of clang and llvm. It's not necessary to install all of them. 

!!! Tip "Tip"
    You can refer to [Automatic installation script](https://apt.llvm.org/) if you want to install a specific version directly.

```
wget https://apt.llvm.org/llvm.sh
sudo bash ./llvm.sh all
```

Next, we'll define the latest version of clang and llvm using the `update-alternatives` method. We'll use version 17. You can use the [provided script](https://gist.github.com/junkdog/70231d6953592cd6f27def59fe19e50d) for this purpose:

```bash title="update-alternatives-clang.sh"
#!/usr/bin/env bash

#     --slave /usr/bin/$1 $1 /usr/bin/$1-\${version} \\

function register_clang_version {
    local version=$1
    local priority=$2

    update-alternatives \
       	--verbose \
        --install /usr/bin/llvm-config          llvm-config          /usr/bin/llvm-config-${version} ${priority} \
        --slave   /usr/bin/llvm-ar              llvm-ar              /usr/bin/llvm-ar-${version} \
        --slave   /usr/bin/llvm-as              llvm-as              /usr/bin/llvm-as-${version} \
        --slave   /usr/bin/llvm-bcanalyzer      llvm-bcanalyzer      /usr/bin/llvm-bcanalyzer-${version} \
        --slave   /usr/bin/llvm-c-test          llvm-c-test          /usr/bin/llvm-c-test-${version} \
        --slave   /usr/bin/llvm-cat             llvm-cat             /usr/bin/llvm-cat-${version} \
        --slave   /usr/bin/llvm-cfi-verify      llvm-cfi-verify      /usr/bin/llvm-cfi-verify-${version} \
        --slave   /usr/bin/llvm-cov             llvm-cov             /usr/bin/llvm-cov-${version} \
        --slave   /usr/bin/llvm-cvtres          llvm-cvtres          /usr/bin/llvm-cvtres-${version} \
        --slave   /usr/bin/llvm-cxxdump         llvm-cxxdump         /usr/bin/llvm-cxxdump-${version} \
        --slave   /usr/bin/llvm-cxxfilt         llvm-cxxfilt         /usr/bin/llvm-cxxfilt-${version} \
        --slave   /usr/bin/llvm-diff            llvm-diff            /usr/bin/llvm-diff-${version} \
        --slave   /usr/bin/llvm-dis             llvm-dis             /usr/bin/llvm-dis-${version} \
        --slave   /usr/bin/llvm-dlltool         llvm-dlltool         /usr/bin/llvm-dlltool-${version} \
        --slave   /usr/bin/llvm-dwarfdump       llvm-dwarfdump       /usr/bin/llvm-dwarfdump-${version} \
        --slave   /usr/bin/llvm-dwp             llvm-dwp             /usr/bin/llvm-dwp-${version} \
        --slave   /usr/bin/llvm-exegesis        llvm-exegesis        /usr/bin/llvm-exegesis-${version} \
        --slave   /usr/bin/llvm-extract         llvm-extract         /usr/bin/llvm-extract-${version} \
        --slave   /usr/bin/llvm-lib             llvm-lib             /usr/bin/llvm-lib-${version} \
        --slave   /usr/bin/llvm-link            llvm-link            /usr/bin/llvm-link-${version} \
        --slave   /usr/bin/llvm-lto             llvm-lto             /usr/bin/llvm-lto-${version} \
        --slave   /usr/bin/llvm-lto2            llvm-lto2            /usr/bin/llvm-lto2-${version} \
        --slave   /usr/bin/llvm-mc              llvm-mc              /usr/bin/llvm-mc-${version} \
        --slave   /usr/bin/llvm-mca             llvm-mca             /usr/bin/llvm-mca-${version} \
        --slave   /usr/bin/llvm-modextract      llvm-modextract      /usr/bin/llvm-modextract-${version} \
        --slave   /usr/bin/llvm-mt              llvm-mt              /usr/bin/llvm-mt-${version} \
        --slave   /usr/bin/llvm-nm              llvm-nm              /usr/bin/llvm-nm-${version} \
        --slave   /usr/bin/llvm-objcopy         llvm-objcopy         /usr/bin/llvm-objcopy-${version} \
        --slave   /usr/bin/llvm-objdump         llvm-objdump         /usr/bin/llvm-objdump-${version} \
        --slave   /usr/bin/llvm-opt-report      llvm-opt-report      /usr/bin/llvm-opt-report-${version} \
        --slave   /usr/bin/llvm-pdbutil         llvm-pdbutil         /usr/bin/llvm-pdbutil-${version} \
        --slave   /usr/bin/llvm-PerfectShuffle  llvm-PerfectShuffle  /usr/bin/llvm-PerfectShuffle-${version} \
        --slave   /usr/bin/llvm-profdata        llvm-profdata        /usr/bin/llvm-profdata-${version} \
        --slave   /usr/bin/llvm-ranlib          llvm-ranlib          /usr/bin/llvm-ranlib-${version} \
        --slave   /usr/bin/llvm-rc              llvm-rc              /usr/bin/llvm-rc-${version} \
        --slave   /usr/bin/llvm-readelf         llvm-readelf         /usr/bin/llvm-readelf-${version} \
        --slave   /usr/bin/llvm-readobj         llvm-readobj         /usr/bin/llvm-readobj-${version} \
        --slave   /usr/bin/llvm-rtdyld          llvm-rtdyld          /usr/bin/llvm-rtdyld-${version} \
        --slave   /usr/bin/llvm-size            llvm-size            /usr/bin/llvm-size-${version} \
        --slave   /usr/bin/llvm-split           llvm-split           /usr/bin/llvm-split-${version} \
        --slave   /usr/bin/llvm-stress          llvm-stress          /usr/bin/llvm-stress-${version} \
        --slave   /usr/bin/llvm-strings         llvm-strings         /usr/bin/llvm-strings-${version} \
        --slave   /usr/bin/llvm-strip           llvm-strip           /usr/bin/llvm-strip-${version} \
        --slave   /usr/bin/llvm-symbolizer      llvm-symbolizer      /usr/bin/llvm-symbolizer-${version} \
        --slave   /usr/bin/llvm-tblgen          llvm-tblgen          /usr/bin/llvm-tblgen-${version} \
        --slave   /usr/bin/llvm-undname         llvm-undname         /usr/bin/llvm-undname-${version} \
        --slave   /usr/bin/llvm-xray            llvm-xray            /usr/bin/llvm-xray-${version}
        

    update-alternatives \
       	--verbose \
        --install /usr/bin/clang					clang					/usr/bin/clang-${version} ${priority} \
        --slave   /usr/bin/clang++					clang++					/usr/bin/clang++-${version}  \
        --slave   /usr/bin/clang-format				clang-format			/usr/bin/clang-format-${version}  \
        --slave   /usr/bin/clang-cpp				clang-cpp				/usr/bin/clang-cpp-${version} \
        --slave   /usr/bin/asan_symbolize			asan_symbolize			/usr/bin/asan_symbolize-${version} \
        --slave   /usr/bin/bugpoint					bugpoint				/usr/bin/bugpoint-${version} \
        --slave   /usr/bin/dsymutil					dsymutil				/usr/bin/dsymutil-${version} \
        --slave   /usr/bin/lld						lld						/usr/bin/lld-${version} \
        --slave   /usr/bin/ld.lld					ld.lld					/usr/bin/ld.lld-${version} \
        --slave   /usr/bin/lld-link					lld-link				/usr/bin/lld-link-${version} \
        --slave   /usr/bin/llc						llc						/usr/bin/llc-${version} \
        --slave   /usr/bin/lli						lli						/usr/bin/lli-${version} \
        --slave   /usr/bin/obj2yaml					obj2yaml				/usr/bin/obj2yaml-${version} \
        --slave   /usr/bin/opt						opt						/usr/bin/opt-${version} \
        --slave   /usr/bin/sanstats					sanstats				/usr/bin/sanstats-${version} \
        --slave   /usr/bin/verify-uselistorder		verify-uselistorder		/usr/bin/verify-uselistorder-${version} \
        --slave   /usr/bin/wasm-ld					wasm-ld					/usr/bin/wasm-ld-${version} \
        --slave   /usr/bin/yaml2obj					yaml2obj					/usr/bin/yaml2obj-${version}
        
}

register_clang_version $1 $2
```

```
sudo bash ./update-alternatives-clang.sh 17 100
```

After running the script, clang and llvm should be properly set up. You can check their versions using the following commands:

```
clang --version
llvm-strip --version
```

### Installing libcap and libbfd

Now, let's install libcap and libbfd along with their development packages

```
sudo apt-get install -y libcap-dev libbfd-dev
```

### Installing libbpf

We also need to install libbpf

```
sudo apt-get install libbpf-dev
```

In case you encounter issues with this library, you may need to compile and install it manually. To do so, follow these steps

```
git clone https://github.com/libbpf/libbpf.git
cd ./libbpf/src
make
sudo make install
```

### Installing bpftool

Finally, to install bpftool, we need to compile and install it.  (i.e., during the build, all four dependencies should be green)

```
git clone --recurse-submodules https://github.com/libbpf/bpftool.git
cd src
make
sudo make install
```

Congratulations! Your installation should now be complete, and you can start developing with in Python with BCC and C using bpftool.

---
- [https://learn.microsoft.com/en-us/community/content/wsl-user-msft-kernel-v6](https://learn.microsoft.com/en-us/community/content/wsl-user-msft-kernel-v6)
- [https://gist.github.com/junkdog/70231d6953592cd6f27def59fe19e50d](https://gist.github.com/junkdog/70231d6953592cd6f27def59fe19e50d)
- [https://www.cnblogs.com/hartmon/p/15935212.html](https://www.cnblogs.com/hartmon/p/15935212.html)
- [https://github.com/microsoft/WSL2-Linux-Kernel](https://github.com/microsoft/WSL2-Linux-Kernel)
- [https://github.com/libbpf/libbpf/](https://github.com/libbpf/libbpf/)
- [https://github.com/iovisor/bcc/blob/master/INSTALL.md#wslwindows-subsystem-for-linux---binary](https://github.com/iovisor/bcc/blob/master/INSTALL.md#wslwindows-subsystem-for-linux---binary)
- [https://apt.llvm.org/]([https://apt.llvm.org/)
- [https://oftime.net/2021/01/16/win-bpf/]([https://oftime.net/2021/01/16/win-bpf/)
- [https://blog.csdn.net/sinat_22338935/article/details/123002910](https://blog.csdn.net/sinat_22338935/article/details/123002910)
- [https://blog.csdn.net/sinat_22338935/article/details/123005213](https://blog.csdn.net/sinat_22338935/article/details/123005213)