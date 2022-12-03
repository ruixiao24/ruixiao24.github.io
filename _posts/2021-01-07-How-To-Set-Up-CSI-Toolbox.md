---
layout: post
title: "How to set up CSI Tool"
date: 2021-01-07 14:57:00 -0000
categories: WiFi
---

I spent about one week to configure the [Linux 802.11n CSI Tool](https://dhalperi.github.io/linux-80211n-csitool/) on both my desktop and laptop. So it’s better for me to write it down in case I need to do that again.

## Key points

- The CSI Tool is expected to work on Linux operating systems that are based on an upstream Linux kernel version between 3.2 (e.g Ubuntu 12.04) and 4.2 (e.g. Ubuntu 14.04.4). So, do not install Ubuntu 14.04.6.
- You may not be able to boot your laptop after installing the Intel 5300 NIC on your laptop (ThinkPad X201 for me). In that case, you need to flash the bios first. (Plug out the NIC before entering PE environment.)

## Environment

- ThinkPad X201 (or any Intel PC with PCIE socket)
- Intel 5300 Wi-Fi NIC
- Ubuntu 14.04

## Connecting to GitHub with SSH

Using the SSH protocol, you can connect and authenticate to remote servers and services. While cloning the CSI repository, I find it’s more stable to use SSH protocol instead of HTTPS.

1. Generating a new SSH key

   Paste the text below, substituting in your GitHub email address

   ```bash
   ssh-keygen -t ed25519 -C "your_email@example.com"
   ```

2. Adding your SSH key to the SSH-agent

   Ensure the ssh-agent is running. You can use the "Auto-launching the ssh-agent" instructions in "[Working with SSH key passphrases](https://docs.github.com/en/free-pro-team@latest/articles/working-with-ssh-key-passphrases)", or start it manually:

   ```bash
   # start the ssh-agent in the background
   $ eval `ssh-agent -s`
   > Agent pid 59566
   ```

3. Add your SSH private key to the ssh-agent. If you created your key with a different name, or if you are adding an existing key that has a different name, replace *id_ed25519* in the command with the name of your private key file.
4. [Add the SSH key to your GitHub account](https://docs.github.com/en/free-pro-team@latest/articles/adding-a-new-ssh-key-to-your-github-account).

## Installation Steps

1. Finish step 1,2,3,4 on [Linux 802.11n CSI Tool](https://dhalperi.github.io/linux-80211n-csitool/) 

   ```bash
   # step 1
   sudo apt-get install gcc make linux-headers-$(uname -r) git-core
   
   # step 2
   CSITOOL_KERNEL_TAG=csitool-$(uname -r | cut -d . -f 1-2)
   git clone https://github.com/dhalperi/linux-80211n-csitool.git
   cd linux-80211n-csitool
   git checkout ${CSITOOL_KERNEL_TAG}
   
   make -C /lib/modules/$(uname -r)/build M=$(pwd)/drivers/net/wireless/iwlwifi modules
   # step 3
   git clone https://github.com/dhalperi/linux-80211n-csitool-supplementary.git
   for file in /lib/firmware/iwlwifi-5000-*.ucode; do sudo mv $file $file.orig; done
   sudo cp linux-80211n-csitool-supplementary/firmware/iwlwifi-5000-2.ucode.sigcomm2010 /lib/firmware/
   sudo ln -s iwlwifi-5000-2.ucode.sigcomm2010 /lib/firmware/iwlwifi-5000-2.ucode
   
   # step 4
   make -C linux-80211n-csitool-supplementary/netlink
   ```

2. Install libgcypt11_1.5.3-2ubuntu4_amd64.deb. *Libgcrypt* is a general purpose cryptographic library originally based on code from GnuPG.

3. Install aircrack0ng_1.1-6_amd64.deb. *Aircrack-ng* is a complete suite of tools to assess WiFi network security.

4. Follow the steps in [dhalperi/linux-80211n-csitool-supplementary/injection](https://github.com/dhalperi/linux-80211n-csitool-supplementary/tree/master/injection) to set up injection mode:

   - Install LORCON

     ```bash
     git clone https://github.com/dhalperi/lorcon-old.git
     ```

   - Compile and make LORCON

5. Set up receiver/transmitter
   ```bash
   # How to use
   sudo airmon-ng check kill
   sudo service network-manager stop
   
   # receiver:
   	./setup_monitor_csi.sh 64 HT20
   	sudo ../netlink/log_to_file ~/Desktop/log.dat
   
   # transmitter:
   	./setup_inject.sh 64 HT20
   	# Then:
   	echo 0x4101 | sudo tee `find /sys -name monitor_tx_rate`
   	sudo ./random_packets 1 100 1
   	# OR:
   	echo 0x1c113 | sudo tee `sudo find /sys -name monitor_tx_rate`
   	sudo ./random_packets 1000000000 100 1 10000
   	# First Param: Total number of packets.
   	# Last Param: Relay time (in ms)
   ```

5. If your can receive packets, it means installation success.

6. If it doesn’t work, use the `setup_monitor_csi.sh` in appendix.

## System test

- Collect data

  ```bash
  # Receiver
  sudo ../netlink/log_to_file > filename.dat
  ```

## Appendix

*setup_monitor_csi.sh*

```bash
#!/usr/bin/sudo /bin/bash
rfkill unblock all
service network-manager stop #restart
SLEEP_TIME=2
WLAN_INTERFACE=$1
if [ "$#" -ne 3 ]; then
    echo "Going to use default settings!"
    chn=64
    bw=HT20
else
    chn=$2
    bw=$3
fi
echo "Bringing $WLAN_INTERFACE down....."
ifconfig $WLAN_INTERFACE down
while [ $? -ne 0 ]
do
    ifconfig $WLAN_INTERFACE down
done
sleep $SLEEP_TIME
echo "Set $WLAN_INTERFACE into monitor mode....."
iwconfig $WLAN_INTERFACE mode monitor
while [ $? -ne 0 ]
do
    iwconfig $WLAN_INTERFACE mode monitor
done
sleep $SLEEP_TIME
echo "Bringing $WLAN_INTERFACE up....."
ifconfig $WLAN_INTERFACE up
while [ $? -ne 0 ]
do
    ifconfig $WLAN_INTERFACE up
done
sleep $SLEEP_TIME
echo "Set channel $chn $bw....."
iw $WLAN_INTERFACE set channel $chn $bw

```

*setup_inject.sh*

```bash
#!/usr/bin/sudo /bin/bash
service network-manager stop
WLAN_INTERFACE=$1
SLEEP_TIME=2
modprobe iwlwifi debug=0x40000
if [ "$#" -ne 3 ]; then
    echo "Going to use default settings!"
    chn=64
    bw=HT20
else
    chn=$2
    bw=$3
fi
sleep $SLEEP_TIME
ifconfig $WLAN_INTERFACE 2>/dev/null 1>/dev/null
while [ $? -ne 0 ]
do
    ifconfig $WLAN_INTERFACE 2>/dev/null 1>/dev/null
done
sleep $SLEEP_TIME
echo "Add monitor mon0....."
iw dev $WLAN_INTERFACE interface add mon0 type monitor
sleep $SLEEP_TIME
echo "Bringing $WLAN_INTERFACE down....."
ifconfig $WLAN_INTERFACE down
while [ $? -ne 0 ]
do
    ifconfig $WLAN_INTERFACE down
done
sleep $SLEEP_TIME
echo "Bringing mon0 up....."
ifconfig mon0 up
while [ $? -ne 0 ]
do
    ifconfig mon0 up
done
sleep $SLEEP_TIME
echo "Set channel $chn $bw....."
iw mon0 set channel $chn $bw
```



## Reference

[1] Caichao’s blog: [https://blog.csdn.net/caichao08/article/details/53894510](https://blog.csdn.net/caichao08/article/details/53894510)

[2] Linux 802.11n CSI tool Monitor模式: [https://blog.csdn.net/u014645508/article/details/82993718](https://blog.csdn.net/u014645508/article/details/82993718)
