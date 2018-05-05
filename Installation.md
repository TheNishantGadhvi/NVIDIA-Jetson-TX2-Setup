# NVIDIA Jetson TX2 Setup with VM Virtual box  5.2.8 r121009 on Macbook Pro 10.13.4, High Sierra
If your TX2, like mine, came without an operating system, you will have no luck booting into a native Linux environment. In any case, NVIDIA suggests updating JetPack to the latest version.

So let's get a Linux environment, JetPack, Cuda, and a few other items installed.

# JetPack 3.2 Installation

First of all, much credit is owed to Klein Yuan [on Github](https://github.com/KleinYuan/tx2-flash) for his tutorial. This walk through will build off of that. Also, [here](https://youtu.be/D7lkth34rgM) is an awesome walkthrough on Youtube of the setup and install of JetPack, but some of the details were missing for a VM (hence why I wrote this!)

I will be using a MacBook Pro(running 10.13.4, High Sierra) with VirtualBox running Ubuntu 16.04 to flash the TX2.

The entire process takes about 2 hours(depends on connection/data transfer rate)

##### 1. Get VirtualBox to run a Linux Environment

Since the Jetson TX2 must be flashed with a Linux environment, and ideally 14.04 Ubuntu(I did using 16.04), we will need to create a VM with Linux running.

Download VirtualBox [here](https://www.virtualbox.org/), or use VMWare if you're on Windows. With VirtualBox, you're also going to need the *Extension Package* to use USB 2.0/3.0. This can be directly downloaded [here](http://download.virtualbox.org/virtualbox/5.2.2/Oracle_VM_VirtualBox_Extension_Pack-5.2.2-119230.vbox-extpack)

Once you have VirtualBox, we need to get a boot image for Linux.

##### 2. Download Linux Boot Image
Since we need to use Linux Ubuntu 14.04, let's grab that from the [Linux Downloads page.](http://releases.ubuntu.com/14.04/ or http://releases.ubuntu.com/16.04/)

[This link](http://releases.ubuntu.com/14.04/ubuntu-14.04.5-desktop-amd64.iso) will directly download the `.iso` file for 64-bit desktop machines. I have used this [Link](http://releases.ubuntu.com/16.04/ubuntu-16.04.4-desktop-amd64.iso) to download the 16.04 release.

##### 3. Setup VM
With the Linux Boot Image downloaded, we need to setup the VM and boot into it. 

Select the *New* Icon wheel in the upper left of VirtualBox. Fill in the necessary information. You should select at least 2+ GB RAM, and 32GB HD.

Go to Settings --> Network --> Adapter 1, change Attached to to Bridged Adapter, and in the Name select the one with Nvidia Tegra. Also, select the tab Adapter 2, check the box of Enable Network Adapter, change Attached to to Nat (and nothing to do with Name here.) Adapter 1 will help communicate host with Jetson and Adapter 2 will give internet to the host Virtual Machine from the Mac.  

Now launch the VM you just created. At this point, if it hasn't already, VirtualBox will ask you which boot image it should use. Navigate to your `.iso` file, and select it.

##### 4. Setup Linux

Once the Linux environment comes up, it will ask if you want to try out Linux, or do a full install. It worked for me to do a full installation. Select that option, and walk through the Linux Install.

##### 5. Get JetPack

Once inside the Linux Environment, get NVIDIA JetPack [here](https://developer.nvidia.com/embedded/jetpack) *(at the time of this writing it was version 3.2)*. You're going to need to setup a developer account with them, if you don't have one already!

Once this is downloaded, take the file and put it in a directory like `~/documents/JetPack/`

##### 6. Wire the NVIDIA Jetson TX2

With the Jetson shutdown and unplugged from power, plug in the:
* HDMI port to a monitor
* Ethernet port to a router you are connected to on your Mac(can be wired or wireless - I used wireless, but ran into a small problem which I provide a solution to later on)
* Micro USB port with the provided USB cable, and connect that to your Mac - which we call the *Host*
* USB port to a hub(ideally) with mouse and keyboard. If not, just to a mouse or keyword and switch when needed
* Antennas that came with the Jetson
* Finally, the power cable to the nearest wall socket.

##### 7. Put the NVIDIA Jetson TX2 into Force Recovery Mode

From idle mode(with a single red LED illuminated on the Jetson), put the Jetson into Forced recovery mode, by following these steps:
* Press and release the Power Button (furthest to the Right, of the 4 buttons onboard which says PWR on the PCB)
* Hold down the Forced Recovery button (says 'REC' on the pcb)
* With REC held down, press and release the reset button (says 'RST' on the pcb)
* Hold REC for 2 more seconds, and release

To verify you've done this correctly, open up a terminal and list the usb devices with one of these commands:
```sh
$ lsusb
$ ioreg -p IOUSB 
$ ioreg -p IOUSB -w0 -l
$ ioreg -p IOUSB -w0 | sed 's/[^o]*o //; s/@.*$//' | grep -v '^Root.*'
```
You should see `NVIDIA Corp. APX`

##### 8. Add NVIDIA to VirtualBox USB Settings
Now, with your VM shutdown, open up the settings of your Linux VM:

Go to Settings --> Ports --> USB, Enable USB Controller - USB 3.0 (xHCI) Controller

Select the *plus* with *usb icon* on the right, select `NVIDIA Corp. APX`

Now, launch the VM, and open a new terminal to verify these settings are functioning and the VM can 'see' the Jetson. Type:
```sh
$ lsusb
```
You should see NVIDIA listed.

##### 9. Run and Install JetPack
Now, on the Linux VM, open a new terminal and set the permissions to executable for the file we downloaded earlier:
```sh
$ cd ~/documents/JetPack/
$ chmod +x ./JetPack-L4T-3.1-linux-x64.run
```
Run jetpack:
```sh
$ ./JetPack-L4T-3.1-linux-x64.run
```
This should bring up a GUI. Click through, select the TX2, do a full installation, accept the terms, and click next.

This will download a lot of items onto the VM, and take a bit of time. At one point you will see this screen: ![alt text](https://camo.githubusercontent.com/188eef0b1822afb9a32630b85949a863ca10d085/687474703a2f2f646f63732e6e76696469612e636f6d2f6a65747061636b2d6c34742f325f312f636f6e74656e742f646576656c6f706572746f6f6c732f6d6f62696c652f6a65747061636b2f696d616765732f6a65747061636b5f6c34745f666f7263655f7265636f766572795f6d6f64652e3030315f363030783336342e706e67)

Since we have already put the device into forced recovery mode, we can open a new terminal, verify we can still 'see' the NVIDIA:
```sh
$ lsusb
```
Then, navigate back to the window, and press ENTER, as it indicates.  The installation will continue, and eventually you will see some activity on the monitor connected to your Jetson. 

If everything went well, a Linux Ubuntu desktop should launch.

##### 10. Troubleshooting
To test that you received all of the packages with the Linux environment, try out a sample. Navigate to this directory(if you're like me, you won't even have a CUDA directory installed!):
```
/home/ubuntu/NVIDIA_CUDA-<version>_Samples/bin/armv7l/linux/release/gnueabihf/
```
This rendering of water will be shown:
![alt text](http://docs.nvidia.com/jetpack-l4t/2_1/content/developertools/mobile/jetpack/images/jetpack_cuda_fft_ocean_simulation.001_500x527.png)

So for the rest of us who might have had issues with finding the IP address for the target...

Re-launch jetpack:
```sh
$ sudo ./JetPack-L4T-3.1-linux-x64.run
```
Click through again, but when you get to this screen:

![alt text](http://docs.nvidia.com/jetpack-l4t/2_1/content/developertools/mobile/jetpack/images/jetpack_l4t_component_mgr.003_700x613.png)

Select ***Custom*** install in the upper right corner. And click ***Clear Actions***.

Now, go down the list and click on the Action, and select those in the lower part of the list for the **Target**. Make sure you *deselect* **Flash OS**. Click next.

It will then ask you for your IP address:

![alt text](http://docs.nvidia.com/jetpack-l4t/2_1/content/developertools/mobile/jetpack/images/jetpack_l4t_device_info.003_700x527.png)

At this point we will make a new wired connection in VM. For this, type below command in the terminal of VM:
```sh
$ cd
$ nm-connection-editor
```
This will open up the Network Manager. There click on Add --> Ethernet (it should be under Hardware) and --> Create
Now, in the Ethernet tab --> Device, select enp0s3. Finally, in IPv4 tab --> Method, select Manual and enter the IP: 192.168.55.X (X can be any number but not 1 as it is already taken by Jetson), and Netmask: 255.255.255.0 (This might change to 24, but don't worry about it.). 

At this point, connect the Jetson with your Mac through Ethernet cable. And, dont forget to connect it from Devices --> USB (in VM top menu bar). 
Now, in the window of Jetpack installation, enter the IP: 192.168.55.1, User ID and Password are nvidia. After this click Next and go through the process. 

For Error: "CUDA cannot be installed on device.This may be caused by other apt-get command running on device when installing CUDA.Please use apt-get command in a terminal to make sure following packages are installed correctly on device before continue:cuda-toolkit-8-0 libgomp1 libfreeimage-dev libopenmpi-dev openmpi-bin.After these packages are installed on device,press Enter key to continue." 

Here the issue is nothing but your Jetson was looking for internet to download Cuda and it doesnt have. Don't close this window. We will get back to it. 

Work around here is, connect a display to Jetson though HDMI and connect it to the Wi-Fi. 
Now, from the VM, open a new terminal window and SSH into the jetson using following commands:
```sh
$ ssh nvidia@192.168.55.1 
```
You should be in the Jetson's terminal. Now, we will download all the packages listed in the error. For this run:
```sh
$ sudo apt-get update       # Here, updating will help fixing some missing components and packages for the next command.
$ sudo apt-get install cuda-toolkit-8-0 libgomp1 libfreeimage-dev libopenmpi-dev openmpi-bin
```
Once done here, go back to error window "Installing Cuda on target" and press enter. (I pressed enter thrice and it worked thereafter).
This should take care of everything. And, you should be all set. 

To retrieve IP of the device, open up a terminal on the machine, and type:
```sh
$ hostname -I
```

Type this `192.XXX.X.XX` string into the first box, and your credentials into the next two; which by default are both `nvidia`.

Click next, and you should be good to go!

Go back to the top of this section, **Troubleshooting**, and try the water samples and a few others to be sure you have Cuda!

##### Additional Resources

If you ever have to reflash the target, you can do so directly with:
```sh
$ sudo ./flash.sh jetson-tx2 mmcblk0p1
```

Here are a few other guides from NVIDIA, albeit somewhat limited:
http://docs.nvidia.com/jetpack-l4t/2_1/content/developertools/mobile/jetpack/jetpack_l4t/2.0/jetpack_l4t_install.htm
http://developer2.download.nvidia.com/embedded/L4T/r28_Release_v1.0/BSP/l4t_quick_start_guide.txt





