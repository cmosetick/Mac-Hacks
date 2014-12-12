# Using a local ZFS disk as a Mac OS X Time Machine backup target
### 2014-12-11
by: Chris Mosetick

## Introduction
The merits of using ZFS as a backup target are too many to list here.
OS X will not _natively_ use anything other than a HFS+ disk for Time Machine backup.
For now, I'll just say that using ZFS is much better than using Apple's [ancient HFS+](http://blog.barthe.ph/2014/06/10/hfs-plus-bit-rot/).

To actually use a ZFS disk as a local backup target, we will have to trick OS X into thinking that a ZFS disk is actually a HFS+ disk. In short, we run HFS+ on top of ZFS. Less than ideal, but it works.


## Requirements
- create a sparse bundle on a ZFS disk to be used for "TimeMachine" backup

- The disk runs ZFS on bare metal.  
- Use the newest Mac installer from here:  
[https://openzfsonosx.org/](https://openzfsonosx.org)

- or install via Mac [Homebrew](http://brew.sh) (check to make sure Homebrew is up to date first)
`brew search zfs`  
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
create some file systems on your new disk for general usage if wanted.
```
zfs create mypool/tmp
zfs create mypool/chris
```
create a FS for Time Machine usage
```
zfs create mypool/timemachine
```

## Create the sparse bundle for TM usage

create a sparse bundle with HFS+ Journaled file system.  
set the label of the disk to 'TimeMachine' (so it makes sense in Finder).  
change the -volname (label as appears on the desktop) to whatever makes you happy.
change the -size to whatever you want / need for your setup.
```
cd /Volumes/mypool/timemachine
hdiutil create -size 600g -type SPARSEBUNDLE -fs "HFS+J" -volname TimeMachine TimeMachine.sparsebundle
```

the open command will mount the sparse bundle
```
open TimeMachine.sparsebundle
```
enable "special" ownership
```
sudo diskutil enableOwnership /Volumes/TimeMachine
```
set the sparse bundle as a Time Machine backup destination.
```
sudo tmutil setdestination '/Volumes/TimeMachine'
```
now open Time Machine in system preferences, and you should see the disk "TimeMachine" as the backup target.

## Credits
Big thank you's have to go to all those who have helped to bring ZFS to all the platforms outside of Solaris.  
[OpenIndiana team](http://wiki.openindiana.org/oi/OpenIndiana+Wiki+Home)  
[Illumos team](http://wiki.illumos.org/display/illumos/About+illumos)  
[FreeBSD team](http://freebsd.org)  
[Open ZFS team](http://www.open-zfs.org/wiki/Main_Page)  
[ZFSonLinux team](http://zfsonlinux.com)  
[ZFSonOSX team](https://github.com/openzfsonosx/zfs/graphs/contributors)
