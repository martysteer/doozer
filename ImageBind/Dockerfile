FROM ubuntu:20.04

ARG DEBIAN_FRONTEND=noninteractive
ENV TZ="Europe/London"

# Set up directory structure
RUN mkdir -p /app
RUN mkdir -p /app/.packages

# Install pre-requisites
RUN apt update && apt install -y \
nano git wget \
build-essential make cmake \
python3 python3-pip

# Update pip 
RUN python3 -m pip install --upgrade pip

# Pre-install pesky ImageBind dependencies
# Mayavi
RUN apt install -y mayavi2
# GEOS
RUN apt install -y libgeos-dev
# Decord (instead of EVA Decord)
RUN apt install -y ffmpeg libavcodec-dev libavfilter-dev libavformat-dev libavutil-dev
RUN git -C /app/.packages clone --recursive https://github.com/dmlc/decord
WORKDIR /app/.packages/decord
RUN mkdir -p build
RUN cmake . -Bbuild -DUSE_CUDA=0 -DCMAKE_BUILD_TYPE=Release && make -C build
WORKDIR python
RUN python3 setup.py install

WORKDIR /

# Install ImageBind
RUN git -C /app clone https://github.com/facebookresearch/ImageBind
WORKDIR /app/ImageBind
RUN sed -i "/^eva-decord/s/^.*$/decord==0.6.0/" requirements.txt
RUN python3 -m pip install .
# Pre-download ImageBind model (this takes )
RUN mkdir -p .checkpoints && wget -P .checkpoints https://dl.fbaipublicfiles.com/imagebind/imagebind_huge.pth

WORKDIR /

# Install Jupyter Lab
RUN python3 -m pip install jupyterlab ipywidgets
# Set Jupyter Lab password
RUN mkdir -p /root/.jupyter && printf '{\n  "IdentityProvider": {\n    "hashed_password": "sha256:22c2e4faf76e:5774f4e58266260008480196ddf9c4de39267f6d2df7566e5dde17ba1210d7d6"\n  }\n}' >> /root/.jupyter/jupyter_server_config.json

## Pre-install pesky ChromaDB dependencies
## SQLite
## ChromaDB requires SQLite > 3.35 and the version available to Ubuntu 20.04 is 3.31.x,
## so building a newer version from source instead.
#RUN apt install -y libreadline-dev
#RUN wget -P /app/.packages https://www.sqlite.org/2024/sqlite-autoconf-3450200.tar.gz
#RUN tar xzf /app/.packages/sqlite-autoconf-3450200.tar.gz -C /app/.packages/
#WORKDIR /app/.packages/sqlite-autoconf-3450200
#RUN ./configure && make
#RUN apt purge sqlite3
#RUN make install
#RUN echo 'export LD_LIBRARY_PATH=/usr/local/lib' >> ~/.bashrc
#ENV LD_LIBRARY_PATH="/usr/local/lib"
## gRPC
#WORKDIR /
#RUN python3 -m pip install grpcio
#
## Install ChromaDB
#RUN python3 -m pip install chromadb

# Finish up
WORKDIR /app
EXPOSE 8888
ENTRYPOINT ["jupyter", "lab", "--ip=0.0.0.0", "--port=8888", "--allow-root", "--no-browser"]