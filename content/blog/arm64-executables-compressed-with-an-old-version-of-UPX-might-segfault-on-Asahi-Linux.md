+++
title = "Arm64 executables compressed with an old version of UPX might segfault on Asahi Linux"
date = "2025-06-25T17:29:10+02:00"

#
# description is optional
#
# description = "An optional description for SEO. If not provided, an automatically created summary will be used."

tags = ["asahi-linux","upx","golang","staticexecutable","segfault","github-actions",]
+++

## The problem

I came across a problem with an arm64 executable downloaded from a typical GitHub repo's release downloads and trying to run it on Asahi Fedora 42 Linux. The repo is for a Golang project that has a Github Actions CI workflow which builds downloadable release binaries (static executable of said tool).

I've downloaded the arm64 executable, tried to run it on Asahi Fedora Linux and it crashed:

```bash
./<tool>_linux_arm64 
Segmentation fault (core dumped)
```

Wrong architecture? No.

```bash
file ./<tool>_linux_arm64 
./<tool>_linux_arm64: ELF 64-bit LSB executable, ARM aarch64, version 1 (SYSV), statically linked, no section header
```

Can´t handle 16K page size? Yes, seems so as it runs fine in `muvm`.

```bash
$ getconf PAGE_SIZE #Asahi host on 16K
16384


$ muvm -ti -- bash
$ uname -p
aarch64
$ getconf PAGE_SIZE # muvm guest on 4K
4096 

$ file ./<tool>_linux_arm64 --version # executable runs fine on 4K
<tool> version 1.0.0
```

So are there older version of the Go compiler that produce executables that only run on 4K pagesize machines? Well, the executable in question was compiled 4 years ago with Go 1.16. And that Go version definitely produces executables that run both on 4K and 16K (and 64K).

It turns out the executable produced by this project's GitHub workflows pipeline [compresses the executable with `upx`, to reduce the filesize](https://upx.github.io/). The executable that I had downloaded was built 4 years ago on GitHub's `ubuntu:latest` image - so that was probably on Ubuntu 20.04 with upx 3.95.

And indeed when I try to compress an arm64 executable (in an Ubuntu 20.04 x86_64 container) with upx 3.95 it won't run anymore on Asahi Linux (and probably any other machine with non-4K page size).

The same issue doesn't happen if I do the same on an up-to-date OS with a more recent upx version (like Ubuntu 24.04 or Fedora 42). 

(Ubuntu 24.04 is on a upx 4.x version and Fedora 42 on a 5.x version).

## The solution

So the problem can be fixed in two ways: If this is our project and CI pipeline, we can just re-run it with an up-to-date image and toolchain. Compressing our executable with the newer version of `upx` is all it needs.

If we have just downloaded some executable that segfaults and we want to get it to run quickly, we can help ourselves and remove the upx compression. (There's no downside except for a bigger file size).

To do so, install upx on Asahi Fedora:

```bash
dnf install upx
```

We can then test if your broken executable is indeed upx compressed:

```bash
upx -t <file>
```

If it is upx compressed it will say something like `testing <file> [OK]`. If it's not, there will be an error message.

We can then remove the compression:

```bash
upx -d <file>
```

And optionally re-apply the compression with the newer upx version on our system:

```bash
upx <file>
```






