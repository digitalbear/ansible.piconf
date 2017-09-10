# Initial Config for Raspberry Pi

Simple playbook for configuring ssh keys and WiFi for first-time set-up of a new Raspberry Pi device

## While flashing the SD card
### Enable SSH
SSH is no longer enabled by default on your Raspberry Pi (in the latest versions of Raspbian).  You can enable ssh once you are logged on but you would need to connect your Pi to a monitor and keyboard to do this.  In many cases you want to avoid this step and go straight to loging on with ssh.  In order to do this you will need to enable ssh *before* you start up your Pi, while the SD card is still attached to your Macbook (or whatever).  Straight after flashing the SD card, add an empty file to the boot partition, called *ssh* (Do this on your Macbook, not your Pi.  If you are using a Windows machine or Linux, the boot volume of the SD card will be located somewhere else.):
```
cd /Volumes/boot/
touch ssh
```

## Getting the IP address

Insert the SD card and connect the network cable *(unless your SD card has already been configured for WiFi access on your WiFi network you are going to need to connect to the network with a cable initially)*.

If the above method does not work, then you may want to the try the *lsleases* method, as follows

Before you connect the power we want to run a program on your local machine (Macbook or whatever) to retrieve the IP address of your Pi when it connects to the network at startup.  
> When your Pi (or any computer) joins a network it requests, and is usually allocated, an IP address by the DHCP server running on your router.  The IP address is just "leased" out your device for a period of time, usually 24-48 hours.  If you disconnect and reconnect to the network within a day or two you will usually get the same IP address, but any longer than that and you may be "leased" a different address, as your previous address may have been "leased" out to some other device.  

We are going to run the [lsleases](https://github.com/j-keck/lsleases) program to inspect the DHCP network packets on your network.  Refer to the lsleases doumentation on how to install.  Once that is done we are going to run as follows:
```
sudo ./lsleases -s
```
Now connect the power to your Raspberry Pi.  Within half a minute or so (once your Pi has booted) you should see some output from your lsleases program, showing the IP address that has just been allocated to your Pi, something like the following:
```
2017/09/09 11:53:27 startup -  version:  1.4.0-14-gc702188
2017/09/09 11:53:27 enable active check - ping every: 15m
2017/09/09 11:54:11 new DHCP Lease: '192.168.0.4     b8:27:eb:80:c2:dd raspberrypi'
```
In this case the IP adress that has been allocated is **192.168.0.4** - this will be used in the following step to connect to your Pi over SSH.

## Configure SSH
### Add SSH Key
The first thing you should do is enable access using SSH keys and disable password access.  Although neither of these steps are essential, they are recommended for enhanced security.

To create an SSH key pair on your local machine (if you have not already done so):
```
ssh-keygen -t rsa
```
I tend to leave the passphrase blank and go fo rthe default file names.  Once complete this will create two files (a private and a public key) in your SSH directory
> ~/.ssh/id_rsa  
> ~/.ssh/id_rsa.pub


To add your public ssh key to your Raspberry Pi execute the ssh playbook, but first we need to add the IP address (from the step above) to the [hosts](./hosts) file - add the following line to the hosts file (change IP address to whatever you got from the lsleases step):
> *192.168.0.4*  ansible_user=pi ansible_become=yes

Then run:
```sh
ansible-playbook -i hosts ssh.yml --ask-pass
```
> We need to include *--ask-pass* since we have not yet configured an ssh key and will need a password to logon.  This will prompt us to enter in the ssh password for connecting to the remote device.  The default Pi password is *raspberry*.  
This playbook will copy the default public key from the SSH directory (~/.ssh/id_rsa.pub) into the *authorized_keys* file on the remote device.  

> We could have achieved the same thing using *ssh-copy-id* command that comes with openSSL, e.g.:
```
ssh-copy-id -i ~/.ssh/id_rsa pi@192.168.0.4
```

Either way, you should now be able to execute this and other playbooks again, this time without *--ask-pass*
```
ansible-playbook -i hosts ssh.yml
```

### Disable Password Access
*Make sure you can connect over SSH with a public/private key first before disabling password access*

Execute the following playbook to disable password access:
```
ansible-playbook -i hosts ssh-only.yml
```

## Configure Wifi
Execute the following playbook *- this will prompt you for the wifi name (ssid) and password*:
```sh
ansible-playbook -i hosts wifi.yml
```

If you are still logged on to your Pi over ethernet you should be able to see the new wifi IP address - just type in the following command on your Pi:
```
ip a s wlan0
```
*You may need to first reboot your Pi:*
```
sudo reboot
```

If, for some reason you cannot access the Pi remotely over ethernet (network cable) then you might want to try the *lsleases* method: run the following steps (in order)
1) start lsleases program running
2) shutdown your Raspberry Pi (disconnect the power source)
3) disconnect the network cable from the Pi
4) ensure your wifi dongle (USB) is connected (or your Pi has built-in wifi)
5) start up the Pi (connect to a power source again)

Make note of the **new IP Address** - this will usually be different to the one you obtained over the network cable since these use different network interface adapters (e.g. eth0/wlan0)

## Configure VNC

If you download the full Raspbian image (not Rasbian Lite) and plan to connect to your Raspberry Pi desktop remotely you will need to ensure VNC installed and enabled on your Pi.  Use the VNC playbook to do this:
```
ansible-playbook -i hosts vnc.yml
```

You will also need to download and install a VNC viewer on your local machine in order to connect.  Check out the following links for more details: 
* <https://www.realvnc.com/en/connect/download/vnc/>
* <https://www.raspberrypi.org/documentation/remote-access/vnc/>