For a broken touchpad, it may take several months or longer to be fixed.
Polling mode could be a fallback solution for enthusiastic Linux users
when they have a new laptop. It also acts like a debugging feature. If
polling mode works for a broken touchpad, we can almost be certain
the root cause is related to the interrupt or power setting.

This patch could fix touchpads of Lenovo AMD gaming laptops including
Legion-5 15ARH05 (R7000), Legion-5P (R7000P) and IdeaPad Gaming 3
15ARH05.

When polling mode is enabled, an I2C device can't wake up the suspended
system since enable/disable_irq_wake is invalid for polling mode.

Three module parameters are added to i2c-hid,
    - polling_mode: by default set to 0, i.e., polling is disabled
    - polling_interval_idle_ms: the polling internal when the touchpad
      is idle, default to 10ms
    - polling_interval_active_us: the polling internal when the touchpad
      is active, default to 4000us

User can change the last two runtime polling parameter by writing to
/sys/module/i2c_hid/parameters/polling_interval_{idle_ms,active_us}.

Note xf86-input-synaptics doesn't work well with this polling mode
for the Synaptics touchpad. The Synaptics touchpad would often locks
into scroll mode when using multitouch gestures [1]. One remedy is to
decrease the polling interval.


## Build a standalone i2c-hid module

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
3.  Append i2c_hid.polling_mode=1 to kernel commandline
    a. edit /etc/default/grub/
    b. append "i2c_hid.polling_mode=1" to GRUB_CMDLINE_LINUX
    c. run `sudo update-grub`
4. For distributions like Ubuntu and Fedora, the driver could be built into the initramfs. So you have to rebuild the initramfs after replacing the old module with new one. On Fedora, run `sudo dracut --force`; On ubuntu, run `sudo update-initramfs -u`. 
