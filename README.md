# Build a standalone i2c-hid module

1. You should have kernel headers installed
```bash
# For gentoo
emerge --ask sys-kernel/linux-headers
# For Arch-based distributions,
sudo pacman -S linux-headers
# For Debian-based distrubtions
sudo apt install linux-headers-$(uname -r)
```
2. Clone this repo with `git clone https://github.com/coiby/standalone_i2c_hid.git`
3. `cd i2c-hid_standalone` twice and run `make` to build the module
4. Unload existing hid module `sudo rmmod i2c_hid`
5. Load new module: `sudo insmod i2c-hid.ko polling_mode=1`

## Revert to the installed module
To revert to the installed module in /lib/modules/`uname -r`,
```bash
# unload current one
sudo rmmod i2c_hid
# use the one installed in /lib/modules/`uname -r`
sudo modprobe i2c_hid
```

## Persist after rebooting

If you want the new module persists after rebooting, you should replace the old i2c-module,

1. Find the path of the old module
```sh
$ modinfo i2c-hid.ko
filename: /lib/modules/5.9.0-1-MANJARO/kernel/drivers/hid/i2c-hid/i2c-hid.ko.xz
```
2. Copy the new module to the path
```sh
$ cp i2c-hid.ko /lib/modules/5.9.0-1-MANJARO/kernel/drivers/hid/i2c-hid/i2c-hid.ko.xz
```
3. For distributions like Ubuntu and Fedora, the driver could be built into the initramfs. So you have to rebuild the initramfs after replacing the old module with new one. On Fedora, run `sudo dracut --force`; On ubuntu, run `sudo update-initramfs -u`.
