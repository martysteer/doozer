# :telephone: pi5 tty

Configure the raspberry pi /boot/firmware/config.txt file to enable tty access via the GPIO pins.

RPi5 have a built-in UART port and TTY will default to that. To use UART via GPIO instead, you need to divert both the UART and TTY to GPIO with the above settings in the config file. In another words, enabling UART via the raspi-config isn't enough on RPi5.

### Hardware

RPi5 = Doozer Pi5 8GB

SSD case = something like a Sabrent SSD case

USB-to-TTL cable = used an old one I had since before RPi4 times because I couldn't find the newer Adafruit cable.

### And here's what I did

1. Unplug my RPi5 from power supply.
2. Removed SSD M.2 from my RPi5 and put in into a SSD case and connected it to my Mac.
3. Once the SSD was mounted on my Mac, located the file named config.txt in the root of the SSD disk and opened it for editing using the command: ``` nano /Volumes/bootfs/config.txt``` 

Note: This text file is on the root of the SSD disk partition. /Volumes is where MacOS mounts external disks. However, when the pi boots up, the PiOS mounts the disk partition in a different location, under /boot/firmware. So, if your pi is booted and you are connected to it somehow, you can edit/modify the exact same config.txt file by editing what appears to be a different file path. Just mentioning this here because some online instructions do not clarify these differences in your own specific compute environment/setup in their instructions, and assume the way they have connected to the device is enough.

4. Added the following 3 lines at the bottom of the config.txt and saved the changes (copy of bonbon's config.txt attached for reference).

```  
enable_uart=1         # Enable shell messages on the serial connection
dtparam=uart0=on      # Enable UART0/ttyAMA0 on GPIO 14 & 15
dtparam=uart0_console # Enable UART0/ttyAMA0 on GPIO 14 & 15 and make it the console UART
```

Note: RPi5 have built-in UART port and TTY will default to that. To use UART via GPIO instead, you need to divert both the UART and TTY to GPIO with the above settings. In another words, enabling UART isn't enough on RPi5.

5. Ejected the SSD from my Mac and mounted it back on my RPi5 (or rather, the Pimoroni NVMe board). Note: My RPi5 is still switched off.
6. Connected the USB-to-TTL cable as follows:

   -   USB2TTL    RPi5


   -   GRD   ->   GRD


   -   RXD   ->   TXD


   -   TXD   ->   RXD


  The colours of my cables are different and Jeff's diagram may be more helpful here:  https://www.jeffgeerling.com/blog/2021/attaching-raspberry-pis-serial-console-uart-debugging

7. Plugged the USB end of the USB-to-TTL device into my Mac.
8. Checked to see if the USB-to-TTL device is recognised by my Mac by running the command: `ls /dev | grep usb`  ...which output:

 ```bash
 cu.usbserial-0001
 tty.usbserial-0001
 ```

9. Opened a new terminal window on my mac and started a screen session using the command (as per Jeff's instructions): `screen /dev/tty.usbserial-0001 115200`  ...which opened a blank sub-window showing just a cursor.

10. Plugged my RPi5 back into the power supply and booted it.

  ...no visible activity at all in the screen session window on my Mac (unlike Jeff's animated GIF screenshot), but after a minute or so, login prompt appeared, and once I logged in, I ran raspi-config and changed my RPi5's WiFi settings to 'UoL Conference' for testing.

11. Check wifi config information to see if your pi has connected to your wifi network properly: `iwconfig`.

12. Check network IP address information to get your Pis DHCP local IP address: `ifconfig`.

Now you can ssh or vnc into your pi using it’s local IP address!

11. Shut down my RPi with the command: `sudo shutdown -h now`.

12. Logged out of the screen session by pressing Ctrl+A and then Ctrl+K and then y (yes to killing the session). 
13. Disconnected the USB-to-TTL device from my Mac and RPi5.
14. Rebooted my RPi5, and no, it is no longer on my network 🥳
15. Re-did the whole process from the beginning and changed the RPi5's WiFi setting back to my home network, and my RPi5 is back on my network again 🎉

### Ansible playbook to manage the config

If you've got ansible configured to manage your pi, you can invoke the configuration playbook like this:

```bash
ansible-playbook enable-tty-console-UART.yml -vv
```

