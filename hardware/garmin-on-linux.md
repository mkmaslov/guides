## Mount Garmin watch as a storage drive in Linux

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
The watch should appear in the File Manager.

#### Troubleshooting:
- If the watch constantly disappears and reappears in the File Manager, this may indicate that the **cable is not attached firmly enough or is broken**. The connection is somewhat slow in any case, but it should be rather stable.
- Keep in mind that different Garmin models have different amount of disk space (to store music). **It is not guaranteed that a more expensive model would have more disk space.** For instance, Garmin Vivoactive 3 has 4GB, more expensive Garmin Venu 2S has 8GB, and yet more expensive Garmin Forerunner 745 has only 4GB again.