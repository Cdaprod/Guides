### Ethernet Gadget Configuration

#### Configure the Zero
1. **Select MAC Addresses**:
   - Each interface must have a unique MAC address. Recommended format: `12:34:56:78:9A:BC` and `12:34:56:78:9A:BD`.

2. **Using the g_ether Module**:
   - **Why**: It's easy and does most of the setup for you.
   - **Why Not**: Deprecated and only supports the Ethernet gadget.
   - **How**:
     1. Backup your `config.txt` and `cmdline.txt`:
 
        ```bash
        sudo cp /boot/config.txt /boot/config.bak
        sudo cp /boot/cmdline.txt /boot/cmdline.bak
        ```
     2. Edit `/boot/config.txt` to include:
 
        ```plaintext
        dtoverlay=dwc2,dr_mode=peripheral
        ```
     3. Edit `/boot/cmdline.txt` to include:
 
        ```plaintext
        modules-load=dwc2,g_ether g_ether.dev_addr=12:34:56:78:9A:BC g_ether.host_addr=12:34:56:78:9A:BD
        ```
     4. Save changes and reboot.

#### Basic Network Configuration
1. **Picking an IP Address Range**:
   - Use private IP ranges like `10.0.0.0/24` to avoid conflicts.

2. **Configuring an IP Address**:
   - **Using `dhcpcd.conf`**:
     1. Backup and edit `dhcpcd.conf`:
 
        ```bash
        sudo cp /etc/dhcpcd.conf /etc/dhcpcd.bak
        sudo nano /etc/dhcpcd.conf
        ```
     2. Add the following:
 
        ```plaintext
        interface usb0
        static ip_address=10.0.0.1/24
        ```
     3. Repeat on the USB host with a different IP:
 
        ```plaintext
        interface usb0
        static ip_address=10.0.0.2/24
        ```
     4. Reboot both devices.

### ethernet-configfs.sh (Shell Script for libcomposite)

The provided script `ethernet-configfs.sh` uses `libcomposite` for finer control over the gadget.

#### Example Script
Here’s a breakdown of a typical `ethernet-configfs.sh` script:

1. **Load libcomposite**:
 
   ```bash
   modprobe libcomposite
   ```

2. **Create directories and set IDs**:
 
   ```bash
   mkdir -p /sys/kernel/config/usb_gadget/mygadget
   cd /sys/kernel/config/usb_gadget/mygadget
   echo 0x1d6b > idVendor
   echo 0x0104 > idProduct
   echo 0x0100 > bcdDevice
   echo 0x0200 > bcdUSB
   ```

3. **Configure text strings**:
 
   ```bash
   mkdir -p strings/0x409
   echo "1234567890" > strings/0x409/serialnumber
   echo "me" > strings/0x409/manufacturer
   echo "My USB Device" > strings/0x409/product
   ```

4. **Initial device configuration**:
 
   ```bash
   mkdir -p configs/c.1/strings/0x409
   echo "Config 1: ECM network" > configs/c.1/strings/0x409/configuration
   echo 250 > configs/c.1/MaxPower
   ```

5. **Configure the Ethernet gadget**:
 
   ```bash
   mkdir -p functions/ecm.usb0
   echo "12:34:56:78:9A:BC" > functions/ecm.usb0/host_addr
   echo "12:34:56:78:9A:BD" > functions/ecm.usb0/dev_addr
   ln -s functions/ecm.usb0 configs/c.1/
   ```

6. **Finish up**:
 
   ```bash
   ls /sys/class/udc > UDC
   ```

7. **Save the script and make it executable**:
 
   ```bash
   chmod a+x /path/to/ethernet-configfs.sh
   ```

8. **Call it from root’s crontab or `/etc/rc.local`**.

By following these steps, you should be able to set up your Raspberry Pi Zero as a USB Ethernet gadget, configure networking, and handle fallback scenarios with access points.