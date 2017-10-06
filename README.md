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

### Content of `wpa_supplicant.conf`:

```text
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1

network={
    ssid="YOURSSID"
    psk="YOURPASSWORD"
    scan_ssid=1
}
```

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

## Node Support

This will install latest version (https://github.com/sdesalas/node-pi-zero):

```bash
wget -O - https://raw.githubusercontent.com/sdesalas/node-pi-zero/master/install-node-v.last.sh | bash
```

## Hostname Change

```bash
sudo vim /etc/hosts
```

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

## Author

@peterblazejewicz
