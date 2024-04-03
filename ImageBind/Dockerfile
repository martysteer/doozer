FROM ubuntu:20.04

ARG DEBIAN_FRONTEND=noninteractive
ENV TZ="Europe/London"

# Install pre-requisites
RUN apt update && apt install -y \
nano git wget \
build-essential make cmake \
python3 python3-pip

RUN mkdir -p /app

# Pre-install pesky ImageBind dependencies
# mayavi
RUN apt install -y mayavi2
# decord (instead of eva-decord)
RUN apt install -y ffmpeg libavcodec-dev libavfilter-dev libavformat-dev libavutil-dev
RUN mkdir -p /app/.packages && git -C /app/.packages clone --recursive https://github.com/dmlc/decord
RUN mkdir -p /app/.packages/decord/build
WORKDIR /app/.packages/decord/build
RUN cmake .. -DUSE_CUDA=0 -DCMAKE_BUILD_TYPE=Release && make
WORKDIR ../python
RUN python3 setup.py install
# geos
RUN apt install -y libgeos-dev

# Install ImageBind
RUN git -C /app clone https://github.com/facebookresearch/ImageBind
WORKDIR /app/ImageBind
RUN sed -i "/^eva-decord/s/^.*$/decord==0.6.0/" requirements.txt
RUN python3 -m pip install .
# pre-download ImageBind model (this takes )
RUN mkdir -p .checkpoints && wget -P .checkpoints https://dl.fbaipublicfiles.com/imagebind/imagebind_huge.pth

# Install Jupyter Lab
RUN python3 -m pip install jupyterlab ipywidgets
# set Jupyter Lab password
RUN mkdir -p /root/.jupyter && printf '{\n  "IdentityProvider": {\n    "hashed_password": "sha256:22c2e4faf76e:5774f4e58266260008480196ddf9c4de39267f6d2df7566e5dde17ba1210d7d6"\n  }\n}' >> /root/.jupyter/jupyter_server_config.json

WORKDIR /app
EXPOSE 8888
ENTRYPOINT ["jupyter", "lab", "--ip=0.0.0.0", "--port=8888", "--allow-root", "--no-browser"]