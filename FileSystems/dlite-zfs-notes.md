# Sparse HFS+ / ZFS image for dlite.

* create an image file of 20GB sparse that is empty
* attach image file to system as if it were a real disk
* create a zpool that lives inside the image and mount it to a specified location

# Part 1 - creation
```
mkdir $HOME/.dlite ; cd $HOME/.dlite

hdiutil create -size 20g -volname dlitepool -type SPARSE -layout NONE dlitepool  # create an empty image to fill up with ZFS

du -sh dlitepool.sparseimage
#  4.0K dlitepool.sparseimage  # Small!!

hdiutil attach -nomount dlitepool.sparseimage
#  /dev/disk2  # device identifier will very based on your system and connected internal/external disks

sudo zpool create -f -m ~/.dlite/dlitepool dlitepool /dev/disk2
# I'm using /dev/disk2 because that's what it is on my system.
# The pool is created now, and thinks it has 20GB of space to work with!

# Lets check the disk usage, now that we've put a file system inside the empty image
du -sh dlitepool.sparseimage
#  13M	dlitepool.sparseimage  # It's only really consuming 13M on the HFS+ volume it really lives on!

sudo chown -R chris:staff ~/.dlite/dlitepool  # take ownership of the FS

du -sh ~/.dlite/dlitepool  # The actual POSIX usable location where the zpool was mounted. Child FS can be created underneath.
#  958K	/Users/chris/.dlite/dlitepool
```
# Print out the free space of file systems
Note the last line, there is 20GB for dlite, but it's not actually consuming 20GB.
```
df -h
Filesystem      Size   Used  Avail Capacity   iused     ifree %iused  Mounted on
/dev/disk1     931Gi  467Gi  463Gi    51% 122542734 121432976   50%   /
devfs          198Ki  198Ki    0Bi   100%       686         0  100%   /dev
map -hosts       0Bi    0Bi    0Bi   100%         0         0  100%   /net
map auto_home    0Bi    0Bi    0Bi   100%         0         0  100%   /home
dlitepool       19Gi  955Ki   19Gi     1%        83  40376348    0%   /Users/chris/.dlite/dlitepool
```

# Optional cleanup / destroy of everything we just created
```
cd $HOME/.dlite
sudo zpool destroy dlitepool
hdiutil detach /dev/disk2  # use the correct device identifier for your system
rm -rf dlitepool
rm -rf dlitepool.sparseimage
ls /dev/disk2
# no such file or directory
# pool is removed and gone
```

# Questions
* Will it survive a reboot!?

* What happens if the image is not attached at /dev/disk2? (this is why Linux has complex systems for managing device locations)

* Should a file system structure be created, as is typical for ZFS?
(in ZFS filesystems are cheap, like dirs everywhere else, but with benefits of snapshots and send/receive command, etc.)
```
sudo zfs create dlitepool/dhyve      # for dyhveos itself
sudo zfs create dlitepool/images     # for docker images
sudo zfs create dlitepool/containers # for running containers
```

A user could then `zfs snapshot ; zfs send` their entire image FS to another location or server if they wanted to,
eliminating the need to have every single image in Hub or a private registry at that exacte moment in time.

