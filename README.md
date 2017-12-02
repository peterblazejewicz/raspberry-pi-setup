# Raspberry PI setup

Setting up a Raspberry PI (macOS)

## Topics

Flashing OS, Boot Configuration, WIFI, SSH, Updates, Upgrades, Space Cleanup, VNC Server (RealVNC), Vim, USB Access, Docker, NodeJS, Hostname, Password, Network Connection over USB, Audio

## Flashing OS

[https://etcher.io/](https://etcher.io/)

## Modify boot configuration

```bash
 cd /Volumes/boot
➜  boot
touch wpa_supplicant.conf
vim wpa_supplicant.conf
```

### WiFi

#### Content of `wpa_supplicant.conf`

```text
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1

network={
    ssid="YOURSSID"
    psk="YOURPASSWORD"
    scan_ssid=1
}
```

#### Disable WiFi on boot

On device with built-in WiFi add a line `dtoverlay=pi3-disable-wifi` to boot configuration:

```bash
cd /Volumes/boot
sudo vim config.txt
```

Ref.: [https://git.io/vFje3](https://git.io/vFje3)

### Bluetooth

#### Disable Bluetooth on boot

On device with built-in Bluetooth add a line `dtoverlay=pi3-disable-bt` to boot configuration:

```bash
cd /Volumes/boot
sudo vim config.txt
```

Ref.: [dtoverlay=pi3-disable-bt](dtoverlay=pi3-disable-bt)

### Enable SSH

```bash
touch ssh
```

## Connecting

### Verify WIFI

```bash
ping -c 3 raspberrypi.local
PING raspberrypi.local (192.168.1.20): 56 data bytes
64 bytes from 192.168.1.20: icmp_seq=0 ttl=64 time=12.041 ms
64 bytes from 192.168.1.20: icmp_seq=1 ttl=64 time=19.085 ms
64 bytes from 192.168.1.20: icmp_seq=2 ttl=64 time=11.659 ms

--- raspberrypi.local ping statistics ---
3 packets transmitted, 3 packets received, 0.0% packet loss
round-trip min/avg/max/stddev = 11.659/14.262/19.085/3.414 ms
```

### Connect via SSH

```bash
ssh pi@raspberrypi.local
```

Default password: **raspberry**

You could eventually update your SSH fingerprints:

```bash
ssh-keygen -R raspberrypi.local
ssh pi@raspberrypi.local
```

### Update/Upgrade setup

- Update

```bash
sudo apt-get update
```

- Then Upgrade

```bash
sudo apt-get dist-upgrade
```

- [Optional] Cleanup

```bash
sudo apt-get clean
```

## Enabling VNC Server

- Install/Reinstall VNC server in one go:

```bash
sudo apt-get update
sudo apt-get install realvnc-vnc-server realvnc-vnc-viewer
```

- Enable VNC server in `raspi-config`:

```bash
sudo raspi-config
```

> Interfacing Options > VNC > Would you like the VNC Server to be enabled? > Yes
> The VNC Server is enabled

Let the PI reboot

- Start VNC server manually:

```bash
sudo systemctl start vncserver-x11-serviced.service
```

- Stop VNC manually:

```bash
sudo systemctl stop vncserver-x11-serviced.service
```

- To stop VNC server from running at boot:

```bash
sudo systemctl disable vncserver-x11-serviced.service
```

## Vim

### Install Vim

```bash
sudo apt-get install vim
```

### Syntax Highlighting in Vim

```txt
" Vim5 and later versions support syntax highlighting. Uncommenting the next
" line enables syntax highlighting by default.
"syntax on
```

```bash
sudo vim /etc/vim/vimrc
```

and removes quotes before `syntax on`.

### Plugin Manager for Vim

The [vim-plug](https://github.com/junegunn/vim-plug) is used here.

#### Installation

```bash
curl -fLo ~/.vim/autoload/plug.vim --create-dirs \
    https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
```

Now configure `Vim` configuration file to support plugins with `vim-plug`:

```.virmrc
" Star of plugin section
" Specify a directory for plugins
" - Avoid using standard Vim directory names like 'plugin'
call plug#begin('~/.vim/plugged')
" Make sure you use single quotes

" Initialize plugin system
call plug#end()
" end of plugin section
```

Example setup:

```.vimrc
" Star of plugin section
" Specify a directory for plugins
" - Avoid using standard Vim directory names like 'plugin'
call plug#begin('~/.vim/plugged')
" Make sure you use single quotes

Plug 'scrooloose/syntastic'

Plug 'tpope/vim-fugitive'

Plug 'pangloss/vim-javascript'

Plug 'scrooloose/nerdtree'

Plug 'tpope/vim-surround'

Plug 'bling/vim-airline'

Plug 'flazz/vim-colorschemes'

" Initialize plugin system
call plug#end()
" end of plugin section
```
For details see: [`vim-plug` installation](https://github.com/junegunn/vim-plug#installation)

![Vim with plugins](https://user-images.githubusercontent.com/14539/33233250-05242628-d213-11e7-82c0-2753e6ccdc70.png)

## Enable USB access

- Edit `config.txt`:

```bash
cd /boot/
sudo vim config.txt
```

Add a line on the end:

```text
dtoverlay=dwc2
```

- Creaty empty `ssh` file if not yet created in `/boot/` directory:

```bash
touch ssh
```

- Edit `cmdline.txt`:

```bash
sudo vim cmdline.txt
```

Add a config item `modules-load=dwc2,g_ether` after `rootwait`:

```text
rootwait modules-load=dwc2,g_ether
```

The content of `cmdline.txt` after modification:

```text
dwc_otg.lpm_enable=0 console=serial0,115200 console=tty1 root=PARTUUID=4cc82cbf-02 rootfstype=ext4 elevator=deadline fsck.repair=yes rootwait modules-load=dwc2,g_ether quiet splash plymouth.ignore-serial-consoles
```

## Docker support

### Docker Installation

Docker support is now official with Raspbian Jessie:

```bash
SUPPORT_MAP="
...
armv6l-raspbian-jessie
...
```

```bash
curl -sSL https://get.docker.com | sh
```

Verify installation:

```bash
sudo docker run hello-world
```

### Removing elevated requirement

```bash
sudo usermod pi -aG docker
logout
```

(and start your terminal session again, e.g. using SSH)

### Example Dockerfile built and run on device

Here is an example of using custom built Docker image to use Blinkt pHAT:

```Dockerfile
FROM resin/rpi-raspbian:jessie

# dependencies
RUN apt-get update -qy
RUN apt-get install -qy python3
RUN apt-get install -qy python3-pip
RUN apt-get install -qy python3-rpi.gpio
# Cancel out any Entrypoint already set in the base image.
ENTRYPOINT []

# Blinkt
RUN apt-get install -qy python3-blinkt

# app code dependecies

RUN pip3 install request

WORKDIR /root/

# COPY library  library
# WORKDIR /root/library
# RUN python3 setup.py install
# RUN pip3 install request

WORKDIR /root/
COPY examples   examples
WORKDIR /root/examples/
# install app specific  requirements (can be cached already)
RUN pip3 install -r requirements.txt
# code
CMD ["python3", "cheerlights.py"]
```


## Node Support

This will install latest version (https://github.com/sdesalas/node-pi-zero):

```bash
wget -O - https://raw.githubusercontent.com/sdesalas/node-pi-zero/master/install-node-v.last.sh | bash
```

This will produce:

```bash
wget -O - https://raw.githubusercontent.com/sdesalas/node-pi-zero/master/install-node-v.last.sh | bash
--2017-11-25 18:05:45--  https://raw.githubusercontent.com/sdesalas/node-pi-zero/master/install-node-v.last.sh
Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 151.101.112.133
Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|151.101.112.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 1703 (1.7K) [text/plain]
Saving to: ‘STDOUT’

-                                       100%[===============================================================================>]   1.66K  --.-KB/s   in 0.001s

2017-11-25 18:05:46 (1.26 MB/s) - written to stdout [1703/1703]

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   99k  100   99k    0     0  75234      0  0:00:01  0:00:01 --:--:-- 75282
--2017-11-25 18:05:48--  https://nodejs.org/dist/v9.2.0/node-v9.2.0-linux-armv6l.tar.gz
Resolving nodejs.org (nodejs.org)... 104.20.23.46, 104.20.22.46, 2400:cb00:2048:1::6814:172e, ...
Connecting to nodejs.org (nodejs.org)|104.20.23.46|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 16896011 (16M) [application/gzip]
Saving to: ‘node-v9.2.0-linux-armv6l.tar.gz’

node-v9.2.0-linux-armv6l.tar.gz         100%[===============================================================================>]  16.11M   878KB/s   in 27s

2017-11-25 18:06:16 (619 KB/s) - ‘node-v9.2.0-linux-armv6l.tar.gz’ saved [16896011/16896011]
```

```bash
$ node --version
v9.2.0
```

### Yarn support

The generic installation script works without a problem. See [Yarn Installation](https://yarnpkg.com/en/docs/install#alternatives-tab):

```bash
curl -o- -L https://yarnpkg.com/install.sh | bash
```

```bash
yarn --version
1.3.2
```

## Hostname Change

Edit configuraction file:

```bash
sudo vim /etc/hostname
```

```bash
sudo /etc/init.d/hostname.sh
sudo reboot
```

## Password Change

> This is a security risk - please login as the 'pi' user and type 'passwd' to set a new password.

```bash
passwd
```

## Network Connection Over USB

On your Mac OS go to `>System Preferences > Sharing`, select correction interface in `Share your connection from` dropdown. Then select `To Computer Using` > `RNDIS/Ethernet Gadget` and finally enable `Internet Sharing` service.

Verify connection from your Raspberry PI:

```bash
ping -c 3 google.com
```

## Audio

### Setup

- Write down your plugged device card and device number:

```bash
arecord -l
```

- Write down your plugged device card and device number:

```bash
aplay -l
```

- create and edit configuration in `/home/pi`:

```bash
touch /home/pi/.asoundrc
sudo vim /home/pi/.asoundrc
```

- add default configuration for sound interface based on previous steps results. Example for Sound Blaster Audio card:

```text
pcm.!default {
  type asym
  capture.pcm "mic"
  playback.pcm "speaker"
}
pcm.mic {
  type plug
  slave {
    pcm "hw:1,0"
  }
}
pcm.speaker {
  type plug
  slave {
    pcm "hw:1,0"
  }
}
```

Verify setup by running `speaker-test`:

```bash
speaker-test -t wav
```

## MongoDB

```bash
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install mongodb-server
```

To allow connection from other machine [bind to local network address](https://docs.mongodb.com/manual/reference/configuration-options/):

```bash
sudo vim /etc/mongodb.conf
```

```txt
bind_ip = 127.0.0.1,192.168.1.13
```

## Services Status Check

If you installed any component as a service and you are interested to check their current status:

```bash
sudo service {SERVICE_NAME} status
```

or list all services statuses:

```bash
sudo service --status-all
```

Here it lists `MongoDB` service as not running:

```bash
 [ + ]  kmod
 [ + ]  lightdm
 [ - ]  mongodb
 [ - ]  motd
 [ - ]  mountall-bootclean.sh
 [ - ]  mountall.sh
 ```

## Increase Swap File Size

 Sometimes when compiling from the source it is usefull to increase swap file size for the duration of the build:

 ```bash
sudo vim /etc/dphys-swapfile
```

Change from defualt `CONF_SWAPSIZE=100` to `CONF_SWAPSIZE=2048`

Restart swap file service:

```bash
sudo /etc/init.d/dphys-swapfile stop
sudo /etc/init.d/dphys-swapfile start
...
free -m
             total       used       free     shared    buffers     cached
Mem:           434        284        149          4         20        173
-/+ buffers/cache:         90        343
Swap:         2047          0       2047
```

## Update Python version

Ref.: [Installing Python 3.6 on Raspbian](https://gist.github.com/dschep/24aa61672a2092246eaca2824400d37f)

## Install TMUX

[tmux wiki](https://github.com/tmux/tmux/wiki)
When using SSH you could detach current session to background using `tmux`:

[tmux cheatsheet](https://gist.github.com/MohamedAlaa/2961058)

## Backup SD Card

```bash
sudo dd if=/dev/disk2 of=~/Desktop/backup.dmg
```

where `/dev/disk2` is a name of a mounted sd card, `~/Desktop/backup.dmg` is a location of the backup file to be created.
After the backup image has been created use Etcher to flush it to new sd card.

## Author

@peterblazejewicz
