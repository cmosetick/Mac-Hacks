# Using ZFS as a Mac OS X Time Machine backup target
### 2015-09-26
Updated by: Phlogi
Original by: Chris Mosetick

## Introduction
The merits of using ZFS as a backup target are too many to list here.
OS X will not _natively_ use anything other than a HFS+ disk for Time Machine backup.
For now, I'll just say that using ZFS is much better than using Apple's [ancient HFS+](http://blog.barthe.ph/2014/06/10/hfs-plus-bit-rot/).

To actually use a ZFS pool as a local backup target, the easiest way is to use a virtual block device from our zfs pool. This is a native feature of ZFS. 


we will have to trick OS X into thinking that a ZFS disk is actually a HFS+ disk. In short, we run HFS+ on top of ZFS. Less than ideal, but it works.


## Requirements
- create a virtual block device on your pool and let OS X format it with HFS

- The disk runs ZFS on bare metal.  
- Use the newest Mac installer from here:  
[https://openzfsonosx.org/](https://openzfsonosx.org)

- or install via Mac [Homebrew](http://brew.sh) (check to make sure Homebrew is up to date first)  `brew search zfs`  
Should return something like:  
`Caskroom/cask/openzfs`


## Set up the disk with ZFS

Substitute `mypool` with whatever you want for your zpool name.
```
zpool create mypool /dev/diskX
zfs set mountpoint=/Volumes/mypool mypool
zfs set compression=lz4 mypool
zfs set atime=off mypool
```
Optional: store two copies of every disk block, twice. Will reduce storage space by 50%, but might make you feel warm.
```
## zfs set copies=2 mypool
```
create the virtual block device
```
zfs create fs create -V 256GB mypool/vdevOSXbackup
```
Now OS X will detect the new disk and ask to initialize it - somply hit that button, choose a name for the partition and format as HFS+.

After that timemachine might popup immediately and let you select the disk, otherwise select it manually from within TimeMachine (open Time Machine in system preferences and you should see the disk as you named it when initializing it with DiskUtil).


## Credits
Big thank you's have to go to all those who have helped to bring ZFS to all the platforms outside of Solaris.  
[OpenIndiana team](http://wiki.openindiana.org/oi/OpenIndiana+Wiki+Home)  
[Illumos team](http://wiki.illumos.org/display/illumos/About+illumos)  
[FreeBSD team](http://freebsd.org)  
[Open ZFS team](http://www.open-zfs.org/wiki/Main_Page)  
[ZFSonLinux team](http://zfsonlinux.com)  
[ZFSonOSX team](https://github.com/openzfsonosx/zfs/graphs/contributors)
