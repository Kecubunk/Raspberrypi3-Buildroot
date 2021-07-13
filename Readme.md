# Raspberry Pi3 B+ Image Buildroot Documentation

## 1. Create directory in home

```
mkdir ~/pi3
cd ~/pi3
```
## 2. Get buildroot inside the created directory

```
git clone git://git.buildroot.net/buildroot
cd buildroot-2020.02.2

                            OR 
git clone https://github.com/Joomlax/Raspberry-Pi3-B-Buildroot.git
tar xvf buildroot-2020.02.2.tar.gz
cd buildroot-2020.02.2
```
## 3. Configure and compile the build tool in the system

```
make raspberrypi3_64_defconfig
You can select this version from the /buildroot/board
```

### 4.1 You can face with some missing libraries at **./configure --enable-local** part. Here is the package installation commands: 

```
sudo apt install libtool-bin libtool-doc
sudo apt install help2man
sudo apt install gperf bison flex texinfo
```

## 5. Setup the make configs for the build root for wifi working

```
make menuconfig
Target Packages -> Networking Applications -> openssh, client, server, key utilities
```

## 6. Generate the root filesystem. It takes 20 to 40 minutes

```
make -j8
```


