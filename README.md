# Quickstart guide to get ROS2 running on your Raspberry Pi SBC

For this guide, I will be using Raspberry Pi 3B, but you should be able to use Raspberry Pi 2, 3 or 4 or any other SBC in this class
We will also go thru a few additional steps to set up a microcontroller to publish data to our ROS2 system.

## Step 1: Installing the OS
I recommend using Ubuntu which has a fantastic ROS and Raspberry Pi support as well as works great on many other SBCs. We will be using the server version of the OS (sorry, no UI) so that we can limit the computing horsepower requirements. As a bonus, we will setup the visualizer on your computer!
If do no have another computer or you really need to run UI on the Raspberry Pi, I recommend using Ubuntu Mate.

- The latest Ubuntu Bionic LTS images can be found on http://cdimage.ubuntu.com/releases/18.04.3/release/ ; for convinience, [here's the link](http://cdimage.ubuntu.com/releases/18.04.3/release/ubuntu-18.04.3-preinstalled-server-arm64+raspi3.img.xz) to downlaod the arm64 (not armhf) preinstalled server image for Raspberry Pi 3
- Use instructions [here](https://ubuntu.com/download/iot/installation-media) to burn the image to your SD card. If you are using Ubuntu, this image can also be burned using the gnome-disk-utility (aka **disks**).

## Step 2: Gimme internet!
There are a few ways we can get get started to setup internet on the Raspberry Pi. The default user for the Ubuntu image is ubuntu, the password too is ubuntu.
1. Using a Keyboard, HDMI monitor
-- On boot, login as user **ubuntu** and password **ubuntu**
-- Ubuntu 18.04 uses a new method to setup network from CLI, YAML! 
-- Config is located in `/etc/netplan/` , lets edit `sudo vim /etc/netplan/50-cloud-init.yaml`, it will look something like this -
  
```  
# This file is generated from information provided by
# the datasource.  Changes to it will not persist across an instance.
# To disable cloud-init's network configuration capabilities, write a file
# /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg with the following:
# network: {config: disabled}
network:
    version: 2
    ethernets:
        eth0:
            dhcp4: true
            match:
                macaddress: YO:UR:PI:SM:AC:E0
            set-name: eth0
```
As you can see, our wired etherent is already setup and set to use dhcp. If you are going to use a wired network, then you are all set. Simply plugin the network cable and you should be all set.

-- If you do want to use wifi instead, edit the config as below -
```  
# This file is generated from information provided by
# the datasource.  Changes to it will not persist across an instance.
# To disable cloud-init's network configuration capabilities, write a file
# /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg with the following:
# network: {config: disabled}
network:
    version: 2
    ethernets:
        eth0:
            dhcp4: true
            match:
                macaddress: YO:UR:PI:SM:AC:E0
            set-name: eth0
    wifis:
        wlan0:
                dhcp4: true
                access-points:
                        "your_wifi_ssid":
                                password: "your_wifi_password"            
```
-- After editing the file run command `sudo netpaln apply` to apply config.

2. Over ssh
-- With the above true, once you plugin your Pi to a wired network, you should get a DHCP IP; you can login to your router and check this IP. 
-- Once you have the IP, use steps in (1) to set up the wifi.

3. Over a serial console
-- Here are a couple of guides - [Instructables](https://www.instructables.com/id/Raspberry-Pi-Serial-Console/) [eLinux](https://elinux.org/RPi_Serial_Connection)
-- Similar to above, once you are logged in, use steps in (1) to set up the wifi.

## Step 2 Bonus:
Accessing the Pi over local network is cool, but how about getting to it over the internet?!
[ZeroTier](https://www.zerotier.com/) lets you setup a secure p2p netowork. You can readup more on its docs on [here](https://www.zerotier.com/manual/).

1. Create a free account for yourself
2. Setup a network and copy the network id
3. Install zerotier on your computer using instructions here 
4. Install zerotier on your raspberry pi by running `curl -s https://install.zerotier.com | sudo bash`
5. Run `sudo zerotier-cli join your_network_id`on your pi
6. On the dashboard you will see your pi trying to join this network, approve it.
7. Done, you can now ping the zerotier ip from your computer running zerotier from anywhere in the world!

## Step 3: Install ROS2

