# Static GDB

Process used to create a "static" gdb for ARM:

```
$ docker run --rm -it -v "$(pwd):/host" -w /host arm32v7/ubuntu:18.04
/host# apt update && apt install gdb binutils file nano dpkg-dev
/host# # uncomment the deb-src files in /etc/apt/sources.list
/host# apt source gdb
/host# mkdir gdb-8.1.1/build-gcc
/host# cd gdb-8.1.1/build-gcc/
/host# ../configure --enable-static --disable-interprocess-agent
/host# make -j8
/host# cp /lib/ld-linux-armhf.so.3 /lib/arm-linux-gnueabihf/libdl.so.2 /lib/arm-linux-gnueabihf/libm.so.6 /lib/arm-linux-gnueabihf/libc.so.6 .
```

This doesn't seem to make it entirely static. It still needs those four system libraries :shrug:

The resulting files now on the host are:

```
$(pwd)/gdb-8.1.1/build-gcc/
	gdb/gdb
	ld-linux-armhf.so.3
	libc.so.6
	libm.so.6
	libdl.so.2
```

The first binary has been renamed to `gdb-arm32v7-ubuntu-18.04` in this folder. If you want to copy and run this on your target machine, then you'll need to copy the other four files from that ubuntu container.

Copy them to target host and run with an LD_LIBRARY_PATH

```
root@target:~# ls -l
-rwxr-xr-x    1 root     root      61669156 Oct 25 03:12 gdb
-rwxr-xr-x    1 root     root        106784 Oct 25 03:12 ld-linux-armhf.so.3
-rwxr-xr-x    1 root     root        947300 Oct 25 03:14 libc.so.6
-rw-r--r--    1 root     root          9740 Oct 25 03:14 libdl.so.2
-rw-r--r--    1 root     root        461344 Oct 25 03:14 libm.so.6
root@target:~# LD_LIBRARY_PATH="/home/root" ./gdb
GNU gdb (GDB) 8.1.1
Copyright (C) 2018 Free Software Foundation, Inc.
...
```

# References

https://stackoverflow.com/questions/9364685/how-i-can-static-build-gdb-from-source
