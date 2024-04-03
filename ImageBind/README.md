# :dog2: bonbon x​ ​ImageBind

ImageBind: https://github.com/facebookresearch/ImageBind

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
│       └── imagebind            <-- mapped to Docker image/containers's /app/scripts directory
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
└── scripts                     <-- mapped to the Pi's ~/Projects/Playpen/imagebind directory
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

3. Create `/app` directory.

   ```bash
   # In the Docker container's shell
   # cwd: /
   mkdir /app
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

   * [mayavi](https://github.com/enthought/mayavi)

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

   * [decord](https://github.com/dmlc/decord) (instead of `eva-decord`)

     ```bash
     # In the Docker container's shell
     # cwd: /
     
     # Install the required packages
     apt install -y ffmpeg libavcodec-dev libavfilter-dev libavformat-dev libavutil-dev
     
     # Check ffmpeg version, make sure it is version 4.2 or later
     ffmpeg -version
     
     # Check cmake version. make sure it is verion 3.8 or later
     cmake -version
     
     # Create a new folder in the home directory
     mkdir /app/.packages
     
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
     
   * [geos](https://libgeos.org/)

     ```bash
     # In the Docker container's shell
     # cwd: /
     
     # Install the required packages for cartopy
     apt install -y libgeos-dev
     ```

6. Install ImageBind.

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

7. Install Jupyter Lab or Jupyter Notebook.

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

   1. Get the ID of the running container:

      ```bash
      # In Raspberry Pi's terminal window
      docker ps -a
      CONTAINER ID   IMAGE          COMMAND   CREATED          STATUS          PORTS   NAMES
      3480abe0e7d3   ubuntu:focal   "bash"    20 minutes ago   Up 20 minutes           flamboyant_tu
      ```
      
   2. Create a new image:

      ```bash
      # In Raspberry Pi's terminal window
      docker commit 3480abe0e7d3 imagebind
      ```
      
   3. Verify:

      ```bash
      # In Raspberry Pi's terminal window
      $ docker image ls
      REPOSITORY    TAG       IMAGE ID       CREATED          SIZE
      imagebind     latest    cce74715fcb6   10 seconds ago   3.02GB
      ubuntu        focal     3048ba078595   5 weeks ago      65.7MB
      hello-world   latest    ee301c921b8a   11 months ago    9.14kB
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

3. Jupyter Lab or Jupyter Notebook should now be available at the following URLs and accessible from another computer that is connected to the same network as your Raspberry Pi:

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

2. In Jupyter Notebook running in the Docker container, navigate to the directory `/app/scripts` and add the example notebook to the directory using Jupyter Notebook's upload button.

   The `/app` directory in the Docker container/Jupyter Notebook should now look like this: 

   ```bash
   app
   ├── .packages
   ├── ImageBind
   └── scripts
       └── example.ipynb
   ```

   Note: The `.packages` directory will be hidden (not visible) in Jupyter Notebook.

<br />

## Troubleshooting

### Python kernel keeps dying when loading ImageBind model in Jupyter notebook.

This is most likely caused by the allocated compute resources running out. Try one or more of the following:

- [Resize the swap](../README.md#swap) on your Raspberry Pi. The swap size should be minimum 2GB.
- [Turn off GUI](../README.md#graphical-user-interface-gui) on your Raspberry Pi.
- Pre-download the ImageBind model.
- Restart Python kernel.
- Restart the Docker container.
- Reboot your Raspberry Pi.
