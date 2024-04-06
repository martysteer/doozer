# :dog2: bonbon x​ ​ImageBind

ImageBind: https://imagebind.metademolab.com | https://arxiv.org/abs/2305.05665

<br />

## Prerequisites

* Increase the Pi's swap size to 2048MB.
* Disable the Pi's GUI before running scripts that use ImageBind.

<br />

## Directory structure

### Raspberry Pi

```bash
/home/pi
├── ...
├── Projects
│   └── playpen
│       └── imagebind            <-- mapped to Docker container's /app/scripts directory
│           ├── example.ipynb
│           ├── processing.ipynb
│           └── search.py
└── ... 
```

### Docker Container

```bash
app
├── .packages                   <-- manually compiled Python dependencies
├── ImageBind                   <-- cloned ImageBind repository
│   ├── .checkpoints            <-- ImageBind looks for this directory when loading the model
│   │   └── imagebind_huge.pth  <-- pre-downloaded ImageBind model
│   └── ...
└── scripts                     <-- mapped to the Pi's ~/Projects/playpen/imagebind directory
    ├── example.ipynb
    ├── processing.ipynb
    └── search.py
```

<br />

## Docker image

Ubuntu 20.04 is being used as the base image because:

- Mayavi, one of ImageBind's dependencies, can be a faff to install on Debian and Alpine Linux; and 
- ImageBind requires Python 3.8.x.

The build process involves pre-downloading the ImageBind model, which takes around 10 minutes. The model is pre-downloaded to avoid potential compute resource issues downstream, and you can skip the step (or comment out in Dockerfile) and download it separately e.g. using ImageBind's method `imagebind_model.imagebind_huge(pretrained=True)` which checks if the model exists already in the expected location (i.e. ImageBind/.checkpoints) and if not, will automatically download it.

### Build with Dockerfile

1. Download the Dockerfile from https://github.com/kunika/bonbon/blob/main/ImageBind/Dockerfile.

2. In the directory where the Dockerfile is located, build the Docker image. Don't forget the dot at the end of the command!

   ```bash
   # In Raspberry Pi's terminal window
   docker build -t imagebind:latest .
   ```

   > **Options**:
   >
   > - `-t` or `--tag`: Name and optionally a tag in the `name:tag` format. 
   > - `--progress` (optional): Type of progress output, which can be `auto`, `plain` or `tty`. Setting this option to `plain` shows container output. Default is  `auto`.
   > - `--no-cache` (optional): Do not use cache when building the image.
   >
   > <br />
   >
   > See: https://docs.docker.com/reference/cli/docker/image/build/

### Build manually

1. Pull Ubuntu 20.04 (Focal Fossa).

   ```bash
   # In Raspberry Pi's terminal window
   docker pull ubuntu:20.04
   ```

2. Start a container and interactive Bash shell.

   ```bash
   # In Raspberry Pi's terminal window
   docker run --rm -it --entrypoint bash ubuntu:20.04
   ```

   > **Options**:
   >
   > - `--rm`: Automatically remove the container when it exits.
   > - `-i` or `--interactive`: Keep STDIN open even if not attached.
   > - `-t` or `--tty`: Allocate a pseudo-TTY. 
   > - `--entrypoint`: Overwrite the default ENTRYPOINT of the image.
   >
   > <br />
   >
   > See: https://docs.docker.com/reference/cli/docker/container/run/

3. Set up directory structure.

   ```bash
   # In the Docker container's shell
   # cwd: /
   mkdir /app
   mkdir /app/.packages
   ```

4. Install pre-requisite packages for installation of ImageBind.

   ```bash
   # In the Docker container's shell
   # cwd: /
   apt update
   apt install -y nano git wget
   apt install -y build-essential make cmake
   apt install -y python3 python3-pip
   ```

5. Install ImageBind's problematic dependencies manually.

   * [Mayavi](https://github.com/enthought/mayavi)

     ```bash
     # In the Docker container's shell
     # cwd: /
     
     # Install mayavi
     apt install -y mayavi2
     
     # Verify installation
     python3
     >>> import mayavi
     >>> print(mayavi.__version__)
     4.7.1
     >>>
     >>> from PyQt5.Qt import PYQT_VERSION_STR
     >>> print(PYQT_VERSION_STR)
     5.14.1
     >>>
     >>> exit()
     ```

   * [GEOS](https://libgeos.org/)

     ```bash
     # In the Docker container's shell
     # cwd: /
     
     # Install the required packages for cartopy
     apt install -y libgeos-dev
     ```
     
   * [Decord](https://github.com/dmlc/decord) (instead of [Eva Decord](https://github.com/georgia-tech-db/eva-decord))

     ```bash
     # In the Docker container's shell
     # cwd: /
     
     # Install the required packages
     apt install -y ffmpeg libavcodec-dev libavfilter-dev libavformat-dev libavutil-dev
     
     # Check ffmpeg version, make sure it is version 4.2 or later
     ffmpeg -version
     
     # Check cmake version. make sure it is verion 3.8 or later
     cmake -version
     
     # Clone the decord repository
     git -C /app/.packages clone --recursive https://github.com/dmlc/decord
     
     # Build the library
     cd /app/.packages/decord
     mkdir build
     cd build
     cmake .. -DUSE_CUDA=0 -DCMAKE_BUILD_TYPE=Release
     make
     
     # Install Python bindings
     cd ../python
     python3 setup.py install --user
     
     # Verify installation
     python3
     >>> import decord
     >>> print(decord.__version__)
     0.6.0
     >>>
     >>> exit()
     ```

6. Install [ImageBind](https://github.com/facebookresearch/ImageBind).

   ```bash
   # In the Docker container's shell
   # cwd: /
   
   # Clone the ImageBind repository
   git -C /app clone https://github.com/facebookresearch/ImageBind
   
   # Edit the requirements file
   # Change the line `eva-decord==0.6.1` to `decord==0.6.0`
   nano /app/ImageBind/requirements.txt
   
   # Install dependencies
   python3 -m pip install /app/ImageBind/.
   
   # Pre-download the model (this will take around 10 minutes)
   mkdir /app/ImageBind/.checkpoints
   wget -P /app/ImageBind/.checkpoints https://dl.fbaipublicfiles.com/imagebind/imagebind_huge.pth
   ```

7. Install [Jupyter Lab/Notebook](https://jupyter.org/).

   - Jupyter Lab

     ```bash
     # In the Docker container's shell
     # cwd: /
     python3 -m pip install jupyterlab ipywidgets
     
     # Set password
     python3 -m jupyter lab password # e.g. raspberry
     ```

   - Jupyter Notebook

     ```bash
     # In the Docker container's shell
     # cwd: /
     python3 -m pip install notebook ipywidgets
     
     # Set password
     python3 -m jupyter notebook password # e.g. raspberry
     ```

8. Save the container as an image.

   1. Open a new terminal window.

   2. Get the ID of the running container:

      ```bash
      # In Raspberry Pi's terminal window
      docker ps -a
      CONTAINER ID   IMAGE          COMMAND   CREATED          STATUS          PORTS   NAMES
      3480abe0e7d3   ubuntu:20.04   "bash"    20 minutes ago   Up 20 minutes           flamboyant_tu
      ```

   3. Create a new image:

      ```bash
      # In Raspberry Pi's terminal window
      docker commit 3480abe0e7d3 imagebind
      ```

   4. Verify:

      ```bash
      # In Raspberry Pi's terminal window
      $ docker image ls
      REPOSITORY    TAG       IMAGE ID       CREATED          SIZE
      imagebind     latest    cce74715fcb6   10 seconds ago   3.02GB
      ubuntu        focal     3048ba078595   5 weeks ago      65.7MB
      hello-world   latest    ee301c921b8a   11 months ago    9.14kB
      ```

9. Return to the terminal window with the Docker container running, exit the container.

   ```bash
   # In the Docker container's shell
   exit
   ```

<br />

## Usage

### Start the Docker container

1. Disable GUI temporarily, if not already disabled by default.

   ```bash
   # In Raspberry Pi's terminal window
   sudo systemctl isolate multi-user.target
   ```

2. Start the Docker container.

   - If the Docker image was built with Dockerfile:

     ```bash
     # In Raspberry Pi's terminal window
     docker run --rm -p 8888:8888 -v ~/Projects/playpen/imagebind:/app/scripts imagebind:latest
     ```

     > **Options**:
     >
     > - `--rm`: Automatically remove the container when it exits.
     > - `-p` or `--publish`: Publish a container's port(s) to the host.
     > - `-v` or `--volume`: Bind mount a volume.
     >
     > <br />
     >
     > See: https://docs.docker.com/reference/cli/docker/container/run/

   - If the Docker image was built manually:

     ```bash
     # In Raspberry Pi's terminal window
     docker run --rm -p 8888:8888 -v ~/Projects/playpen/imagebind:/app/scripts -it --entrypoint bash imagebind:latest -c "jupyter lab --ip=0.0.0.0 --port=8888 --allow-root --no-browser"
     ```

     > **Options**:
     >
     > - `--rm`: Automatically remove the container when it exits.
     > - `-i` or `--interactive`: Keep STDIN open even if not attached.
     > - `-t` or `--tty`: Allocate a pseudo-TTY. 
     > - `--entrypoint`: Overwrite the default ENTRYPOINT of the image. To run a shell command in the container once it is up and running, append the command to execute to `docker run` e.g. `docker run -it --entrypoint bash image_name:image_tag -c whoami`.
     >
     > <br />
     >
     > See: https://docs.docker.com/reference/cli/docker/container/run/

3. Jupyter Lab/Notebook should now be available at the following URLs and accessible from another computer that is connected to the same network as your Raspberry Pi:

   - http://<raspberry_pi_ip_address>:8888 e.g. http://192.168.1.173:8888
   - http://<raspberry_pi_hostname>:8888 e.g. http://bonbon.local:8888

   To find the IP address and/or hostname of your Raspberry Pi, run the following commands on your Pi:
   
   - IP Address: `hostname -I` 
   - Hostname: `hostname`

### Stop the Docker container

1. If you are inside the container, exit the container.

   ```bash
   # In the Docker container's shell
   exit
   ```
   
2. Enable GUI, if not disabled by default (optional).

   ```bash
   # In Raspberry Pi's terminal window
   sudo systemctl isolate graphical.target
   ```

### Run the example notebook

1. Download the example notebook from https://github.com/kunika/bonbon/blob/main/ImageBind/example.ipynb.

2. In Jupyter Lab/Notebook running in the Docker container, navigate to the directory `/app/scripts` and add the example notebook to the directory using Jupyter Lab/Notebook's upload button.

   The `/app` directory in the Docker container (or Jupyter Lab/Notebook's file tree) should now look like this: 

   ```bash
   app
   ├── .packages
   ├── ImageBind
   └── scripts
       └── example.ipynb
   ```

   Note: The `.packages` directory will be hidden (not visible) in Jupyter Lab/Notebook.

<br />

## ChromaDB

ChromaDB: https://www.trychroma.com

### Install with Dockerfile

1. Add the following to the ImageBind Dockerfile, uncomment the commented block after the installation of Jupyter Lab so that it looks like this:

   ```bash
   # Pre-install pesky ChromaDB dependencies
   # SQLite
   # ChromaDB requires SQLite > 3.35 and the version available to Ubuntu 20.04 is 3.31.x,
   # so building a newer version from source instead.
   RUN apt install -y libreadline-dev
   RUN wget -P /app/.packages https://www.sqlite.org/2024/sqlite-autoconf-3450200.tar.gz
   RUN tar xzf /app/.packages/sqlite-autoconf-3450200.tar.gz -C /app/.packages/
   WORKDIR /app/.packages/sqlite-autoconf-3450200
   RUN ./configure && make
   RUN apt purge sqlite3
   RUN make install
   RUN echo 'export LD_LIBRARY_PATH=/usr/local/lib' >> ~/.bashrc
   ENV LD_LIBRARY_PATH="/usr/local/lib"
   # gRPC
   WORKDIR /
   RUN python3 -m pip install grpcio
   
   # Install ChromaDB
   RUN python3 -m pip install chromadb
   ```

2. In the directory where the ImageBind Dockerfile is located, build a new Docker image. Don't forget the dot at the end of the command!

   ```bash
   # In Raspberry Pi's terminal window
   docker build -t imagebind-chromadb:latest .
   ```

### Install manually

1. Start ImageBind Docker container.

   ```bash
   # In Raspberry Pi's terminal window
   docker run --rm -it --entrypoint bash imagebind:latest
   ```

2. Install ChromaDB's problematic dependencies manually.

   - [SQLite](https://www.sqlite.org/)

     ChromaDB requires SQLite > 3.35 and the version available to Ubuntu 20.04 is 3.31.x, so building a newer version from source instead:

     ```bash
     # In the Docker container's shell
     # cwd: /
     
     # Install dependencies
     sudo apt install -y libreadline-dev
     
     # Download the source
     wget -P /app/.packages https://www.sqlite.org/2024/sqlite-autoconf-3450200.tar.gz
     
     # Extract the tarball
     tar xzf /app/.packages/sqlite-autoconf-3450200.tar.gz -C /app/.packages/
     
     # configure and make
     cd /app/.packages/sqlite-autoconf-3450200
     ./configure && make
     
     # Make sure there is no apt managed SQLite installed on the system
     sudo apt purge sqlite3
     
     # Make install
     make install
     
     # Add to path
     echo 'export LD_LIBRARY_PATH=/usr/local/lib' >> ~/.bashrc
     ```

   - [gRPC](https://grpc.io/)

     ```bash
     # In the Docker container's shell
     # cwd: /
     
     # Update pip to the latest version
     python3 -m pip install --upgrade pip
     
     # Install gRPC
     python3 -m pip install grpcio
     ```

3. Install ChromaDB

   ```bash
   # In the Docker container's shell
   # cwd: /
   python3 -m pip install chromadb
   ```

4. Save the container as an image.

   1. Open a new terminal window.

   2. Get the ID of the running container:

      ```bash
      # In Raspberry Pi's terminal window
      docker ps -a
      CONTAINER ID   IMAGE       COMMAND   CREATED          STATUS          PORTS   NAMES
      760fcc5da7ae   imagebind   "bash"    10 minutes ago   Up 10 minutes           loving_driscoll
      ```

   3. Create a new image:

      ```bash
      # In Raspberry Pi's terminal window
      docker commit 760fcc5da7ae imagebind-chromadb
      ```

5. Return to the terminal window with the Docker container running, exit the container.

   ```bash
   # In the Docker container's shell
   exit
   ```

### Start the Docker container

- If the Docker image was built with Dockerfile:

  ```bash
  # In Raspberry Pi's terminal window
  docker run --rm -p 8888:8888 -v ~/Projects/playpen/imagebind:/app/scripts imagebind-chromadb:latest
  ```

- If the Docker image was built manually:

  ```bash
  # In Raspberry Pi's terminal window
  docker run --rm -p 8888:8888 -v ~/Projects/playpen/imagebind:/app/scripts -it --entrypoint bash imagebind-chromadb:latest -c "jupyter lab --ip=0.0.0.0 --port=8888 --allow-root --no-browser"
  ```


<br />

## Troubleshooting

### IPython kernel keeps dying/restarting when loading ImageBind model in Jupyter notebook.

This is most likely caused by the allocated compute resources running out. Try one or more of the following:

- [Resize the swap](../README.md#swap) on your Raspberry Pi. The swap size should be minimum 2GB.
- [Turn off GUI](../README.md#graphical-user-interface-gui) on your Raspberry Pi.
- Pre-download the ImageBind model.
- Restart IPython kernel.
- Restart Jupyter Lab/Notebook.
- Restart the Docker container.
- Reboot your Raspberry Pi.

### Running `import torch` in Jupyter notebook raises 'OSError: /lib/aarch64-linux-gnu/libgomp.so.1: cannot allocate memory in static TLS block'.

No idea what this is or what could be the cause, but one or more of the following seem to make the error go away for now. To investigate at some point...?

* Change the order of imports, and import PyTorch before any other modules e.g.

  ```bash
  import torch
  from imagebind import data as imagebind_data
  from imagebind.models import imagebind_model
  from imagebind.models.imagebind_model import ModalityType
  ```

* Restart IPython kernel.

* Restart Jupyter Lab/Notebook.

* Preload `libgomp.so.1`.

  1. Stop the Docker container, if running.

  2. Confirm `libgomp.so.1` exists in the Docker container:

     ```bash
     # In Raspberry Pi's terminal window
     docker run --rm -it --entrypoint bash imagebind:latest -c "find / -iname libgomp.so.1"
     # Expected output: /usr/lib/aarch64-linux-gnu/libgomp.so.1
     ```

  3. Make sure `libgomp.so.1` loads before Jupyter Lab/Notebook starts by adding its path to `LD_PRELOAD` Environment Variable:

     ```bash
     # In Raspberry Pi's terminal window
     
     # If the Docker image was built with Dockerfile
     docker run --rm -p 8888:8888 -v ~/Projects/playpen/imagebind:/app/scripts -e LD_PRELOAD=/usr/lib/aarch64-linux-gnu/libgomp.so.1 imagebind:latest
     
     # If the Docker image was built manually
     docker run --rm -p 8888:8888 -v ~/Projects/playpen/imagebind:/app/scripts -it --entrypoint bash imagebind:latest -c "LD_PRELOAD=/usr/lib/aarch64-linux-gnu/libgomp.so.1 jupyter lab --ip=0.0.0.0 --port=8888 --allow-root --no-browser"
     ```

  If the error occurs frequently, add the following to your Jupyter Lab/Notebook configuration file `~/.jupyter/jupyter_lab_config.py` or  `~/.jupyter/jupyter_notebook_config.py` as appropriate: 

  ```bash
  import os
  config = get_config()
  os.environ['LD_PRELOAD'] = '/usr/lib/aarch64-linux-gnu/libgomp.so.1'
  config.Spawner.env.update('LD_PRELOAD')
  ```

  If the configuration file does not exist, generate it by running one of the command as appropriate:

  ```bash
  # In the Docker container's shell
  
  # If you have Jupyter Lab installed
  jupyter lab --generate-config
  
  # If you have Jupyter Notebook installed
  jupyter notebook --generate-config
  ```

* Restart the Docker container.

* Reboot your Raspberry Pi.

### All other issues

![unexpected_reboot](../.assets/images/unexpected_reboot.jpg)
