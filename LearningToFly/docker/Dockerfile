FROM nvidia/cuda:11.3.1-cudnn8-devel-ubuntu18.04
ENV DEBIAN_FRONTEND "noninteractive"
ENV TZ "America/New_York"

# Python package management and basic dependencies
RUN apt-get -y update && apt-get install -y curl python3.7 python3.7-dev python3.7-distutils

# Register the version in alternatives
RUN update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.7 1

# Set python 3 as the default python
RUN update-alternatives --set python3 /usr/bin/python3.7

# Upgrade pip to latest version
RUN curl -s https://bootstrap.pypa.io/get-pip.py -o get-pip.py && \
    python3 get-pip.py --force-reinstall && \
    rm get-pip.py

# Fix NO_PUBKEY A4B469963BF863CC error https://github.com/NVIDIA/nvidia-docker/issues/1632
RUN rm -rf /etc/apt/sources.list.d/cuda.list
RUN rm -rf /etc/apt/sources.list.d/nvidia-ml.list

RUN set -x \
  && echo "Preparing system..." \
  && apt-get -y update \
  && apt-get -y install \
    wget \
    curl \
    unzip \
    git \
    mpich \
    libosmesa6-dev libgl1-mesa-glx libglfw3 \
    x11-apps \
    vim \
    cmake \
    libgl1-mesa-dev x11-xserver-utils libglu1-mesa-dev xorg-dev \
  && rm -rf /var/lib/apt/lists/* 

RUN set -x \
  && echo "Installing jupyter kernel..." \
  && pip3 install --no-cache-dir ipykernel \
  && python3 -m ipykernel install


# Install Mujoco v150
# RUN mkdir -p /root/.mujoco \
#     && wget https://www.roboti.us/download/mjpro150_linux.zip \
#     && unzip mjpro150_linux.zip -d /root/.mujoco/ \
#     && wget https://www.roboti.us/file/mjkey.txt -O /root/.mujoco/mjkey.txt \
#     && rm mjpro150_linux.zip
# ENV LD_LIBRARY_PATH /root/.mujoco/mjpro150/bin:${LD_LIBRARY_PATH}
# ENV MUJOCO_PY_MUJOCO_PATH /root/.mujoco/mjpro150

# Install Mujoco v210 which is compatible with mujoco-py (https://github.com/openai/mujoco-py)
RUN wget https://github.com/deepmind/mujoco/releases/download/2.1.0/mujoco210-linux-x86_64.tar.gz \
    && mkdir -p /root/.mujoco/ \
    && tar -zxf mujoco210-linux-x86_64.tar.gz -C /root/.mujoco/ \
    && wget https://www.roboti.us/file/mjkey.txt -O /root/.mujoco/mjkey.txt \
    && rm mujoco210-linux-x86_64.tar.gz
ENV LD_LIBRARY_PATH /root/.mujoco/mujoco210/bin:${LD_LIBRARY_PATH}
ENV MUJOCO_PY_MUJOCO_PATH /root/.mujoco/mujoco210



# RUN ln -s /usr/lib/x86_64-linux-gnu/libGL.so.1 /usr/lib/x86_64-linux-gnu/libGL.so
RUN pip3 install tensorflow==1.15.5 protobuf==3.20.3

ADD requirements.txt /app/requirements.txt
RUN set -x \
 && echo "Installing python dependencies from requirements.txt..." \
 && pip3 install --no-cache-dir -r /app/requirements.txt \
 && rm /app/requirements.txt

# L2F package: 
WORKDIR /root
RUN git clone https://github.com/eanswer/LearningToFly.git
RUN cd LearningToFly/SimulationUI && mkdir build && cd build \
    && cmake -DCMAKE_BUILD_TYPE=Release ../ \
    && make -j100

RUN chmod 777 -R /root/.mujoco
RUN chmod 755 /root
#RUN rm -rf /root/LearningToFly/.git && chmod 777 -Rv /root
RUN chmod 777 -R /usr/local/lib/python3.7/dist-packages/mujoco_py/generated

# RUN cd LearningToFly/Training/code && python3 run.py --model QuadPlane --visualize
# RUN cd LearningToFly/Training/code && python3 run.py --model QuadPlane 

RUN apt-get update
RUN apt-get install -y dbus

ENV NVIDIA_DRIVER_CAPABILITIES compute,utility,display

RUN pip3 install ipdb 
RUN apt-get install -y libglew-dev libglfw3-dev
ENV LD_PRELOAD=$LD_PRELOAD:/usr/lib/x86_64-linux-gnu/libGLEW.so
#RUN ldconfig

EXPOSE 8888

CMD [ "bash" ]


