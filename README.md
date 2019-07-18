# pids - Digital Signage for the Raspberry Pi

## What is this?
**pids** is a simple way to use a [Raspberry Pi](https://www.raspberrypi.org/) to display a slideshow, with little to no configuration needed. It's built on top of [Raspbian](https://www.raspbian.org/) and uses [FIM](https://www.nongnu.org/fbi-improved/) to display images.

**pids** has been tested on both the Model 2B and Model 3B devices.

## Running
* Download the latest image from https://pids.nsnw.ca/
* Uncompress and write the image to an SD card
* Plug in a USB storage device (such as a USB key) into a Pi, and boot it off the SD card
* Watch your image slideshow

**NOTE**: This is pretty alpha quality. It works, but I take no responsibility for any pain, suffering, destruction and/or disaster that may happen as a result. In particular, the configuration file is simply `source`d from the attached USB device with zero sanity checks.

## How does it work?
A small script (**pids**) runs on boot which does the following:-

* checks for the existence of device at `/dev/sda`, assuming it will be a USB storage device.
* mounts `/dev/sda1` at `/mnt/usb`.
* checks for the existence of a configuration file (`pids.conf`) on the USB storage device.
* displays the images (using `fim`) on the USB storage device in a loop, and in a random order.

Prebuilt images are available (see https://pids.nsnw.ca/), or you can use the files in this repo to integrate it into your own image. The files are:-

* `pids` - the main **pids** script
* `pids.service` and `pids-boot.service` - unit files for systemd
* `pids.conf` - an example configuration that can be put on a USB storage device
* `config.txt` and `cmdline.txt` - modified configuration files that live in `/boot`

By default, both `pids` and the `pids-boot` service look for `pids.png` and `pids-nousb.png` in `/usr/local/share/pids` respectively. These are shown on boot (by `pids-boot`) and when no USB storage device is detected (by `pids`).

For `config.txt` and `cmdline.txt`, do not just copy these to your Pi's `/boot` directory. There may be options you have configured differently on your Pi, and this will overwrite them. Neither of these files are required to make **pids** work - they just set a few options to make it work better:-

* `config.txt`
  * `disable_overscan=1` - this disables the black border around the screen. By default this is set to `0`, so if you have problems with the output you can comment this line out.
  * `disable_splash=1` - this disables any splash screens that Raspbian might display on boot.
* `cmdline.txt`
  * `logo.nologo` - this disables the Raspberry Pi logo that is displayed at the top of the screen on boot.
  * `consoleblank=0` - this disables console blanking, which normally happens after a period of inactivity.
  * `loglevel=1` - this configures the boot logging to only show important messages.
  * `vt.global_cursor_default=0` - this disables the blinking cursor.
  * `quiet` - this quietens a few other things that might otherwise be displayed on boot.

## Configuration
As mentioned above, **pids** will look for a configuration file on the attached USB storage device. The file it looks for is called `pids.conf`, and it will look for it in the root of the USB storage device.

This file can contain a few options:-

* `INTERVAL`, which tells `fim` how long to show each image for, in seconds. The default is **5** seconds.
* `RANDOM`, which tells `fim` whether to randomise the images. The default is **1** (enabling randomisation).
* `QUIET`, which tells `fim` whether to show details about each image. The default is **1** (enabling quiet mode).
* `DISABLED`, which tells `pids` to exit on boot and not run any further. The default is **0** (not disabled).
* `SSH_KEY`, which if specified should be an SSH key in the same format as found in `~/.ssh/authorized_keys`. This will be placed into `/home/pi/.ssh/authorized_keys` on the Pi to allow login as the `pi` user.
* `PERSISTENT`, which tells `pids` to copy all the data off the USB storage device to allow subsequent reboots to happen without the USB storage device. The default is **0** (don't persist).
* `URL`, which if specified should be a URL pointing to a list of images for `pids` to download.
* `REFRESH`, which will cause `pids` to redownload the images from `URL` after the given number of seconds. By default this is not set, which means it will not redownload images until the next reboot.

By default, the `pi` user does not have a password set. This is to disable SSH login *unless* an SSH key is provided using the `SSH_KEY` option above. The only exception to this is if there's a problem on boot - for more information see the troubleshooting section below.

At the moment, only `eth0` on the device is used, and it will attempt to configure itself via DHCP. Static network and wireless configuration is not yet supported, but is planned for a future release.

## Troubleshooting
By default, the `pi` user does not have a password set. If one is set manually, it will be deleted again the next time the Pi is rebooted. This prevents remote logins as the `pi` user unless an SSH key is specified in the `pids.conf` file with the `SSH_KEY` option. However, if a USB storage device is not connected on boot or there is another problem, `pids` will set the password of the `pi` user to `pids`, allowing SSH login with this password.

`pids` itself is started on boot by systemd. The `pids-boot` service also runs, but much earlier in the boot process, to display the splash screen.

Right now, `pids` is hard-coded to look for `/dev/sda`. This means that if you have multiple USB storage devices connected, it will only use the first one.

## Bugs, limitations and development
If you have any ideas for features, feel free to open an issue on GitHub and I'll take a look.

## Copyright
The files in this repository are **©2019 Andy Smith** (andy@nsnw.ca), and are released under the [MIT License](https://opensource.org/licenses/MIT).
