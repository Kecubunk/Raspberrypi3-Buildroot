# Raspberry Pi3 B+ Image Buildroot + Wifi connection Documentation

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

### 4. Make configures

```
System configuration -> /dev management -> Dynamic using devtmpfs + mdev 
Target packages -> Hardware handling -> Firmware -> rpi-wifi-firmware
Target packages -> Networking applications -> wpa_supplicant
Target packages -> Networking applications -> wpa_supplicant -> Enable nl80211 support 
Target packages -> Networking applications -> wpa_supplicant -> Install wpa_passphrase binary (for the making wireless pass)
Target packages -> Networking applications -> wireless.tools (for the iwconfig iwlist process)
Target packages -> Networking applications -> dehydrated -> all dropdown selections (specially dhcp)
Target Packages -> Networking Applications -> openssh, client, server, key utilities
```
## 5. Edit the files of buildroot for wifi enabling

```
cd /buildroot/board/raspberrypi
touch interfaces
nano interfaces
auto lo
iface lo inet loopback
auto eth0
iface eth0 inet dhcp
    pre-up /etc/network/nfs_check
    wait-delay 15
auto wlan0
iface wlan0 inet dhcp
    pre-up wpa_supplicant -B -Dnl80211 -iwlan0 -c/etc/wpa_supplicant.conf
    post-down killall -q wpa_supplicant
    wait-delay 15
iface default inet dhcp
```

```
cd /buildroot/board/raspberrypi
touch wpa_supplicant.conf
wpa_passphrase <WIFI ESSID> <WIFI PASSWORD>
copy this codes output and paste it wpa_supplicant.conf

It should look like:
network={
    ssid="SSID"
    #psk="PASSWORD"
    psk=XXX
}
```

```
cd /buildroot/board/raspberrypi/post-build.sh
ADD FOLLOWING LINES TO THE END OF FILE:
cp package/busybox/S10mdev ${TARGET_DIR}/etc/init.d/S10mdev
chmod 755 ${TARGET_DIR}/etc/init.d/S10mdev
cp package/busybox/mdev.conf ${TARGET_DIR}/etc/mdev.conf
cp board/raspberrypi3/interfaces ${TARGET_DIR}/etc/network/interfaces
cp board/raspberrypi3/wpa_supplicant.conf ${TARGET_DIR}/etc/wpa_supplicant.conf
```

## 6. Make the buildroot

```
make -j8
```

## 7. When buildroot boots

It will automatically connected to the wifi of created wpa_passphrase. You can check with **ifconfig** command and you can see wlan0 has inet addr:xx.xx.xx.xx. If it not correctly connected:

First kill the process of connection
``` ps ax | grep "wpa_supplicant -B" | grep -v grep ```
this will give you the pid of connection as outpu
``` kill <output pid> ```
First up the wlan0
``` ifconfig wlan0 up ```
Then scan wifi with wlan0
``` iwlist wlan0 scan | grep ESSID ```
this will give you the ESSIDs of the wifi.
Then create a passphrase and write it to the wpa_supplicant.conf
``` wpa_passphrase <WIFI ESSID> <WIFI PASS> | tee /etc/wpa_supplicant.conf ```
Then connect with the wpa_supplicant
``` wpa_supplicant -c /etc/wpa_supplicant.conf -i wlan0 ```
If this connection correct than make it works at the background
``` wpa_supplicant -B -c /etc/wpa_supplicant.conf -i wlan0```

Congratulations, Enjoy the buildroot connected to wifi :)

