# RoboScan, fork sygolene

! This is a fork of Roboscan. The scope of edits are customizing & tailoring to my own hardware (no led control, different motor driver,...). No changes on features nor algorithms.
Original instruction follows. Local edits will be marked with a #sygolene edit comment tag.

This is the source code for a Lego+Raspberry Pi-powered analog film roll scanner. Watch it in action:
[![RoboScan](https://yt-embed.herokuapp.com/embed?v=yRDomN48SOs)](https://youtu.be/yRDomN48SOs)

## Parts
You'll need these items to build RoboScan:
* A digital camera with a macro lens: must be compatible with [libgphoto2 with image capture and preview support](http://gphoto.org/proj/libgphoto2/support.php).
* A Raspberry Pi: you may choose a Pi 4 if your camera supports USB 3, otherwise a Pi 2 or 3 is fine.
* A Nema17 Stepper Motor with A4988 driver
* The A4988 circuit includes a power inlet & a capacitor.
* An external backlight, indepently operated from the circuit. This implies the original RoboScan code is modified to remove this part.
* A film advance mechanism mechanically linked to the motor.

## Part 1: Wiring

### Diagram
![Roboscan wiring diagram](./documents/roboscan-wiring.svg)

(made using Fritzing with the help of parts from [e-radionica.com](https://github.com/e-radionicacom/e-radionica.com-Fritzing-Library-parts-) and [Blomquist](https://forum.fritzing.org/t/led-driver-board-350ma-pwm-updated-with-test-part-v3/3322/19))

### ULN2003A wiring
Put a 50V, 47 μF capacitor between the LED+ and LED - pins of the driver.

| ULN2003A Stepper Motor driver | Raspberry Pi |
| ----------- | ----------- |
| IN1 | GPIO 5 |
| IN2 | GPIO 6 |
| IN3 | GPIO 13 |
| IN4 | GPIO 19 |
| POWER+ | 5V power (such as the one next to the Ground) |
| POWER - | Ground (such as the one next to the 5V power) |

## Part 2: Software installation

### Prepare the Raspberry Pi
Follow Raspberry foundation documentation, such as: [https://projects.raspberrypi.org/en/projects/raspberry-pi-setting-up](https://projects.raspberrypi.org/en/projects/raspberry-pi-setting-up).
Tip: you can set the hostname of your Raspberry Pi to "piscanner" as it's what's used in this tutorial. You can use [raspi-config](https://www.raspberrypi.org/documentation/configuration/raspi-config.md) for this.

### Install Docker
The easiest is to use the "convenience script" as described here: [https://docs.docker.com/engine/install/debian/#install-using-the-convenience-script](https://docs.docker.com/engine/install/debian/#install-using-the-convenience-script).

In a nutshell:
```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
# Add your user to the docker group
sudo groupadd docker
sudo usermod -aG docker $USER
```

### Install docker-compose
Full docker-compose installation instructions are available here: [https://docs.docker.com/compose/install/#install-compose-on-linux-systems](https://docs.docker.com/compose/install/#install-compose-on-linux-systems).

On Raspbian, the easiest is to install it through Python's pip:
```bash
# Install Python3 and pip
sudo apt install python3 python3-pip
# Install docker-compose
sudo pip install docker-compose
```

### Clone this repository on your Raspberry Pi
```bash
git clone https://github.com/bezineb5/RoboScan.git
cd RoboScan
```

### Start the application
Now, you will ask docker to build and start the application. This might take a while (30-120 minutes).
```bash
# sygolene edit : add a sudo to the following command
cd docker-compose up -d --build
```

It will start all components and restart them at reboot.

## Part 3: Using RoboScan
The camera must be connected to the Raspberry Pi via USB. It must be compatible with [libgphoto2](www.gphoto.org/proj/libgphoto2/support.php).

### Connect to the web interface
Simply navigate to [http://piscanner/](http://piscanner/) (adjust the hostname to your Raspberry Pi)

### Optional: Google Coral TPU
You can improve the machine learning inference performance by using a [Google Coral Edge TPU USB Accelerator](https://coral.ai/products/accelerator) plugged on a USB port of the Raspberry Pi.
To do so, you have to change the file src/Dockerfile. Replace:
```Dockerfile
CMD ["python", "webapp.py", "--destination", "/storage/share", "--archive", "/storage/archive", "--temp", "/storage/tmp"]
```
by:
```Dockerfile
CMD ["python", "webapp.py", "-tpu", "--destination", "/storage/share", "--archive", "/storage/archive", "--temp", "/storage/tmp"]
```

### Optional: for developers
The easiest is to code on you PC and deploy docker containers remotely. To do so, [enable remote access to the docker daemon](https://docs.docker.com/engine/install/linux-postinstall/#configure-where-the-docker-daemon-listens-for-connections).

```bash
# Set the DOCKERHOST variable (only once)
# Adjust the hostname to your Raspberry Pi
export DOCKER_HOST=tcp://piscanner.local:2376 DOCKER_TLS_VERIFY=

# Then deploy as usual
cd docker
docker-compose up -d
cd ..
```
