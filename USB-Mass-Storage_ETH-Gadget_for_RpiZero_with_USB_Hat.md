Yes, you can configure your Raspberry Pi Zero to act as both a USB Mass Storage device and an Ethernet Gadget simultaneously. This can be achieved by using the `configfs` method to configure the USB gadget. Here’s a detailed guide on how to set this up:

### Step-by-Step Guide

#### 1. Enable ConfigFS

1. **Edit the `/boot/config.txt` file** to enable the `dwc2` overlay:
   ```bash
   sudo nano /boot/config.txt
   ```
   Add the following line at the end of the file:
   ```plaintext
   dtoverlay=dwc2
   ```

2. **Edit the `/boot/cmdline.txt` file** to load the `dwc2` module:
   ```bash
   sudo nano /boot/cmdline.txt
   ```
   Add `modules-load=dwc2` after `rootwait` (ensure everything is on a single line):
   ```plaintext
   rootwait modules-load=dwc2
   ```

#### 2. Create and Configure the USB Gadget

1. **Create a shell script to set up the USB gadget**:
   ```bash
   sudo nano /usr/bin/usb-gadget.sh
   ```

   Add the following content to the script:
   ```bash
   #!/bin/bash

   # Load necessary kernel modules
   modprobe libcomposite
   cd /sys/kernel/config/usb_gadget/
   
   # Create gadget directory
   mkdir -p g1
   cd g1

   # Set vendor and product ID
   echo 0x1d6b > idVendor  # Linux Foundation
   echo 0x0104 > idProduct  # Multifunction Composite Gadget

   # Set device attributes
   echo 0x0100 > bcdDevice  # v1.0.0
   echo 0x0200 > bcdUSB  # USB2

   # Set English locale strings
   mkdir -p strings/0x409
   echo "fedcba9876543210" > strings/0x409/serialnumber
   echo "YourCompany" > strings/0x409/manufacturer
   echo "Raspberry Pi Zero" > strings/0x409/product

   # Create configuration
   mkdir -p configs/c.1/strings/0x409
   echo "Config 1: ECM network" > configs/c.1/strings/0x409/configuration
   echo 250 > configs/c.1/MaxPower

   # Add Mass Storage function
   mkdir -p functions/mass_storage.usb0
   echo 1 > functions/mass_storage.usb0/stall
   echo 0 > functions/mass_storage.usb0/lun.0/removable
   echo /piusb.bin > functions/mass_storage.usb0/lun.0/file

   # Add ECM (Ethernet) function
   mkdir -p functions/ecm.usb0
   echo "12:34:56:78:9A:BC" > functions/ecm.usb0/host_addr
   echo "12:34:56:78:9A:BD" > functions/ecm.usb0/dev_addr

   # Bind functions to configuration
   ln -s functions/mass_storage.usb0 configs/c.1/
   ln -s functions/ecm.usb0 configs/c.1/

   # Enable the USB gadget
   ls /sys/class/udc > UDC
   ```

2. **Make the script executable**:
   ```bash
   sudo chmod +x /usr/bin/usb-gadget.sh
   ```

3. **Run the script at boot**:
   - Add the script to the `/etc/rc.local` file so it runs at boot:
     ```bash
     sudo nano /etc/rc.local
     ```
   - Add the following line before `exit 0`:
     ```bash
     /usr/bin/usb-gadget.sh
     ```

#### 3. Prepare the Filesystem Image

1. **Create a filesystem image**:
   ```bash
   dd if=/dev/zero of=/piusb.bin bs=1M count=512
   mkfs.vfat /piusb.bin
   ```

2. **Mount the image and add files**:
   ```bash
   sudo mkdir /mnt/usb
   sudo mount -o loop /piusb.bin /mnt/usb
   sudo cp /path/to/your/files/* /mnt/usb/
   sudo umount /mnt/usb
   ```

#### 4. Reboot and Test

- **Reboot your Raspberry Pi Zero** to apply the changes:
  ```bash
  sudo reboot
  ```

- **Connect the Raspberry Pi Zero** to your host computer using the USB male hat. The Pi should be recognized as both a mass storage device and an Ethernet gadget.

### Verifying the Setup

1. **Check USB Mass Storage**:
   - On your host computer, verify that the Raspberry Pi Zero appears as a USB drive.
   
2. **Check USB Ethernet Gadget**:
   - Verify that the Ethernet gadget appears as a network interface on your host computer.
   - You can configure network settings as needed on both the Raspberry Pi Zero and the host computer to establish a network connection.

By following these steps, you can successfully configure your Raspberry Pi Zero to function as both a USB Mass Storage device and an Ethernet gadget simultaneously.