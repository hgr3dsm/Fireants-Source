# macOS Build Guide

> **Updated for macOS Monterey [12.3](https://www.apple.com/macos/monterey/)**  
> Tested on OS X 10.8 through 12.3.1 on 64-bit _Intel processors only_.

This guide describes how to build fireantsd, command-line utilities, and the 
GUI application on macOS.

## Preparation

The commands in this guide should be executed in a Terminal application.
macOS comes with a built-in Terminal located in:

```
/Applications/Utilities/Terminal.app
```

### 1. Xcode Command Line Tools

The Xcode Command Line Tools are a collection of build tools for macOS.
These tools must be installed in order to build Fireants from source.

To install, run the following command from your terminal:

``` bash
xcode-select --install
```

Upon running the command, you should see a popup appear.
Click on `Install` to continue the installation process.

### 2. Homebrew Package Manager

Homebrew is a package manager for macOS that allows one to install packages 
from the command line easily. While several package managers are available for 
macOS, this guide will focus on Homebrew as it is the most popular. Since the 
examples in this guide which walk through the installation of a package will 
use Homebrew, it is recommended that you install it to follow along. Otherwise,
you can adapt the commands to your package manager of choice.

To install the Homebrew package manager, see: https://brew.sh

Note: If you run into issues while installing Homebrew or pulling packages, 
refer to [Homebrew's troubleshooting](https://docs.brew.sh/Troubleshooting).

### 3. Install Required Dependencies

The first step is to download the required dependencies.
These dependencies represent the packages required to get a barebones 
installation up and running.

See [dependencies.md](dependencies.md) for a complete overview.

To install the initial packages, run the following from your terminal:

``` bash
brew install automake libtool boost --c++11 pkg-config protobuf libevent
```

#### Install OpenSSL 1.0

OpenSSL is no longer supported in macOS so we need to install this from a 
Homebrew tap:

```shell
brew install rbenv/tap/openssl@1.0
```

### 4. Clone Fireants repository

`git` should already be installed by default on your system.
Now that all the required dependencies are installed, let's clone the Fireants 
Core repository to a directory. All build scripts and commands will run from 
this directory.

``` bash
git clone https://github.com/hgr3dsm/Fireants-Source.git
```

### 5. Install Optional Dependencies

#### Wallet Dependencies

It is not necessary to build wallet functionality to run `fireantsd` or 
`fireants-qt`.

##### Install Berkley DB 6.2

Berkley DB 6.2 is required for the wallet to function in the GUI. Unfortunately
Homebrew does not have a port of version 6.2 and only support the most current 
version of DB as well as DB 4. This creates somewhat of an issue but luckily 
you have two options when it comes to installing DB 6.2 – either via MacPorts, 
or building from source. 

Installing from MacPorts is the easiest way however for those who do not want 
to have both Homebrew and MacPorts on their Mac then building from source is 
the only other way to go.

###### Install from MacPorts

You can install MacPorts from here <https://www.macports.org/install.php>. Once
you have installed MacPorts you can run the following command:

```shell
sudo port install db62
```

###### Install from Source

If you opt not to install MacPorts then you will need to compile Berkley DB 6.2
yourself. Follow the instructions contained in 
[doc/build-unix.md](https://github.com/justinhartman/Fireants-Source/blob/main/doc/build-unix.md#berkeley-db).

Run the following commands in your Fireants root directory:

```shell
BITCOIN_ROOT=$(pwd)

# Pick some path to install BDB to, here we create a directory within the fireants directory
$ BDB_PREFIX="${BITCOIN_ROOT}/build"
$ mkdir -p $BDB_PREFIX

# Fetch the source and verify that it is not tampered with
$ wget 'http://download.oracle.com/berkeley-db/db-6.2.32.tar.gz'
$ echo 'a9c5e2b004a5777aa03510cfe5cd766a4a3b777713406b02809c17c8e0e7a8fb  db-6.2.32.tar.gz' | sha256sum -c
# -> db-6.2.32.tar.gz: OK
$ tar -xzvf db-6.2.32.tar.gz

# Build the library and install to our prefix
$ cd db-6.2.32/build_unix/
#  Note: Do a static build so that it can be embedded into the executable, instead of having to find a .so at runtime
$ ../dist/configure --enable-cxx --disable-shared --with-pic --prefix=$BDB_PREFIX
$ make install
```
---

#### GUI Dependencies

##### Qt

Fireants includes a GUI built with the cross-platform Qt Framework.
To compile the GUI, we need to install `qt@5`.
Skip if you don't intend to use the GUI.

``` bash
brew install qt@5
```

Ensure that the `qt@5` package is installed and not the `qt` package (which is 
currently Qt6). If **qt** is installed, the build process will fail so you 
need to remove the `qt` package with the following command:

``` bash
brew uninstall qt
```

> **Note:** Building with Qt binaries downloaded from the Qt website is not officially supported.
> See the notes in [#7714](https://github.com/bitcoin/bitcoin/issues/7714).

##### qrencode

The GUI can encode addresses in a QR Code. To build in QR support for the GUI, install `qrencode`.
Skip if not using the GUI or don't want QR code functionality.

``` bash
brew install qrencode
```
---

#### Port Mapping Dependencies

##### miniupnpc

miniupnpc may be used for UPnP port mapping.
Skip if you do not need this functionality.

``` bash
brew install miniupnpc
```
---

#### ZMQ Dependencies

Support for ZMQ notifications requires the following dependency.
Skip if you do not need ZMQ functionality.

``` bash
brew install zeromq
```

ZMQ is automatically compiled in and enabled if the dependency is detected.
Check out the [further configuration](#further-configuration) section for 
more information.

For more information on ZMQ, see: [zmq.md](zmq.md)

---

#### Test Suite Dependencies

There is an included test suite that is useful for testing code changes when 
developing. To run the test suite (recommended), you will need to have Python 3
installed:

``` bash
brew install python
```

---

#### Deploy Dependencies

You can deploy a `.dmg` containing the Fireants application using 
`make deploy`. This command depends on a couple of python packages, so it is 
required that you have `python` installed.

Ensuring that `python` is installed, you can install the deploy dependencies by
running the following commands in your terminal:

``` bash
pip3 install ds_store mac_alias
```

## Building Fireants

### 1. Configuration

There are many ways to configure Fireants, here are a few common examples:

#### Wallet and GUI Application (no tests)

This explicitly enables the Fireants Qt application with wallet support and 
disables the tests. This also builds the `fireantsd`, `fireants-cli` and 
`fireants-tx` binaries.

``` bash
./autogen.sh
# If you installed Berkley via MacPorts then run:
./configure --with-gui=qt5 --disable-gui-tests --disable-tests --disable-bench --no-recursion
# Else, if you installed Berkley via source then run:
./configure LDFLAGS="-L${BDB_PREFIX}/lib/" CPPFLAGS="-I${BDB_PREFIX}/include/" --with-gui=qt5 --disable-gui-tests --disable-tests --disable-bench --no-recursion
```

#### Wallet Support, No GUI Application

Builds `fireantsd`, `fireants-cli` and `fireants-tx` without Qt app:

``` bash
./autogen.sh
# If you installed Berkley via MacPorts then run:
/configure --with-gui=no
# Else, if you installed Berkley via source then run:
./configure LDFLAGS="-L${BDB_PREFIX}/lib/" CPPFLAGS="-I${BDB_PREFIX}/include/" --with-gui=no
```

#### No Wallet or GUI

``` bash
./autogen.sh
./configure --without-wallet --with-gui=no
```

#### Further Configuration

You may want to dig deeper into the configuration options to achieve your desired behavior.
Examine the output of the following command for a full list of configuration options:

``` bash
./configure -help
```

### 2. Compile

After configuration, you are ready to compile. Run the following in your 
terminal to compile Fireants:

``` bash
make        # use "-j N" here for N parallel jobs
make check  # Run tests if Python 3 is available
```

### 3. Deploy (optional)

You can also create a  `.dmg` containing the `.app` bundle by running the 
following command:

``` bash
make deploy
```

## Running Fireants

Fireants should now be available at `./src/fireantsd`. 

If you compiled support for the GUI, it should be available at 
`./src/qt/fireants-qt`.

If you ran `make deploy` you will have a macOS app at `/dist/Fireants.app`
and a DMG at `./Fireants.dmg`.

The first time you run `fireantsd`, `fireants-qt`, or `Fireants.app` it will
start downloading the blockchain. This process could take many hours, or even 
days on slower than average systems.

By default, blockchain and wallet data files will be stored in:

``` bash
/Users/${USER}/Library/Application Support/Fireants/
```

Before running, it's recommended to create an RPC configuration file:

```shell
mkdir -p "/Users/${USER}/Library/Application Support/Fireants"

echo -e "rpcuser=fireantsrpc\nrpcpassword=$(xxd -l 16 -p /dev/urandom)" > "/Users/${USER}/Library/Application Support/Fireants/fireants.conf"

chmod 600 "/Users/${USER}/Library/Application Support/Fireants/fireants.conf"
```

You can monitor the download process by looking at the debug.log file:

```shell
tail -f $HOME/Library/Application\ Support/Fireants/debug.log
```

## Other commands:

```shell
./src/fireantsd -daemon      # Starts the fireants daemon.
./src/fireants-cli --help    # Outputs a list of command-line options.
./src/fireants-cli help      # Outputs a list of RPC commands when the daemon is running.
./src/qt/fireants-qt -server # Starts the fireants-qt server mode, allows fireants-cli control
```
