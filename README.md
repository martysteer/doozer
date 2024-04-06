# :dog2: bonbon



## Hardware

* [Raspberry Pi 5](https://www.raspberrypi.com/products/raspberry-pi-5/), 8GB
* [PSU for Raspberry Pi 5](https://www.raspberrypi.com/products/27w-power-supply/) (27W)
* [Raspberry Pi Active Cooler](https://www.raspberrypi.com/products/active-cooler/)*
* [Pimoroni NVMe Base](https://shop.pimoroni.com/products/nvme-base?variant=41219587178579)
* [PCIe Pipe 50mm](https://shop.pimoroni.com/products/pcie-flex-cable-for-nvme-base-and-raspberry-pi-5?variant=41449370943571)
* [Crucial P3 Plus 1TB PCIe 3 NVMe SSD](https://uk.crucial.com/products/ssd/crucial-p3-plus-ssd)
* [Pimoroni Pibow case](https://shop.pimoroni.com/products/pibow-5?variant=41045302575187)
* Display ([Pi-Top FHD Touch](https://www.pi-top.com/start/touch-display))
* Keyboard ([Logitech K400](https://www.logitech.com/en-gb/products/keyboards/k400-plus-touchpad-keyboard.html))

\* [Argon THRML 60mm Radiator Cooler](https://thepihut.com/products/argon-thrml-60mm-radiator-cooler-for-raspberry-pi-5) may possibly be better cooling solution than the official Active Cooler if using the Pi to generate image/video/audio embeddings.

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

6. Verify.

   ```bash
   free -h
                  total        used        free      shared  buff/cache   available
   Mem:           7.9Gi       642Mi       6.5Gi        45Mi       866Mi       7.2Gi
   Swap:          2.0Gi          0B       2.0Gi
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

Source: https://docs.docker.com/engine/install/debian/

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
   sudo docker run --rm hello-world
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

6. Add the Raspberry Pi user to `docker` group so that Docker command can be run without `sudo`.

   ```bash
   sudo usermod -a pi -G docker
   ```

   This will come into effect on next reboot, but for now, add the user to the new group manually:

   ```bash
   newgrp docker
   ```

### Manage Docker's disk usage

Docker persists build cache, containers, images, and volumes to disk, and over time, these artifacts can build up and take up a lot of space on a system.

| Task                                                         | Command                                                    |
| ------------------------------------------------------------ | ---------------------------------------------------------- |
| View disk usage.                                             | `docker system df`                                         |
| List unused containers.                                      | `docker ps --filter status=exited --filter status=dead -q` |
| Remove all stopped containers.                               | `docker container prune`                                   |
| Remove dangling images.                                      | `docker image prune`                                       |
| Remove anonymous volumes.                                    | `docker volume prune`                                      |
| Remove build cache.                                          | `docker buildx prune`                                      |
| Remove unused networks.                                      | `docker network prune`                                     |
| Remove all unused containers, networks, images (dangling) and build cache (unused). | `docker system prune`                                      |
| :warning: ​​Remove all dangling and unused images.             | `docker image prune -a`                                    |
| :warning: ​Remove all anonymous and unused volumes.           | `docker volume prune -a`                                   |
| :warning: ​Remove all unused containers, networks, and images (both dangling and unused), and build cache (unused). | `docker system prune -a`                                   |
| :warning: ​Remove all unused containers, networks, images (both dangling and unused), volumes (anonymous and unused) and build cache (all). | `docker system prune -a --volumes`                         |

Use the  `-f` or `--force` option with `prune` to mute the prompts for confirmation e.g. `docker image prune -f`

See: https://docs.docker.com/reference/cli/docker/

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

## Pyenv

Source: https://github.com/pyenv/pyenv

1. Install build dependencies.

   ```bash
   sudo apt update; sudo apt install build-essential libssl-dev zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev curl libncursesw5-dev xz-utils tk-dev libxml2-dev libxmlsec1-dev libffi-dev liblzma-dev llvm
   ```

   See: https://github.com/pyenv/pyenv/wiki#suggested-build-environment

2. Install Pyenv.

   ```bash
   curl https://pyenv.run | bash
   ```

3. Set up shell environment for Pyenv.

   Add the following to `~/.bashrc`:

   ```bash
   export PYENV_ROOT="$HOME/.pyenv"
   command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"
   eval "$(pyenv init -)"
   ```

   If there is `~/.bash_profile`, add the following to it:

   ```bash
   export PYENV_ROOT="$HOME/.pyenv"
   [[ -d $PYENV_ROOT/bin ]] && export PATH="$PYENV_ROOT/bin:$PATH"
   eval "$(pyenv init -)"
   ```

   Otherwise, add the following to `~/.profile`:

   ```bash
   export PYENV_ROOT="$HOME/.pyenv"
   command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"
   eval "$(pyenv init -)"
   ```

4. Reload ~/.bashrc settings.

   ```bash
   source ~/.bashrc
   ```

   

<br />

## VSCodium

Source: https://vscodium.com/

1. Add VSCodium's GPG key.

   ```bash
   wget -qO - https://gitlab.com/paulcarroty/vscodium-deb-rpm-repo/raw/master/pub.gpg | gpg --dearmor | sudo dd of=/usr/share/keyrings/vscodium-archive-keyring.gpg
   ```

2. Add VSCodium's repository to `apt` sources.

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

### Use remote Python kernel in VSCodium/Visual Studio Code

Source: https://code.visualstudio.com/docs/datascience/jupyter-kernel-management#_existing-jupyter-server

1. On your Raspberry Pi, start Jupyter Notebook or Jupyter Lab, either on the Pi or in Docker container.

2. On another computer that is on the same network as your Raspberry Pi, open a new or existing notebook in VSCodium or Visual Studio Code.

3. Click the **Select Kernel** button (top right hand of VSCodium/Visual Sudio Code window), and select  **Existing Jupyter Server** (the options will appear in the address bar).

4. Enter the URL of your Jupyter Notebook or Jupyter Lab running on the Raspberry Pi.

5. Enter the password or token for your Jupyter Notebook or Jupyter Lab.

6. For **Change server display name**, accept the given name or delete it.

7. Select **Python 3 (ipykernel)**.

8. Verify by putting the following in a cell and running the cell.

   ```bash
   import os
   os.listdir()
   ```

   This should output a list of directories on Raspberry Pi e.g. in Docker container:

   ```bash
   ['sbin',
    'media',
    'lib',
    'opt',
    'etc',
    'sys',
    'usr',
    'home',
    'run',
    'tmp',
    'boot',
    'srv',
    'root',
    'dev',
    'bin',
    'var',
    'proc',
    'mnt',
    'app',
    '.dockerenv']
   ```

   
