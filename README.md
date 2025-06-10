# NTFS-MacOS-13-26
How to write on an NTFS drive on macOS 15 Sequoia and macOS 26 Tahoe, for Apple Silicon, without a kernel module.

I figured this out today and it works on my MacBook Air M2 which is on MacOS 26 Tahoe.

First you need Homebrew. I'll let you find a tutorial to install it.

Then we need some dependencies, run into the terminal:

```bash
brew install autoconf automake libtool libgcrypt pkg-config gettext bash mounty
```

Restart your shell so that your shell use the updated bash, run bash and see if it's 5.0 version, else make sure homebrew binaries are first in your PATH.

Then we need fuse-t, a version of macFuse without any kernel module.

You can download it here: https://www.fuse-t.org/downloads

Or install it with brew:

```bash
brew tap macos-fuse-t/homebrew-cask

brew install fuse-t
```

Then make a symlink (not sure if necessary but do it anyways):

```bash
sudo ln -s /usr/local/lib/libfuse-t.dylib /usr/local/lib/libfuse.2.dylib
```


Now go into a directory of your choice and run

```bash
git clone https://github.com/tuxera/ntfs-3g

cd ntfs-3g
```

We'll need to trick pkg-cache, so run 

```bash
sudo nano /usr/local/lib/pkgconfig/fuse.pc
```

Inside the file, write this:

```bash
prefix=/usr/local
exec_prefix=${prefix}
libdir=${exec_prefix}/lib
includedir=${prefix}/include

Name: fuse
Description: Compatibility wrapper that maps fuse-t -> -lfuse-t
Version: 2.9.9          # anything ≥ 2.6.0 will satisfy the test
Libs:   -F/Library/Frameworks -framework fuse_t -Wl,-rpath,/Library/Frameworks
Cflags: -I/Library/Frameworks/fuse_t.framework/Headers -D_FILE_OFFSET_BITS=64
```

Now run :

```bash
hash -r

autoreconf -fvi

./configure --prefix=/usr/local --with-fuse=external

make -j"$(sysctl -n hw.ncpu)" rootlibdir=/usr/local/lib rootbindir=/usr/local/bin

sudo make install rootlibdir=/usr/local/lib rootbindir=/usr/local/bin

echo user_allow_other | sudo tee /etc/fuse.conf

# Just in case

sudo install_name_tool -add_rpath /Library/Frameworks /usr/local/bin/ntfs-3g
sudo install_name_tool -add_rpath /Library/Frameworks /usr/local/bin/lowntfs-3g
sudo install_name_tool -add_rpath /Library/Frameworks /usr/local/bin/ntfs-3g.probe
```

Now ntfs-3g should be installed.

You have two options:

1 - Mount manually your NTFS partition:

If your NTFS partition is /dev/disk4s3 (check with Disk Utility), do:

```bash
sudo umount /dev/disk4s3

sudo mkdir /Volumes/NTFS

sudo chown $(id -u) /Volumes/NTFS

sudo /usr/local/bin/ntfs-3g /dev/disk4s3 /Volumes/NTFS -o local -o allow_other -o auto_xattr -o big_writes
```

Now go to finder and you should see a new volume called "fuse-t" containing a folder called "NTFS". This is your NTFS drive and you can write in it

2 (preferred) - Mount using Mounty

We installed Mounty, launch it and agree.

Plug your NTFS drive AFTER LAUNCHING MOUNTY and in the toolbar click on the Mounty icon, then you should see "Re-mount", click on it, then click on "mount automatically". 

Now go to finder and you should see a new volume called "fuse-t" containing a folder. This folder is your NTFS drive and you can write in it

Now, when you'll plug your drive and Mounty is launched, it will automatically mount your drive.

If you have any questions or problem, comment, or open an issue on Github, or contact me by mail at leodomecbialek@outlook.fr

Thnaks :)
