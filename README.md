# Build a standalone i2c-hid module

1. You have should have kernel headers installed
```bash
# For gentoo
emerge --ask sys-kernel/linux-headers
# For Arch-based distributions,
sudo pacman -S linux-headers
# For Debian-based distrubtions
sudo apt install linux-headers-$(uname -r)
```
2. Clone this repo
3. `cd i2c-hid_standalone` and run make to build the module
4. Unload existing hid module `sudo rmmod i2c_hid`
5. Load new module: `sudo insmod i2c-hid.ko polling_mode=1`

To revert to the installed module in /lib/modules/`uname -r`,
```bash
# unload current one
sudo rmmod i2c_hid
# use the one installed in /lib/modules/`uname -r`
sudo modprobe i2c_hid
```
