# WIFI

## Packages
```
apt-get install wireless-tools wpasupplicant
```

## Configuration

### `/etc/network/interfaces`

```
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet dhcp

allow-hotplug wlan0
auto wlan0
iface wlan0 inet manual
wpa-roam /etc/wpa_supplicant/wpa_supplicant.conf
iface default inet dhcp
```

### `/etc/wpa_supplicant/wpa_supplicant.conf` for WPA2

```
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
fast_reauth=1
network={
    ssid="<SSID>"
    scan_ssid=1
    key_mgmt=IEEE8021X WPA-EAP
    eap=PEAP
    identity="<ID>"
    password="<PASSWORD>"
    phase2="auth=MSCHAPV2"
    eapol_flags=0
}
```

### Turn off WIFI Power Management
Install `pm-utils` package through `apt-get install pm-utils`

Add file `/etc/pm/power.d/wireless`

```sh
#!/bin/sh

/sbin/iwconfig wlan0 power off
```
Run `chmod +x /etc/pm/power.d/wireless`