#Raspberry Pi3 B+ Image Buildroot Documentation

1. Create directory in home

```
mkdir ~/pi3
cd ~/pi3
```

2. Get buildroot inside the created directory

```
wget -c https://buildroot.org/downloads/buildroot-2020.02.2.tar.gz
tar xvf buildroot-2020.02.2.tar.gz
cd buildroot-2020.02.2
```
3. Clone the cross compile repo to the file system

```
git clone https://github.com/crosstool-ng/crosstool-ng.git
cd crosstool-ng/
```

4. Configure and compile the build tool in the system

```
./bootstrap
./configure --enable-local
make -j8
```

4.1 You can face with some missing libraries at **./configure --enable-local** part. Here is the package installation commands: 

```
sudo apt install libtool-bin libtool-doc
sudo apt install help2man
sudo apt install gperf bison flex texinfo
```

5. Configure the build tool (inside of _crosstool-ng_)

```
./ct-ng list-samples
```

This comman will show list of some packages.

```
./ct-ng aarch64-rpi3-linux-gnu
./ct-ng menuconfig
```

6. Time to configure the cross-ng

```
> Paths and misc options -> Maximum log level -> DEBUG.
> Operating System -> Version Of Linux -> 4.19.105.
> Debug Facilities -> Disable all options in there.
```

7. Build toolchain this will take some time.

```
./ct-ng build
```

8. Toolchain installed successfully now add to path to the env. variables in **.bashrc**

```
echo 'export PATH=$PATH:$HOME/x-tool/aarch64-rpi3-linux-gnu/bin/' >> ~/.bashrc
source ~/.bashrc
```

9. Setup the make configs for the build root

```
make menuconfig
> Target options -> Target Architecture -> Aarch64 (Little endian)
> Toolchain -> Toolchain type -> External toolchain. The overall build time will be > reduced
> Toolchain -> Toolchain -> Custom toolchain
> Toolchain -> Toolchain path. And enter the path to the toolchain. **Step 4** Ex: /home/USERNAME/x-tools/aarch64-rpi3-linux-gnu
> Toolchain -> Toolchain has SSP support
> Toolchain -> Toolchain has RPC support
> Toolchain -> Toolchain prefix -> $(ARCH)-rpi3-linux-gnu
> Toolchain -> External toolchain kernel headers series -> 4.19.x or later.
> Toolchain -> External toolchain C library -> glibc/eglibc
> Target packages -> Networking applications -> dropbear
> System configuration -> Root password. _Enter the new password._ 
```

9.1 If you get this error:

```
date.c:(.text.date_main+0x21c): undefined reference to `stime'
collect2: error: ld returned 1 exit status
Note: if build needs additional libraries, put them in CONFIG_EXTRA_LDLIBS.
```

```
> curl https://www.nayab.xyz/patches/0003-compile-error-fix-stime.patch --output ~/rpi3/buildroot-2020.02.2/package/busybox/0003-compile-error-fix-stime.patch
```

10. Generate the root filesystem

```
make -j8
```

11. Extract the root filesystem

```
mkdir -p ~/pi3/nfs_tmp
tar -C ~/pi3/nfs_tmp -xvf ~/pi/buildroot-2020.02.2/output/images/rootfs.tar
```
12. Install Linux kernel modules

```
cd ~/pi3/linux
export ARCH=arm64
export CROSS_COMPILE=aarch64-rpi3-linux-gnu-
export PATH=$PATH:~/x-tools/aarch64-rpi3-linux-gnu/bin/
make modules_install INSTALL_MOD_PATH=~/rpi3/nfs_tmp/
```



