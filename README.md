# Readally

"Readally" is a portmanteau combining "read-only" and "read all".

Similar to [bindfs](https://bindfs.org/), Readally is a FUSE FileSystem that exposes an altered version of a given directory (aka "the original directory").

Specifically, Readally makes it:

- 100% read-only: any attempt to write or change anything is met with errno 30, i.e. `EROFS: Read-only file system`;
- 100% readable: although each file retains its original owner, group and mode, any file can still be read by any user -- essentially, standard Unix permissions are ignored.

## What for?

Unprivileged backup is one possible use case: the process that backs up your data no longer needs to run as root to read the entirety of a given filesystem.

Solutions like bindfs or ID-mapped mounts also allow this but they alter perceived file ownership, which is not always desirable.

## Is this not dangerous?

Anything that alters file ownership and/or the behaviour of Unix permissions is dangerous.
From this perspective, Readally is as dangerous as bindfs or ID-mapped mounts.

Consequently, these solutions should be used with caution.
A typical approach is to protect the mountpoint's parent directory with regular Unix permissions reflecting who is allowed to access the dataset exposed through Readally.

Example:

```
drwxr-xr-x root   root   /
drwxr-xr-x root   root   mnt
dr-x------ backup root   only_backup_shall_pass
drwx------ root   root   readally_mountpoint
-rw------- root   root   actual_data
```

## Options

### one-file-system

Similar to find's `-xdev` and du's `-x, --one-file-system`, this option makes Readally ignore any file related to a filesystem other than the one holding the original directory.

Default value: disabled.

### banned-types

This option makes Readally ignore a given list of filetypes.
Here, filetypes are neither file extensions nor MIME types but rather `find`-like file types:

Filetypes you likely want to keep:
- `f`: regular files
- `l`: symbolic links

Filetypes you likely want to ignore:
- `b`: block devices
- `c`: character devices
- `p`: named pipes / FIFOs
- `s`: sockets

Alien filetypes:
- `D`: Solaris Doors
- `P`: Solaris event ports
- `W` : whiteouts
- `?`: unknown 

Default value: `bcpsDPW?` i.e. by default Readally exposes only directories, regular files and symbolic links.

## Implementation

 - Python with [fusepy](https://github.com/fusepy/fusepy)

## How to use it

```
readally [-o OPTIONS] [--foreground] /original/directory /mount/point
```

fstab syntax:
```
/original/directory    /mount/point    fuse.readally    banned-types=DPW?,one-file-system    0 0
```
