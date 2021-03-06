Dockerfile for ROS melodic `desktop-full`.

Features:

* X11 authentication for GUIs
* Image size: 2.9 GB
* User created during runtime
* Nvidia GPU and OpenGL support

## Build the image

### Intel GPU

```
docker build --rm --pull -t diegoferigo/ros .
```

### Nvidia GPU

```
docker build --rm --pull --build-arg from=nvidia/opengl:1.0-glvnd-runtime-ubuntu18.04 -t diegoferigo/ros:nvidia .
```

## User configuration

Several reasons could be found for having a non-root user within the container
that matches the GID and UID of your host user.

This docker image allows the creation of a runtime user, whose default UID and
GID is 1000. To override these values and to start the container, execute:

```
docker run -i -t --rm \
	-e USER_UID=$(id -u) \
	-e USER_GID=$(id -g) \
	-e USERNAME=$(whoami) \
	--name ros \
	diegoferigo/ros \
	bash
```

Then, create as many ttys as needed with

```
docker exec -it ros bash
```

The image does not contain `sudo` and the root password is not set. When the active
user is not root, to execute commands that require admin privileges you should
open a new tty or use `exit` on the current shell getting back to the root user.

## X11 authentication

As an example, `rqt` is forwarded from within the container to the host's Xserver:

```
XSOCK=/tmp/.X11-unix
XAUTH=/tmp/.docker.xauth

touch $XAUTH
xauth nlist $DISPLAY | sed -e 's/^..../ffff/' | xauth -f $XAUTH nmerge -

docker run -i -t --rm \
	-v $XSOCK:$XSOCK:rw \
	-v $XAUTH:$XAUTH:rw \
	-e XAUTHORITY=${XAUTH} \
	-e DISPLAY \
	-e USER_UID=$(id -u) \
	-e USER_GID=$(id -g) \
	-e USERNAME=$(whoami) \
	--name ros \
	diegoferigo/ros \
	rqt
```

If you need HW acceleration (only for Intel graphic cards), add also this device
flag `--device=/dev/dri`.

For what concerns Nvidia GPUs, you need to configure and install [`nvidia-docker`](https://github.com/nvidia/nvidia-docker/wiki/Installation-(version-2.0)). With such setup, you only have to:

1. Add the additional option `--runtime nvidia` 
2. Use the `diegoferigo/ros:nvidia` image

## Resources

* [Docker and ROS][0]
* X11 Authentication: [stackoverflow][1], [ROS Wiki][2]
* Runtime user: [docker-browser-box][6]
* [Hardware Acceleration][3]

## TODO

* Setup a ROS [docker-compose][4] system (see also [docker hub][5] - Compose)
* Setup sudo w/o password

[0]: http://wiki.ros.org/docker/Tutorials
[1]: https://stackoverflow.com/questions/16296753/can-you-run-gui-apps-in-a-docker-container
[2]: http://wiki.ros.org/docker/Tutorials/GUI
[3]: http://wiki.ros.org/docker/Tutorials/Hardware%20Acceleration
[4]: http://toddsampson.com/post/131227320927/docker-experimental-networking-and-ros
[5]: https://hub.docker.com/_/ros/
[6]: https://github.com/sameersbn/docker-browser-box
