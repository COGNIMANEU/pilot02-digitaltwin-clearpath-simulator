FROM osrf/ros:humble-desktop-full

# Set environment variables
ENV DEBIAN_FRONTEND=noninteractive
ENV DISPLAY=:1
ENV CONFIG_DIR=/clearpath_ws/config
ENV LAUNCH_FILE=gazebo_test.launch.py
ENV VNC_PASSWORD=1234
ENV VNC_PORT=6080
ENV RESOLUTION=1280x800
ENV COL_DEPTH=24
ENV ROS_DISTRO=humble

RUN apt-get update && apt-get install -y \
    wget \
    gnupg \
    lsb-release \
    && echo "deb http://packages.osrfoundation.org/gazebo/ubuntu-stable $(lsb_release -cs) main" > /etc/apt/sources.list.d/gazebo-stable.list \
    && wget https://packages.osrfoundation.org/gazebo.key -O /tmp/gazebo.key \
    && apt-key add /tmp/gazebo.key \
    && rm /tmp/gazebo.key \
    && apt-get update

RUN apt-get install -y \
    ros-humble-ros-gz-sim \
    ros-humble-ros-gz-bridge \
    ros-humble-ros-gz-image \
    ros-humble-rviz2 \
    ros-humble-teleop-twist-keyboard \
    ros-humble-moveit-planners-chomp \
    ros-humble-gripper-controllers \
    ros-humble-moveit-configs-utils \
    ros-humble-moveit-kinematics \
    ros-humble-moveit-planners \
    ros-humble-moveit-ros-move-group \
    ros-humble-moveit-ros-warehouse \
    ros-humble-moveit-simple-controller-manager \
    ros-humble-position-controllers \
    ros-humble-xacro \
    ros-humble-clearpath-viz \
    ros-humble-ign-ros2-control \
    ros-humble-ros-gz \
    ros-humble-franka-description \
    ros-humble-kortex-description \
    ros-humble-robotiq-description \
    ros-humble-ur-description \
    ros-humble-microstrain-inertial-description \
    ros-humble-realsense2-description \
    ros-humble-velodyne-description \
    ros-humble-clearpath-mecanum-drive-controller \
    ros-humble-controller-manager \
    ros-humble-diff-drive-controller \
    ros-humble-imu-filter-madgwick \
    ros-humble-interactive-marker-twist-server \
    ros-humble-joint-state-broadcaster \
    ros-humble-joint-trajectory-controller \
    ros-humble-joy-linux \
    ros-humble-robot-localization \
    ros-humble-teleop-twist-joy \
    ros-humble-twist-mux \
    ros-humble-moveit-setup-srdf-plugins 

RUN apt-get install -y \
    ignition-fortress \
    libignition-gazebo6-dev \
    libignition-gazebo6-plugins \
    xfce4 xfce4-goodies \
    x11vnc xvfb \
    novnc websockify \
    net-tools curl \
    libqt5x11extras5 libxcb-xinerama0 libxcb-util1 python3-vcstool \
    git \
    dbus dbus-x11 pm-utils\
    && rm -rf /var/lib/apt/lists/*

RUN apt-get purge -y xfce4-power-manager upower gvfs gvfs-daemons

# Source ROS 2 in default shell
RUN echo "source /opt/ros/humble/setup.bash" >> /root/.bashrc

RUN mkdir -p /clearpath_ws/src
RUN mkdir -p /clearpath_ws/config
WORKDIR /clearpath_ws/src

# Clone simulator source
RUN git clone -b humble https://github.com/clearpathrobotics/clearpath_simulator.git
RUN vcs import < clearpath_simulator/dependencies.repos

# Install ROS dependencies
WORKDIR /clearpath_ws
RUN rosdep update && rosdep install -r --from-paths src -i -y

# Build the workspace
RUN /bin/bash -c "source /opt/ros/humble/setup.bash && colcon build --symlink-install"

# Source Clearpath setup in future shells
RUN echo "source /clearpath_ws/install/setup.bash" >> /root/.bashrc

# Default config
COPY config/*.yaml /clearpath_ws/install
COPY config/*.py /clearpath_ws/install/clearpath_gz/share/clearpath_gz/launch

RUN echo '#!/bin/bash\n\
if [ -z "$DBUS_SESSION_BUS_ADDRESS" ]; then\n\
    eval $(dbus-launch --sh-syntax)\n\
fi\n\
cp $CONFIG_DIR/*.py /clearpath_ws/install/clearpath_gz/share/clearpath_gz/launch\n\
source /opt/ros/humble/setup.bash\n\
source /clearpath_ws/install/setup.bash\n\
ros2 run clearpath_generator_common generate_bash -s /clearpath_ws/install\n\
export GAZEBO_MODEL_PATH=/config/models:$GAZEBO_MODEL_PATH\n\
export GZ_SIM_SYSTEM_PLUGIN_PATH=/usr/lib/x86_64-linux-gnu/ign-gazebo-6/plugins:$GZ_SIM_SYSTEM_PLUGIN_PATH\n\
rm -f /tmp/.X1-lock\n\
Xvfb :1 -screen 0 ${RESOLUTION}x${COL_DEPTH} &\n\
sleep 2\n\
startxfce4 --replace --no-daemon &\n\
x11vnc -display :1 -passwd $VNC_PASSWORD -forever -shared -rfbport 5900 &\n\
/usr/share/novnc/utils/launch.sh --vnc localhost:5900 --listen $VNC_PORT &\n\
ros2 launch clearpath_gz $LAUNCH_FILE' > /startup.sh

# Make startup script executable
RUN chmod +x /startup.sh

# Default container command
CMD ["/bin/bash", "/startup.sh"]