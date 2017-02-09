# docs

## Raspberry PI + OS images

### Clone image over to SD card in OSX

- `diskutil list` - to check the name of your SD card, eg: `/dev/disk3`
- `df -h` - to see if SD card partitions are mounted (will error with `dd: /dev/rdisk3: Resource busy` if still mounted)
- `diskutil unmountDisk disk3` - to unmount SD card partitions if necessary
- `sudo su` - switch to root
- `dd bs=1m if=/PathToImgFile | pv -s 8g | dd bs=1m of=/dev/rdisk3` - copy backed up bootable OS image to the SD card, here `pv -s 8g` helps to see progress when image size is known in advance, otherwise `pv` can be used. `/dev/rdisk3` is attached SD card, `rdisk3` is used instead of `disk3` as it is going to be faster. This can be shortened if `pv` is not used - `dd bs=1m if=/PathToImgFile of=/dev/rdisk3`

### Backup SD card in OSX

- `diskutil list` - to check the name of your SD card, eg: `/dev/disk3`
- `sudo dd if=/dev/disk3 bs=1m of=/pathToFolder/fileName.img.gz` - without compression and progress indicator, see below for alternative
- `sudo dd if=/dev/disk3 bs=1m | pv -s 16g | gzip > /pathToFolder/fileName.img.gz` - uses compression and reduces image size as otherwise images are going to be the same size as SD card size, compression will collapse free space in cards. Here `-s 16g` is the size of SD card.

