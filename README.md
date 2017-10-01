# Raspberry PI setup

Setting up a Raspberry PI (macOS)

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

## Author

@peterblazejewicz