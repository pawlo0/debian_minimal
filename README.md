# Debian minimal
To remind myself how to install Debian in external SSD

Required a wired network. Might work under wi-fi, but these instruction most likely will require wired internet.

## 1. Download the Netinst ISO

Download the Debian Netinst ISO. This is a minimal installation image that allows you to install Debian with just the basics.

Go to the Debian [download page](https://www.debian.org/CD/netinst/) and grab the Netinst ISO file.

Next create a bootable USB drive with the ISO file using a tool like Rufus (for Windows) or Etcher (for Linux/Mac).

## 2. Install Debian Minimal

### 2.1 Boot in BIOS

Insert the bootable USB into your computer and restart. Boot from the USB drive by selecting it in the **BIOS**. This is important because whichever the usb is booted it will be the mode Debian will be installed. I need it to be in BIOS.

Start the Debian installation as usual and follow the graphical install.

![graphical install](https://ostechnix.com/wp-content/uploads/2024/09/Start-Debian-Installation.png)

### 2.2 Partitions

When it gets to partitioning, in the disk where Debian is going to be installed, make the following partitions:

1. Primary - Ext4 - 1Gb - Mount on /Boot
2. Primary - Ext4 - (what's left) - Mount on /
3. Primary - swap - 4Gb

### 2.3 Software Selection:

This is the IMPORTANT step. When you get to the "Software Selection" step, deselect everything except "Standard System Utilities."

![software selection](https://ostechnix.com/wp-content/uploads/2024/09/Choose-Standard-System-Utilities.png)

This will ensure you only install the bare minimum.

### 2.4 Complete the Installation:

Follow the remaining prompts to complete the installation. Once done, reboot your system.

## 3. Set network

### 3.1 Set up wired DHPC

May skip this section if network works after reboot. In my case, because I fired the SSD/NVME in a different laptop at this stage, network didn't not work because of different network interfaces names.

After rebooting, log into your system. 

To list all available network interfaces, use the following command:

    ip a

Edit interfaces:

    sudo nano /etc/networks/interfaces

The /etc/networks/interfaces file looks like the following:

```
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
allow-hotplug enp0s3
iface enp0s3 inet dhcp
```

Duplicate both 2 last lines, allow-hotplug and iface, for the new interface’s name. The restart networking.

    sudo systemctl restart networking

## 3.2 Set up wi-fi

The minimal install won't install wi-fi firmwire drivers. Wi-fi won't work unless I download drivers for setting wi-fi. This is the reason I need a wired laptop for all this.

    sudo apt install firmware-iwlwifi

Run the modprobe command to add or remove firmware and driver 

    sudo modprobe -r iwlwifi
    sudo modprobe iwlwifi
    sudo dmesg

Now I can skip to the following section unless I really want wi-fi at this point. Reason is I can set the wi-fi password later in GNOME in NetworkManager. But just in case, here's how to do it:

Finding out Intel Wifi interface name on Debian. We use the ip command:

    ip -c a s
    ip -c link show

I am going to use wpa_supplicant. Should be installed, but just in case:

    sudo apt install wpasupplicant

Next create a new config file:

    sudo sh -c 'wpa_passphrase your_ssid "wifi_password"' > /etc/wpa_supplicant/wpa_supplicant.conf

Edit the /etc/wpa_supplicant/wpa_supplicant.conf using a text editor such as nano, run:

    sudo nano /etc/wpa_supplicant/wpa_supplicant.conf

Update it by appending ctrl_interface and update_config as follows:

    network={
      ssid="your_ssid"
      psk=some_random_encrypted_wifi_password
    }
    ctrl_interface=/run/wpa_supplicant 
    update_config=1

Next, create a new config file for our interface:

    sudo vim /etc/network/interfaces.d/wlp1s0.conf

Append the following config:

    allow-hotplug wlp1s0
    iface wlp1s0 inet dhcp
    wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf

Replace wlp1s0 with my wi-fi interface. Restart services using the systemctl command

    sudo systemctl reenable wpa_supplicant.service
    sudo systemctl restart wpa_supplicant.service
    sudo systemctl restart networking.service

Verify IP address and connectivity using the ping command,

## 4. Install GNOME

### 4.1 Install GNOME Core

This will install the essential parts of GNOME without any extra bloat.

    sudo apt update
    sudo apt install gnome-core -y

### 4.2 Remove ifupdown Package

The Debian installer uses the ifupdown package for network management, but GNOME uses NetworkManager. To avoid conflicts and ensure your WiFi card works, remove ifupdown:

    sudo apt purge ifupdown -y

### 4.3 Lid behaviour

Edit the file at `/etc/systemd/logind.conf`

    HandleLidSwitch=ignore

Make sure the "#" symbol is removed from the start of this line.

### 4.4 Get rid of GRUB menu

When booting, the grub waits 5 secods for a selection. To get rid of that edit the grub file:

    sudo nano /etc/default/grub

Place a `#` symbol at the start of line `GRUB_TIMEOUT=5` to comment it out.

Save changes and run `sudo update-grub` to apply changes.

### 4.5 Reboot Your System

After removing ifupdown, reboot your system:

    sudo shutdown -r now

## 5. Configure NetworkManager

Once you're in GNOME, you'll need to make a small change to the NetworkManager configuration file to ensure it manages your network devices correctly.

#### 5.1 Configure file

Edit the Configuration File:

    sudo nano /etc/NetworkManager/NetworkManager.conf

Change the line `managed=false` to `managed=true`.

Save & exit the editor.

Reboot again:

    sudo shutdown -r now

### 5.2 Configure IP (if needed)

After log in to the system, go to Settings -> Network. Click the gear button next to your network connection and configure the IP address for your network interface if needed.

## 6. Install Additional Tools (Optional)

At this point, we have a barebones GNOME environment. If we want to add more tools like a web browser, git, CLI downloaders, and text editor etc., you can do so by running:

### 6.1 basics

   sudo apt install git wget curl vim -y

### 6.2 Replace firefox by librewolf:

  sudo apt purge firefox-esr -y
  sudo apt update && sudo apt install extrepo -y
  sudo extrepo enable librewolf
  sudo apt update && sudo apt install librewolf -y


