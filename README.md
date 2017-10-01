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


## Author

@peterblazejewicz