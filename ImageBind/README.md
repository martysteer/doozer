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
│   └── Playpen
│       └── imagebind            <-- mapped to Docker image/containers's /app/scripts directory
│           ├── example.ipynb
│           ├── processing.ipynb
│           └── search.py
└── ... 
```

### Docker Image/Container

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

## Build Docker image

1. Pull Ubuntu 20.04 (Focal Fossa).

   ```bash
   # In Raspberry Pi's terminal window
   docker pull ubuntu:focal
   ```

   > **Note**: ImageBind requires Python 3.8.x, hence Ubuntu 20.04 being used here, but any other OS running Python 3.8.x should work (though some of the steps below may need adjusting).

2. Start a container and interactive Bash shell.

   ```bash
   # In Raspberry Pi's terminal window
   docker run --rm -it --entrypoint bash ubuntu:focal
   ```

3. Create `/app` directory.

   ```bash
   # In the Docker container's shell
   # Path: /
   mkdir /app
   ```
   
4. Install pre-requisite packages for installation of ImageBind.

   ```bash
   # In the Docker container's shell
   # Path: /
   apt update
   apt install -y nano git wget
   apt install -y build-essential make cmake
   apt install -y python3 python3-pip
   ```
   
5. Install ImageBind's problematic dependencies manually.

   * [mayavi](https://github.com/enthought/mayavi)

     ```bash
     # In the Docker container's shell
     # Path: /
     
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
     # Path: /
     
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

6. Install ImageBind.

   ```bash
   # In the Docker container's shell
   # Path: /
   
   # Install the required packages for cartopy
   apt install -y libgeos-dev
   
   # Clone the ImageBind repository
   git -C /app clone https://github.com/facebookresearch/ImageBind
   
   # Edit the requirements file
   # Change the line `eva-decord==0.6.1` to `decord==0.6.0`
   nano /app/ImageBind/requirements.txt
   
   # Install dependencies
   python3 -m pip install /app/ImageBind/.
   
   # Pre-download the model
   mkdir /app/ImageBind/.checkpoints
   wget -P /app/ImageBind/.checkpoints https://dl.fbaipublicfiles.com/imagebind/imagebind_huge.pth
   ```

7. Install Jupyter Notebook.

   ```bash
   # In the Docker container's shell
   # Path: /
   python3 -m pip install notebook
   python3 -m jupyter notebook password # e.g. raspberry
   ```
   
   > **Note**: Jupyter Notebook is installed here, instead of Jupyter Lab, to minimize software footprint and potential dependency conflicts (no issues yet for ImageBind but known issues for LLaVA).
   
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

   Press the **Ctrl**+**Alt**+**F1** keys to enter console mode and run the command:

   ```bash
   # In Raspberry Pi's terminal window
   sudo systemctl isolate multi-user.target
   ```
   
2. Start the Docker container.

   ```bash
   # In Raspberry Pi's terminal window
   docker run --rm -p 8888:8888 -v ~/Projects/Playpen/imagebind:/app/scripts -it --entrypoint bash imagebind:latest
   ```
   
3. Once inside the container, start Jupyter notebook (optional).

   ```bash
   # In the Docker container's shell
   # Path: /
   python3 -m jupyter notebook --ip 0.0.0.0 --port 8888 --allow-root --no-browser
   ```
   
   Open Jupyter Notebook from another computer, and run/edit.

### Stop the Docker container

1. If you are inside the container, exit the container.

   ```bash
   # In the Docker container's shell
   exit
   ```
   
2. Enable GUI, if not disabled by default.

   Run the command, and press the **Ctrl**+**Alt**+**F1** keys to switch to Desktop:

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
