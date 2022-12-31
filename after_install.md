# This is a collection of  daily problems and more settings.

## Power Management
### Swap file creation
refer to [swap](https://wiki.archlinux.org/title/Swap#Swap_file) for latest settings.
```bash
# create a swap file the size of your choosing,such as 16G
dd if=/dev/zero of=/swapfile bs=1M count=16384 status=progress
# Set the right permissions
chmod 600 /swapfile
# format it to swap
mkswap -U clear /swapfile
# Activate the swap file
swapon /swapfile
# Finally, edit the fstab configuration to add an entry for the swap file
/etc/fstab
/swapfile none swap defaults 0 0
```

### Configure the initramfs
```bash
/etc/mkinitcpio.conf
# add the resumn after udev
HOOKS=(base udev autodetect keyboard modconf block filesystems resume fsck)
# regenerate the initramfs
mkinitcpio -P
```

### Required kernel parameters
```bash
# Using a swap file requires also setting the resume=UUID=swap_device_uuid and additionally a resume_offset=swap_file_offset kernel parameters

# The following command may be used to identify swap_device_uuid:
findmnt -no UUID -T /swapfile
# The following command may be used to identify swap_file_offset:
filefrag -v /swapfile | awk '$1=="0:" {print substr($4, 1, length($4)-2)}

# add kernel parameters
/etc/default/grub
resume=UUID=swap_device_uuid resume_offset=swap_file_offset

# regenerate grub.cfg
grub-mkconfig -o /boot/grub/grub.cfg

# reboot
reboot
```

### ACPI EVENTS
```bash
# when the power key/button is pressed, enter hibernate mode
/etc/systemd/logind.conf
HandlePowerKey=hibernate

# To apply any changes
sudo systemctl kill -s HUP systemd-logind
```


# GRUB 
To make grub more beautiful , I have installed `catppuccin-mocha-grub-theme-git`.

To let it take effect:
```bash
/etc/default/grub

GRUB_THEME="/usr/share/grub/themes/catppuccin-mocha/theme.txt"
```
## Firefox
about firefox [Hardware video acceleration](https://wiki.archlinux.org/title/Firefox#Hardware_video_acceleration)
```bash
about:config

media.ffmpeg.vaapi.enabled = true
media.av1.enabled = false       # my graphics don't support av1.
```

## Obs-Studio
I have cloned `catppuccin` theme for obs. To enable it you need to open obs setting and select it.

## Problems
### nvidia-gpu:i2c timeout
when I boot,there are four errors appearing.
```bash
nvidia-gpu 0000:01:00.3: i2c timeout error e0000000
ucsi_ccg 0-0008: i2c_transfer failed -110
ucsi_ccg 0-0008: ucsi_ccg_init failed - -110
ucsi_ccg: probe of 0-0008 failed with error -110
```

This problem is related to `nvidia USB type-c`,and now I haven't looked for good solutions.So I just blacklist it.
```bash
/etc/modprobe.d/blacklist-i2c-nvidia-gpu.conf
blacklist ucsi_ccg
```

### texlive
lack of libcrypt.so.1
```bash
paru -S libxcrypt-compat
```
