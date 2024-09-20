# Unbrick your Sovol touchscreen

The Sovol touchscreen of the SV07 and SV07 Plus are based on Makerbase displays
with their custom Armbian builds. When you manually run
`apt-get update; apt-get upgrade` on one via SSH, it has the potential to break
the SPI driver because of a pending kernel update from a 5.x kernel to a 6.x
(`22.05.0-trunk` to `24.5.1`). This will also make the display non-functional.

## Possible problems

### The backlight turns on, no image is shown, but mainsail still works

Alright, this one is pretty easy to fix.

1. Make a backup of all configuration files as the update might reset them.
2. Download the firmware update `armbian-update.deb` for your SV07/SV07 Plus
   from <https://www.sovol3d.com/pages/Download>
3. Format a USB stick (> 64MB) with a FAT32 partition and MBR partition table
   (usually standard when formatting in Windows)
4. Plug the USB stick into the touchscreen
5. Wait until 15 minutes (it usually takes 5 minutes, but you definitely don't
   want to break it even more)
6. Remove the USB stick from the touchscreen and toggle the power off and on

You should have a working touchscreen again. If not, please open an issue.

### The backlight doesn't turn on and mainsail doesn't work

You might have bricked your entire OS install if you removed the power of the
touchscreen too early while updating, but this is fixable.

1. Download the base image for your printer:
    - SV07: <n/a, waiting for support to respond>
    - SV07 Plus: <https://mega.nz/file/LAFgCCQJ#GjC4pgNtD3FL30aAo76nSrlvqs0ib8BT4Od0UDwzabc>
      It's an outdated image and needs to get updated after installation.
      You can use this with your SV07, but you have to change the printer.cfg.
2. Flash it into a USB stick via Balena Etcher or `dd`
  `dd` command: `dd bs=4M status=progress if=./SV07.img of=/dev/USBSTICK`
  (make sure to adjust the target device after checking `lsblk`)
3. Plug your PC into the USB-C port of the touchscreen
4. Configure a serial terminal program to read the newly discovered serial port
   at `1500000` baud:
    - Windows: Configure PuTTY to use the new COM port and `1500000` baud
    - Linux: `minicom --baudrate 15000000 --device /dev/ttyUSB0`
      (you might have to modify the `ttyUSB0`, just check `dmesg`, it should
      tell you on what device node the serial port got attached to)
5. Turn on the printer and wait 30 seconds
6. Plug the USB stick into your touchscreen
7. Type `run usb_boot` and press <kbd>Enter</kbd>
8. Wait a bit, the display should turn on and show you the stock set up page.
   Don't interact with it. Your are running off of the USB stick right now.
9. In your serial terminal, log in with username `mks` and password `makerbase`
10. If you have a Linux PC, you can now pull a backup of the integrated storage
    to extract your files out later. Plug a USB-to-Ethernet adapter into the
    touchscreen.
    1. Run `ip a` and look for the touchscreen's IP.
    2. On your PC, run the following command:
       `ssh mks@<IP> "dd if=/dev/mmcblk1 | gzip -1 -" | dd of=./sv07-backup.gz`
    3. In the newly created `sv07-backup.gz` file, you have the entirety of
       the internal storage. You can mount it like so:

       ```bash
       gunzip --keep ./sv07-backup.gz
       sudo losetup -v -P /dev/loop77 "$PWD/sv07-backup"
       sudo kpartx /dev/loop77
       sudo mount --mkdir /dev/loop77p2 /mnt/sv07-rootfs
       ```

       Now you can open /mnt/sv07-backup in your file explorer and see all their
       files.

11. Copy the contents of the USB stick back onto the integrated storage
    `dd status=progress bs=4M if=/dev/sda of=/dev/mmcblk1`
12. When that's done, power off the touchscreen by running `sync && poweroff`
13. Unplug the USB stick from the touchscreen
14. Turn the touchscreen back on by pressing the button on the side

You should have a working touchscreen again. If not, please open an issue.

## In case you want to make your own image

You can pull a full disk image from your touchscreen onto your Linux PC pretty
easily in case you ever need it for something later.

`ssh mks@<IP> "dd if=/dev/mmcblk1 | gzip -1 -" | dd of=./backup.gz`
