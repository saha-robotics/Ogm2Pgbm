# This is an auto generated Dockerfile for ros:ros-core
# generated from docker_images/create_ros_core_image.Dockerfile.em
FROM ubuntu:focal

# setup timezone
RUN echo 'Etc/UTC' > /etc/timezone && \
    ln -s /usr/share/zoneinfo/Etc/UTC /etc/localtime && \
    apt-get update && \
    apt-get install -q -y --no-install-recommends tzdata && \
    rm -rf /var/lib/apt/lists/*

# install packages
RUN apt-get update && apt-get install -q -y --no-install-recommends \
    dirmngr \
    gnupg2 \
    git \
    && rm -rf /var/lib/apt/lists/*

# setup keys
RUN set -eux; \
       key='C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654'; \
       export GNUPGHOME="$(mktemp -d)"; \
       gpg --batch --keyserver keyserver.ubuntu.com --recv-keys "$key"; \
       mkdir -p /usr/share/keyrings; \
       gpg --batch --export "$key" > /usr/share/keyrings/ros1-latest-archive-keyring.gpg; \
       gpgconf --kill all; \
       rm -rf "$GNUPGHOME"

# setup sources.list
RUN echo "deb [ signed-by=/usr/share/keyrings/ros1-latest-archive-keyring.gpg ] http://packages.ros.org/ros/ubuntu focal main" > /etc/apt/sources.list.d/ros1-latest.list

# setup environment
ENV LANG C.UTF-8
ENV LC_ALL C.UTF-8

ENV ROS_DISTRO noetic

RUN apt-get update && apt-get install -y --no-install-recommends \
    ros-noetic-ros-core=1.5.0-1* \
    python3-catkin-tools \
    python3-rosdep \
    && rm -rf /var/lib/apt/lists/*

COPY ./docker/ros_entrypoint.sh /


SHELL ["bash", "-c"]
ENV USER=sr
ENV DEBIAN_FRONTEND=noninteractive
RUN  echo 'debconf debconf/frontend select Noninteractive' | sudo debconf-set-selections
RUN sudo apt update && sudo apt install -y -q keyboard-configuration python3-wstool ninja-build stow
RUN useradd -ms /bin/bash $USER
RUN adduser $USER sudo
RUN echo '%sudo ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers

# Set the user to $USER
USER $USER
#
# setup entrypoint
#

WORKDIR /home/$USER

RUN mkdir -p ~/catkin_ws/src \
&& mkdir -p ~/cartographer_ws/src \
&& cd ~/catkin_ws/src \
&& git clone https://github.com/saha-robotics/Ogm2Pgbm.git \
&& cd ~/cartographer_ws \
&& wstool init src \
&& wstool merge -t src https://raw.githubusercontent.com/cartographer-project/cartographer_ros/master/cartographer_ros.rosinstall \
&& wstool update -t src

RUN sudo chown -R $USER:$USER ~/catkin_ws/src \
&&  sudo chown -R $USER:$USER ~/cartographer_ws/src 

WORKDIR /home/$USER/cartographer_ws
RUN . /opt/ros/$ROS_DISTRO/setup.bash \ 
  && sudo apt update \
  && sudo rosdep init \
  && rosdep update --rosdistro $ROS_DISTRO \
  && rosdep install --from-paths src --ignore-src --rosdistro $ROS_DISTRO \
  --skip-keys libabsl-dev \
  -y \
  && src/cartographer/scripts/install_abseil.sh \
  && catkin_make_isolated --install --use-ninja

WORKDIR /home/$USER/catkin_ws

RUN . /opt/ros/$ROS_DISTRO/setup.bash \ 
    . /home/$USER/cartographer_ws/install_isolated/setup.bash \ 
    && sudo apt update \
#   && sudo rosdep init \
#   && rosdep update --rosdistro $ROS_DISTRO \
  && rosdep install --from-paths src --ignore-src --rosdistro $ROS_DISTRO \
  -y \
  && catkin config --install \
  && catkin build --cmake-args -DCMAKE_BUILD_TYPE=Release

RUN sudo chown -R $USER:$USER /ros_entrypoint.sh 
RUN echo 'source /opt/ros/${ROS_DISTRO}/setup.bash' >> /home/$USER/.bashrc
RUN echo 'source /home/$USER/cartographer_ws/install_isolated/setup.bash' >> /home/$USER/.bashrc
RUN echo 'source /home/$USER/catkin_ws/install/setup.bash' >> /home/$USER/.bashrc
ENTRYPOINT ["/ros_entrypoint.sh"]
CMD ["bash"]
