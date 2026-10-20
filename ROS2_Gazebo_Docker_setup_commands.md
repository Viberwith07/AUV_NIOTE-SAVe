# Setup commands, Section 2 (and the first build in Section 3)

Every command from the setup lectures, in order, ready to copy. Run them in a
terminal on Ubuntu 24.04. Lines starting with `#` are comments.

These were re-tested on 2026-10-19 against a clean Ubuntu 24.04 image. One
change from the video: ROS 2 now installs its apt source and key from a single
package (`ros2-apt-source`) instead of downloading the key by hand. The older
method shown on screen can fail because that key was rotated in 2025. Use the
commands below.

## Install ROS 2 (Jazzy)

```bash
# 1. software sources: enable "universe" and get curl
sudo apt update && sudo apt install -y software-properties-common curl
sudo add-apt-repository -y universe

# 2. the official ROS 2 apt source and signing key, as one package
export ROS_APT_SOURCE_VERSION=$(curl -s https://api.github.com/repos/ros-infrastructure/ros-apt-source/releases/latest | grep -F "tag_name" | awk -F\" '{print $4}')
curl -L -o /tmp/ros2-apt-source.deb "https://github.com/ros-infrastructure/ros-apt-source/releases/download/${ROS_APT_SOURCE_VERSION}/ros2-apt-source_${ROS_APT_SOURCE_VERSION}.$(. /etc/os-release && echo $VERSION_CODENAME)_all.deb"
sudo dpkg -i /tmp/ros2-apt-source.deb

# 3. the full desktop install (tools + RViz)
sudo apt update && sudo apt upgrade -y
sudo apt install -y ros-jazzy-desktop ros-dev-tools

# 4. sourcing: the step people forget. Add it to every new terminal:
echo "source /opt/ros/jazzy/setup.bash" >> ~/.bashrc
source /opt/ros/jazzy/setup.bash

# 5. check the install
ros2 doctor
```

`ros2 doctor` should end with `All <n> checks passed`. A warning about
network interfaces on a laptop is normal.

## Install Gazebo (Harmonic) and the ROS-Gazebo bridge

```bash
# 1. Gazebo's software source and key
sudo apt install -y curl lsb-release gnupg
sudo curl https://packages.osrfoundation.org/gazebo.gpg --output /usr/share/keyrings/pkgs-osrf-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/pkgs-osrf-archive-keyring.gpg] https://packages.osrfoundation.org/gazebo/ubuntu-stable $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/gazebo-stable.list > /dev/null

# 2. Gazebo Harmonic
sudo apt update
sudo apt install -y gz-harmonic

# 3. the bridge between ROS 2 and Gazebo
sudo apt install -y ros-jazzy-ros-gz

# 4. check: this opens an empty Gazebo world
gz sim -v 4 empty.sdf
```

## The colcon workspace

This uses the course assets zip from the "Course Assets" lecture. Unzip it
into your home folder first, so you have `~/rov_course`.

```bash
cd ~/rov_course/ros2_ws
colcon build --symlink-install
source install/setup.bash
# so every new terminal also knows your workspace:
echo "source ~/rov_course/ros2_ws/install/setup.bash" >> ~/.bashrc
```

After the build you will see three new folders next to `src`: `build`,
`install` and `log`.

## First publisher (Section 3), the build and run commands

The code is `ros2_ws/src/diver_examples/diver_examples/depth_publisher.py`.

```bash
cd ~/rov_course/ros2_ws
colcon build --symlink-install --packages-select diver_examples
source install/setup.bash
ros2 run diver_examples depth_publisher
```

In a second terminal:

```bash
ros2 topic echo /depth
```

## Install Docker (Section 4)

The four steps from the lecture: download the script as a file, read it, run
it, then let your user run Docker without sudo.

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
less get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER
# log out and back in (or reboot), then:
docker run --rm hello-world
```

## If a command fails

Post in the course Q&A with the command you ran and the last lines it
printed. I read every question.
