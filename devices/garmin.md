## Mount Garmin devices as storage drives

Add your user to the group `lp`:
```
sudo gpasswd -a <username> lp
```
Create new `udev` rule:
```console
$ sudo vim /etc/udev/rules.d/51-garmin.rules

ATTRS{idVendor}=="091e", ATTRS{idProduct}=="0003", MODE="666"
```

Reload `udev` rules:
```
sudo udevadm control --reload-rules
```