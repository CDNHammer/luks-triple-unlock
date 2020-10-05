# luks-triple-unlock
Script to set up the unlocking of an encrypted LVM Debian install through the console, a flash drive, or via remote SSH.

*Use at your own risk, I'm not responsible for any damage this script might do to your system, make backups, make sure you have a safe boot option, test it in a VM first, etc.

Tested on:
- Debian 10.6 (headless)

Usage:
- Install Debian with full disk encrypted LVM
- `sudo apt-get install -y git`
- `git clone https://github.com/CDNHammer/luks-triple-unlock.git && cd luks-triple-unlock`
- `sudo ./install.sh [keyfile]`   
It will ask you for the passphrase for the luks drive. \[keyfile\] is a path to a file you want to use as a key for the luks volume. This file will be read at boot from a flash drive partition formatted with ext(2/3/4)/fat32/ntfs. If no keyfile provided on the commandline, a file `.keyfile` will be generated in the current directory.
- `sudo reboot`

Options to unlock your machine:
- Console
- SSH. Copy /etc/initramfs-tools/root/.ssh/id_rsa, this is the private key you need to log into dropbear (no password, root@host). When you connect it will ask you for the passphrase to unlock the machine.
- Removable USB flash drive. Copy `.keyfile` (or the file you provided on the commandline to ./install.sh) to any ext(2/3/4)/fat32/ntfs partition onto a flash drive. Plug it into the machine and boot. It should automatically detect the keyfile and unlock the system.

Optional:
- edit /etc/dropbear-initramfs/config, uncomment and edit `DROPBEAR_OPTIONS=""` to add `"-s -j -k -I 60 -p 4748"`. `-s` disallows password logins, `-j` and `-k` prevent port forwarding, `-I <seconds>` sets a timeout, and `-p` sets the ssh port. I recommend using an alternate ssh port such as port 4748. Otherwise you might encounter warnings about a possible man-in-the-middle attack since you shouldn't be sharing your actual host keys with dropbear (they'd be stored decrypted in /boot).
- the ip address wil be set by dhcp. If you don't have static ips configured through your router for your host MAC, you have multiple network interfaces, or just want to set a fixed ip you should probably edit /etc/initramfs-tools/conf.d/dropbear and change the IP value:
```DROPBEAR=y
  # See http://www.kernel.org/doc/Documentation/filesystems/nfs/nfsroot.txt.
  #IP=<client-ip>:<server-ip>:<gw-ip>:<netmask>:<hostname>:<device>:<autoconf>
  #IP=10.10.1.199::10.10.1.1:255.255.255.0::eth0:off
  #IP=192.168.1.99::192.168.1.1:255.255.255.0::wlan0:off
  #IP=192.168.1.99::192.168.1.1:255.255.255.0::wlan0:dhcp
  #IP=:::::wlan0:dhcp
  #IP=dhcp
```
- `sudo update-initramfs -u` to apply the changes.
