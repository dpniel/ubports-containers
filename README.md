# Ubuntu Touch Cross Containers

This script will create a configure cross build containers for ubuntu touch development.

It currently supports creating containers for

* xenial (armhf)
* vivid (armhf)

## Usage

LXD is used to create the containers so that needs installing and configuring first.

```bash
sudo apt install lxd qemu-user-static
sudo lxd init # and accept all defaults
sudo reboot
```

Now we can create our first container.

> At the time of writing the xenial container is broken due to some dependency issue. So only vivid will work for now

The `create-cross-container` script has the following options

```bash
➜  ubports-containers ./create-cross-container -h

    create-cross-container [OPTIONS...]
    -t, --target         What platform to target. xenial or vivid (default: vivid)
    -n, --name           Name of the container. If no name is given then one of TARGET-cross will be used
    -m, --mount          Directory to mount as containers home directory (default: /home/dan)
    -o, --use-overlay    use the stable overlay ppa
    -u, --use-ubports    use the ubports repo
```

### Minimal setup

This creates a raw container with no additional repositories or packages installed

```bash
cd ubports-containers
./create-cross-container --target vivid --no-libs
```

### Vivid container with overlay ppa

This creates a container with the sdk libs installed from the overlay ppa

```bash
cd ubports-containers
./create-cross-container --target vivid --use-overlay
```

### Mount a specific directory as the container home

This is good for if you would like to only mount your project in the container


```bash
cd ubports-containers
./create-cross-container -t vivid -o --mount ${HOME}/path/to/directory
```

## Using the containers

Run `lxc list` to see the status of your containers. If you find one isn't running then
you can `lxc start $CONTAINER` you can also stop with `lxc stop $CONTAINER`

To get a bash prompt in the container as either root or a user.

```bash
# for root
lxc exec $CONTAINER_NAME bash
# for user
lxc exec $CONTAINER_NAME -- sudo --login --user ubuntu bash
```

Or you can pass command directly into the container like so

```bash
lxc exec $CONTAINER -- sudo --login --user ubuntu sh -c "echo \"Hello World\""
```

### Building a click

This works very much the same way as anywhere else. Let's use a qmake based application as an example

```bash
➜  ubports-containers lxc exec vivid-cross -- sudo --login --user ubuntu
ubuntu@vivid-cross:~$ export QT_SELECT=qt5
ubuntu@vivid-cross:~$ cd myproject && mkdir build && cd build
ubuntu@vivid-cross:~$ qt5-qmake-arm-linux-gnueabihf ..
ubuntu@vivid-cross:~$ make -j4
ubuntu@vivid-cross:~$ make INSTALL_ROOT=./click-pkg install
ubuntu@vivid-cross:~$ click build ./click-pkg
```

### Delete container

```bash
lxc stop $CONTAINER
lxc delete $CONTAINER
```