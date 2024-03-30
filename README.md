# :dog2: bonbon



## Hardware

* [Raspberry Pi 5](https://www.raspberrypi.com/products/raspberry-pi-5/), 8GB
* [PSU for Raspberry Pi 5](https://www.raspberrypi.com/products/27w-power-supply/) (27W)
* [Raspberry Pi Active Cooler](https://www.raspberrypi.com/products/active-cooler/)
* [Pimoroni NVMe Base](https://shop.pimoroni.com/products/nvme-base?variant=41219587178579)
* [PCIe Pipe 50mm](https://shop.pimoroni.com/products/pcie-flex-cable-for-nvme-base-and-raspberry-pi-5?variant=41449370943571)
* [Crucial P3 Plus 1TB PCIe 3 NVMe SSD](https://uk.crucial.com/products/ssd/crucial-p3-plus-ssd)
* [Pimoroni Pibow case](https://shop.pimoroni.com/products/pibow-5?variant=41045302575187)
* Display ([Pi-Top FHD Touch](https://www.pi-top.com/start/touch-display))
* Keyboard ([Logitech K400](https://www.logitech.com/en-gb/products/keyboards/k400-plus-touchpad-keyboard.html))

<br />

## Operating System

1. Install the OS.

   * **Operating System**: Raspberry Pi OS (Bookworm, 64-bit)
   * **Hostname**:  `bonbon`
   * **Username**: `pi`
   * **Password**: `let***n!***`

2. Configure the Raspberry Pi to boot from the NVMe SSD.

   See https://learn.pimoroni.com/article/getting-started-with-nvme-base#setting-the-raspberry-pi-to-boot-from-the-nvme-ssd

<br />

## Swap

Disabling, enabling and modifying Raspberry Pi's default swap on the boot microSD Card or SSD.

### Enable swap file service

1. Check if the swapfile service is enabled.

   ```bash
   systemctl is-enabled dphys-swapfile
   ```

   If the output is `disabled`, enable it with the command:

   ```bash
   sudo systemctl enable dphys-swapfile
   ```

2. Check if the swapfile service is active/running.

   ```bash
   systemctl is-active dphys-swapfile
   ```

   If the output is `inactive`, start it with the command:

   ```bash
   sudo systemctl start dphys-swapfile
   ```

### Increase/decrease swap size

1. Turn the swap off.

   ```bash
   sudo dphys-swapfile swapoff
   ```

2. Open and edit the configuration file.

   ```bash
   sudo nano /etc/dphys-swapfile
   ```

   Search for the following line in the file (e.g. use the **Ctrl**+**W** shortcut to search in-file) and edit the swap size. The size must given in megabytes, and the specified size must be available on the boot microSD card/SSD.

   ```bash
   CONF_SWAPSIZE=100
   ```

   e.g. increase the swapsize to 1GB:

   ```bash
   CONF_SWAPSIZE=1024
   ```

   e.g. increase the swapsize to 2GB:

   ```bash
   CONF_SWAPSIZE=2048
   ```

3. Reinitialize the swap file.

   ```bash
   sudo dphys-swapfile setup
   ```

   The command will delete the original/previous swap file and recreate it with the new size.

4. Turn the swap on.

   ```bash
   sudo dphys-swapfile swapon
   ```

5. Reboot.

   ```bash
   sudo reboot
   ```

### Disable swap file service

1. Turn the swap off.

   ```bash
   sudo dphys-swapfile swapoff
   ```

2. Uninstall the swap file.

   ```bash
   sudo dphys-swapfile uninstall
   ```

3. Stop the swap file service.

   ```bash
   sudo systemctl stop dphys-swapfile
   ```

4. Disable the swap file service.

   ```bash
   sudo systemctl disable dphys-swapfile
   ```

5. Reboot.

   ```bash
   sudo reboot
   ```

<br />

## Docker

https://docs.docker.com/engine/install/debian/

1. Add Docker's GPG key.

   ```bash
   sudo apt update
   sudo apt install ca-certificates curl
   # If keyrings isn't already installed.
   #sudo install -m 0755 -d /etc/apt/keyrings
   sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
   sudo chmod a+r /etc/apt/keyrings/docker.asc
   ```

2. Add Docker's repository to `apt` sources.

   ```bash
   echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
   ```

3. Update the`apt` list.

   ```bash
   sudo apt update
   ```

4. Install Docker.

   ```bash
   sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
   ```

   > To install a specific version:
   >
   > 1. List the available versions.
   >
   >    ```bash
   >    apt-cache madison docker-ce | awk '{ print $3 }'
   >    
   >    5:25.0.0-1~debian.12~bookworm
   >    5:24.0.7-1~debian.12~bookworm
   >    ...
   >    ```
   >
   > 2. Select the desired version and install.
   >
   >    ```bash
   >    VERSION_STRING=5:25.0.0-1~debian.12~bookworm
   >    sudo apt-get install docker-ce=$VERSION_STRING docker-ce-cli=$VERSION_STRING containerd.io docker-buildx-plugin docker-compose-plugin

5. Verify that the installation is successful by running the example `hello-world` image.

   ```bash
   sudo docker run hello-world
   ```

   The output will look like this:

   ```bash
   Hello from Docker!
   This message shows that your installation appears to be working correctly.
   
   To generate this message, Docker took the following steps:
    1. The Docker client contacted the Docker daemon.
    2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
       (arm64v8)
    3. The Docker daemon created a new container from that image which runs the
       executable that produces the output you are currently reading.
    4. The Docker daemon streamed that output to the Docker client, which sent it
       to your terminal.
   
   To try something more ambitious, you can run an Ubuntu container with:
    $ docker run -it ubuntu bash
   
   Share images, automate workflows, and more with a free Docker ID:
    https://hub.docker.com/
   
   For more examples and ideas, visit:
    https://docs.docker.com/get-started/
   ```

   Once successful installation of Docker is confirmed, remove the example container.

   First, get the ID of the example container:

   ```bash
   sudo docker ps -a
   CONTAINER ID   IMAGE         COMMAND    CREATED         STATUS                     PORTS     NAMES
   a025fe0a6fd0   hello-world   "/hello"   1 minutes ago   Exited (0) 1 minutes ago             funny_keller
   ```

   Remove the container:

   ```bash
   sudo docker rm a025fe0a6fd0
   ```

6. Add the Raspberry Pi user to `docker` group so that Docker command can be run without `sudo`.

   ```bash
   sudo usermod -a pi -G docker
   ```

   This will come into effect on next reboot, but for now, add the user to the new group manually:

   ```bash
   newgrp docker
   ```

<br />

## Graphical User Interface (GUI)

### Disable/enable GUI temporarily

* To disable, press the **Ctrl**+**Alt**+**F1** keys to enter console mode, and then run the command:

  ```bash
  sudo systemctl isolate multi-user.target
  ```

* To enable, run the following command, and then press the **Ctrl**+**Alt**+**F1** keys to switch to Desktop.

  ```bash
  sudo systemctl isolate graphical.target
  ```

### Disable/enable GUI by default

* To disable, press the **Ctrl**+**Alt**+**F1** keys to enter console mode, and then run the commands:

  ```bash
  sudo systemctl set-default multi-user.target
  sudo reboot
  ```

* To enable, run the commands:

  ```bash
  sudo systemctl set-default graphical.target
  sudo reboot
  ```

<br />

## VSCodium

https://vscodium.com/

1. Add VSCodium's GPG key.

   ```bash
   wget -qO - https://gitlab.com/paulcarroty/vscodium-deb-rpm-repo/raw/master/pub.gpg | gpg --dearmor | sudo dd of=/usr/share/keyrings/vscodium-archive-keyring.gpg
   ```

2. Add Docker's repository to `apt` sources.

   ```bash
   echo 'deb [ signed-by=/usr/share/keyrings/vscodium-archive-keyring.gpg ] https://download.vscodium.com/debs vscodium main' | sudo tee /etc/apt/sources.list.d/vscodium.list
   ```

3. Update the`apt` list.

   ```bash
   sudo apt update
   ```

4. Install VSCodium.

   ```bash
   sudo apt install codium
   ```

