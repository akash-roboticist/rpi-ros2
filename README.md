# Quickstart guide to get ROS2 running on your Raspberry Pi SBC and using your computer to visualize

For this guide, I will be using Raspberry Pi 3B, but you should be able to use Raspberry Pi 2, 3 or 4 or any other SBC in this class
We will also go thru a few additional steps to set up a microcontroller to publish data to our ROS2 system.

## Step 1: Installing the OS
I recommend using Ubuntu which has a fantastic ROS and Raspberry Pi support as well as works great on many other SBCs. We will be using the server version of the OS (sorry, no UI) so that we can limit the computing horsepower requirements. As a bonus, we will setup the visualizer on your computer!
If do no have another computer or you really need to run UI on the Raspberry Pi, I recommend using Ubuntu Mate.

- The latest Ubuntu Bionic LTS images can be found on http://cdimage.ubuntu.com/releases/18.04.3/release/ ; for convinience, [here's the link](http://cdimage.ubuntu.com/releases/18.04.3/release/ubuntu-18.04.3-preinstalled-server-arm64+raspi3.img.xz) to downlaod the arm64 (not armhf) preinstalled server image for Raspberry Pi 3
- Use instructions [here](https://ubuntu.com/download/iot/installation-media) to burn the image to your SD card. If you are using Ubuntu, this image can also be burned using the gnome-disk-utility (aka **disks**).

## Step 2: Gimme Internet
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

## Step 2 Bonus: Free P2P VPN!
Accessing the Pi over local network is cool, but how about getting to it over the internet?!
[ZeroTier](https://www.zerotier.com/) lets you setup a secure p2p netowork. You can readup more on its docs on [here](https://www.zerotier.com/manual/).

1. Create a free account for yourself
2. Setup a network and copy the network id
3. Install zerotier on your computer using instructions here 
4. Install zerotier on your raspberry pi by running `curl -s https://install.zerotier.com | sudo bash`
5. Run `sudo zerotier-cli join your_network_id`on your pi
6. On the dashboard you will see your pi trying to join this network, approve it.
7. Done, you can now ping the zerotier ip from your computer running zerotier from anywhere in the world!

## Step 3: Install ROS2 on Raspberry Pi
For this setup we will use the latest stable release of ROS2 - [Eloquent Elusor](https://index.ros.org/doc/ros2/Releases/Release-Eloquent-Elusor/). Though Dashing is planned to have a [longer support](https://index.ros.org/doc/ros2/Releases/) than Eloquent, the (feature updates and changes)[https://index.ros.org/doc/ros2/Releases/Release-Eloquent-Elusor/#id2], makes it a better choice to get started. 

Official install instructions for Eloquent are at https://index.ros.org/doc/ros2/Installation/Eloquent/ , lets get started. We will install from [debains](https://index.ros.org/doc/ros2/Installation/Eloquent/Linux-Install-Debians/).

For ease of install, here are all the commands you need to run -
```
# Setup locale
sudo locale-gen en_US en_US.UTF-8
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
export LANG=en_US.UTF-8

# Setup sources
sudo apt update && sudo apt install curl gnupg2 lsb-release
curl -s https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc | sudo apt-key add -

sudo sh -c 'echo "deb [arch=amd64,arm64] http://packages.ros.org/ros2/ubuntu `lsb_release -cs` main" > /etc/apt/sources.list.d/ros2-latest.list'

# Install ROS2 base packages (since we will not be running RViz on the Raspberry PI, no need to install the full desktop paskages)
sudo apt update
sudo apt install ros-eloquent-ros-base
source /opt/ros/eloquent/setup.bash

# Install argcomplete (optional, but recommended)
sudo apt install python3-argcomplete
```

Add the line `source /opt/ros/eloquent/setup.bash` to your `.bashrc` so that each of your terminal instance automatically sources the setup files.
```
sudo echo "source /opt/ros/eloquent/setup.sh" >> ~/.bashrc
```

Thats it! Your Raspberry Pi now has ROS2 Eloquent Elusor.


If you would like to try out a couple of quick demos, install the demo package
```
sudo apt install ros-eloquent-demo-nodes-py
```

and run the listner node in one terminal 
```
ros2 run demo_nodes_py listener
```

and the talker node in another.
```
ros2 run demo_nodes_py talker
```

You would see outputs on both terminals
```
user@ubuntu-pi:~$ ros2 run demo_nodes_py listener
[INFO] [listener]: I heard: [Hello World: 0]
[INFO] [listener]: I heard: [Hello World: 1]
[INFO] [listener]: I heard: [Hello World: 2]
[INFO] [listener]: I heard: [Hello World: 3]
[INFO] [listener]: I heard: [Hello World: 4]
[INFO] [listener]: I heard: [Hello World: 5]
```
```
user@ubuntu-pi:~$ ros2 run demo_nodes_py talker
[INFO] [talker]: Publishing: "Hello World: 0"
[INFO] [talker]: Publishing: "Hello World: 1"
[INFO] [talker]: Publishing: "Hello World: 2"
[INFO] [talker]: Publishing: "Hello World: 3"
[INFO] [talker]: Publishing: "Hello World: 4"
[INFO] [talker]: Publishing: "Hello World: 5"
```

Noticed something odd? Yep, there is no roscore; ROS2 – ending the ROS “slave trade”! ROS 2 is built on top of DDS/RTPS as its middleware, which provides discovery, serialization and transportation. You can read more about it [here](https://index.ros.org/doc/ros2/Concepts/DDS-and-ROS-middleware-implementations/); in a nutshell, you dont need a central ROS master anymore, messages are passed over DDS/RTPS and are available to the entire network. 
This does raise questions on security, will my data be accessible to the entire network I am on? Yes; unless you use the Authentication / Access control / Cryptographic plugins , you can read up more on that [here](https://design.ros2.org/articles/ros2_dds_security.html)

## Step 4: Install ROS2 on your workstation
Similar to the Raspberry Pi, we will use Ubuntu 18.04 for the workstation. You can install this on your coputer natively (Recommended, but be careful, YOU MIGHT ERASE YOUR COMPUTER IN DUE PROCESS. YOU HAVE BEEN WARNED!). Else you can setup a Virtual Machine using VirtualBox / VM Ware Player (less risk, but sacrifices on performance).

- Ubuntu install images are available at http://releases.ubuntu.com/18.04/ ; here's a [direct download link](http://releases.ubuntu.com/18.04/ubuntu-18.04.3-desktop-amd64.iso) for 18.04.3 desktop amd64
- [OSBOXES team publishes](https://www.osboxes.org/ubuntu/#ubuntu-1804-vbox) ready to use VMs for Virtualbox and VMWare Player, just in case you dont want to install the OS yourself.

(Optional) Setup a ZeroTier client on this system too if you are intending to the P2P VPN.
In case, you are planning to use a virtual machine, and are not installing ZeroTier, do set the network adapter on your virtual machine in "Bridge mode" instead of "NAT", that way you get a "direct link" into your VM to the local network.


The steps for ROS2 installation are practically same as for the Raspberry Pi, the only change being, instead of installing `ros-eloquent-ros-base` we will install `ros-eloquent-ros-desktop` ; a condensed version of the commands are below -

```
# Setup locale
sudo locale-gen en_US en_US.UTF-8
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
export LANG=en_US.UTF-8

# Setup sources
sudo apt update && sudo apt install curl gnupg2 lsb-release
curl -s https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc | sudo apt-key add -

sudo sh -c 'echo "deb [arch=amd64,arm64] http://packages.ros.org/ros2/ubuntu `lsb_release -cs` main" > /etc/apt/sources.list.d/ros2-latest.list'

# Install ROS2 desktop packages
sudo apt update
sudo apt install ros-eloquent-ros-desktop
sudo source /opt/ros/eloquent/setup.bash
sudo echo "source /opt/ros/eloquent/setup.sh" >> ~/.bashrc

# Install argcomplete (optional, but recommended)
sudo apt install python3-argcomplete
```

Done! You now have a functioning ROS2 workstation. 

## Step 5: Publish from Raspberry Pi to Workstation
Let's see if we are able to publish between the Raspberry Pi and our Workstation.
- On the Raspberry Pi, run the listner demo `user@ubuntu-pi:~$ ros2 run demo_nodes_py listener`
- On the Workstation, run the talker demo `user@ubuntu-desktop:~$ ros2 run demo_nodes_py talker`
You should see data on both terminals!
```
user@ubuntu-desktop:~$ ros2 run demo_nodes_py talker
[INFO] [talker]: Publishing: "Hello World: 0"
[INFO] [talker]: Publishing: "Hello World: 1"
[INFO] [talker]: Publishing: "Hello World: 2"
[INFO] [talker]: Publishing: "Hello World: 3"
[INFO] [talker]: Publishing: "Hello World: 4"
[INFO] [talker]: Publishing: "Hello World: 5"
[INFO] [talker]: Publishing: "Hello World: 6"
[INFO] [talker]: Publishing: "Hello World: 7"
[INFO] [talker]: Publishing: "Hello World: 8"
[INFO] [talker]: Publishing: "Hello World: 9"
[INFO] [talker]: Publishing: "Hello World: 10"
```
```
user@ubuntu-pi:~$ ros2 run demo_nodes_py listener
[INFO] [listener]: I heard: [Hello World: 0]
[INFO] [listener]: I heard: [Hello World: 1]
[INFO] [listener]: I heard: [Hello World: 2]
[INFO] [listener]: I heard: [Hello World: 3]
[INFO] [listener]: I heard: [Hello World: 4]
[INFO] [listener]: I heard: [Hello World: 5]
[INFO] [listener]: I heard: [Hello World: 6]
[INFO] [listener]: I heard: [Hello World: 7]
[INFO] [listener]: I heard: [Hello World: 8]
[INFO] [listener]: I heard: [Hello World: 9]
[INFO] [listener]: I heard: [Hello World: 10]
```

Thats it, you have a functioning distributed ROS2 setup!

### Additional notes:
Install colcon so that we can build packages for ROS2
```
sudo apt install python3-colcon-common-extensions
```


## Step 5 Bonus: Write a subscriber to read data from a microcontroller and publish to our ROS2 network [WIP]

## Step 6: Visualize IMU data in rviz2 [WIP]
