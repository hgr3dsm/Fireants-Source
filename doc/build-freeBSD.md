# FreeBSD Build Guide

**Updated for FreeBSD [12.2](https://www.freebsd.org/releases/12.2R/announce.html)**

This guide describes how to build fireantsd, command-line utilities on FreeBSD.
we ignore the build of GUI, because FreeBSD is mostly used on servers

## Dependencies

The following dependencies are **required**:

 Library                                                               | Purpose          | Description
 ----------------------------------------------------------------------|------------------|---------------------------------------------
 [autoconf](https://svnweb.freebsd.org/ports/head/devel/autoconf/)     | Build            | Automatically configure software source code
 [automake](https://svnweb.freebsd.org/ports/head/devel/automake/)     | Build            | Generate makefile (requires autoconf)
 [libtool](https://svnweb.freebsd.org/ports/head/devel/libtool/)       | Build            | Shared library support
 [pkgconf](https://svnweb.freebsd.org/ports/head/devel/pkgconf/)       | Build            | Configure compiler and linker flags
 [git](https://svnweb.freebsd.org/ports/head/devel/git/)               | Clone            | Version control system
 [gmake](https://svnweb.freebsd.org/ports/head/devel/gmake/)           | Compile          | Generate executables
 [boost-libs](https://svnweb.freebsd.org/ports/head/devel/boost-libs/) | Utility          | Library for threading, data structures, etc
 [libevent](https://svnweb.freebsd.org/ports/head/devel/libevent/)     | Networking       | OS independent asynchronous networking
 [db5](https://svnweb.freebsd.org/ports/head/databases/db5/)           | Berkeley DB      | Wallet storage (only needed when wallet enabled)
 [libzmq4](https://svnweb.freebsd.org/ports/head/net/libzmq4/)         | ZMQ notification | Allows generating ZMQ notifications (requires ZMQ version >= 4.0.0)
 [sqlite3](https://svnweb.freebsd.org/ports/head/databases/sqlite3/)   | SQLite DB        | Wallet storage (only needed when wallet enabled)
 [python3](https://svnweb.freebsd.org/ports/head/lang/python3/)        | Testing          | Python Interpreter (only needed when running the test suite)

## Preparation

### 1. Install Required Dependencies
Install the required dependencies the usual way you [install software on FreeBSD](https://www.freebsd.org/doc/en/books/handbook/ports.html) - either with `pkg` or via the Ports collection. The example commands below use `pkg` which is usually run as `root` or via `sudo`. If you want to use `sudo`, and you haven't set it up: [use this guide](http://www.freebsdwiki.net/index.php/Sudo%2C_configuring) to setup `sudo` access on FreeBSD.

```bash
pkg install autoconf automake boost-libs git gmake libevent libtool pkgconf db5 sqlite3 python3 OpenSSL miniupnpc libzmq4 help2man

```

### 2. Clone FireAnts Repo
Now that `git` and all the required dependencies are installed, let's clone the Fireants-Source repository to a directory. All build scripts and commands will run from this directory.
``` bash
git clone https://github.com/hgr3dsm/Fireants-Source.git fireants
```
change to the working directory:
```bash
cd fireants/
```
Also we need to set some rights for now:
```bash
chmod +x autogen.sh configure.ac depends/config.guess depends/config.site.in depends/config.sub share/genbuild.sh
```

## Building FireAnts

### 1. Configuration

First we need to set some basics:
```bash
setenv AUTOCONF_VERSION 2.69 # use your installed version here, check with "autoconf --version"

setenv AUTOMAKE_VERSION 1.16  # use your installed version here, check with "automake --version"

setenv BITCOIN_ROOT /root/fireants/ # use your full path to fireants/ directory
```

There are many ways to configure FireAnts, here are a few common examples:
##### Wallet (BDB + SQlite) Support, No GUI:
This explicitly enables legacy wallet support and disables the GUI. If `sqlite3` is installed, then descriptor wallet support will be built.
```bash
./autogen.sh
./configure --with-gui=no --with-incompatible-bdb \
    BDB_LIBS="-ldb_cxx-5" \
    BDB_CFLAGS="-I/usr/local/include/db5" \
    MAKE=gmake
```

##### No Wallet or GUI
``` bash
./autogen.sh
./configure --without-wallet --with-gui=no MAKE=gmake
```

### 2. Compile
**Important**: Use `gmake` (the non-GNU `make` will exit with an error).

```bash
gmake # use "-j N" for N parallel jobs
gmake check # Run tests if Python 3 is available
```

### 3. Installing

strip all bins:
```bash
strip src/fireantsd
strip src/fireants-cli
strip src/fireants-tx
```
copy bins to /usr/local/bin/:
```bash
cp src/fireantsd /usr/local/bin
cp src/fireants-cli /usr/local/bin
cp src/fireants-tx /usr/local/bin
```

thats it!