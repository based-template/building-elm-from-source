# Building from source

The following has been tested for the 0.19.1 release on Ubuntu 21.04, based on the Dockerfile under installers/linux. Check the scripts under installers/mac, installers/win, and installers/npm when building for those platforms.

## Prerequisites

You'll need ghc and cabal, which are the Haskell compiler and build system, respectively. You'll also need some some other libraries/tools (namely git, C library headers, libncurses headers, ncurses). Installing those looks something like:

```shell
sudo apt install "build-essential"
```

```shell
sudo apt install libncurses-dev wget
```

if you're on Ubuntu. Or 

```shell
sudo yum groupinstall "Development Tools"
```

```shell
sudo yum install ncurses-devel wget
```

These should cover your bases

It would be ideal to gather more information about the environments/toolchains used by the other developers on the project, as newer versions of certain tools like the GHC and Cabal definitely don't work for 0.19.1, and didn't seem to work for the upstream either. At the same time, it seems that the 0.19.1 release will work with multiple versions of GHC (8.10.6 and 8.4.3), if not multiple versions of Cabal (2.4.1 the only one confirmed to work).

## Getting the source package

```
wget https://github.com/elm/compiler/archive/refs/tags/0.19.1.tar.gz
```

Will get you the source package for the 0.19.1 release of Elm, which as of this writing (26 AUG 2021) is the latest to be released. 

## Installing the Haskell toolchain

The Elm compiler codebase is comprised mostly of Haskell code (95% according to Github). **installing the Haskell toolchain is probably the single most important step described in this entire document**

We're going to be installing ghcup [haskell.org](https://www.haskell.org/ghcup), an installer for Haskell.


Following the instructions in the link above, run

```
curl --proto '=https' --tlsv1.2 -sSf https://get-ghcup.haskell.org | sh
```

which of course requires the tool 'curl', and involves executing a random script from the internet on your machine.

*If you prefer to read over scripts before executing them on your system:*

```
curl --proto '=https' --tlsv1.2 -sSf https://get-ghcup.haskell.org >> get_ghcup.sh
```

Which saves the script to a file, which you may read over before executing via: 

```
sh get_ghcup.sh
```

*According the the website, the above should work for all Linux, macOS, FreeBSD or Windows Subsystem 2 for Linux. Instructions for installing on Windows via powershell are also listed there*

### For the extra-conscientious:

Manual installation instructions for ghcup are provided at [gitlab.haskell.org](https://gitlab.haskell.org/haskell/ghcup-hs#manual-install)

## Configuring the Haskell toolchain:

Using ghcup, we will set the following tools to versions we know will/should work:

Ghc -> 8.4.3
Cabal -> 2.4.1

*How was this deduced? See the note at the top of this file. The Dockerfile for 0.19.1 specifies Alpine 3.10 as the base image, then installing ghc and cabal. Searching [pkgs.alpinelinux.org](https://pkgs.alpinelinux.org/packages?name=&branch=v3.10) for ghc and cabal gives us these version numbers*

ghcup provides a wonderful little terminal UI via 

```
ghcup tui
```

which will allow you to do everything needed as described below (just to remember to toggle on all versions).

Running 

```
ghcup list -t ghc
```

and 

```
ghcup list -t cabal
```

will show you all of the versions of ghc and cabal available, installed, and 'set' on your system. As of GHCup 0.16.2, a version of a toolchain which hasn't been installed is indicated with an ❌, installed packages are indicated with a single ✔ , and the currently 'set' version of a package is indicated via double ✔✔ .

Installing the correct packages involves:
```
ghcup install ghc 8.4.3
```

```
ghcup install cabal 2.4.1
```

And now 'setting' the tools to the correct version:

```
ghcup set ghc 8.4.3
```

```
ghcup set cabal 2.4.1
```

Running the list commands as above should show two checkmarks next to each of those tools with the correct version. Even if the output looks correct, make sure to perform the following check:

#### Important: Confirming the correct version of the tools has been set:

Running 

```
ghc --version
```

should report that the Glasgow Haskell Compilation System is on version 8.4.3, and

```
cabal --version 
```

reports 2.4.1.0 

If either tool produces the incorrect version number, run 

```
which <cabal|ghc>
```

Which will tell you the version your shell knows about. On my system, there was a 'local' cabal that resided in \${HOME}/.cabal/bin. Annoyingly, the ghcup environment file \${HOME}/.ghcup/env was exporting \${HOME}/.cabal/bin into my \$PATH, fixing this involved editing the env file to only add \${HOME}/.ghcup/bin, which contains all of the different toolchain versions that ghcup knows about/manages. YMMV

## Actually compiling the source code:

Install all pre-requisites, as described above, with special attention given to the particulars of the Haskell toolchain.

From whatever directory, your Elm tarball is located (See: "Getting the source package", above), run:

```
tar -xvzf ./0.19.1.tar.gz
```

```
cd compiler-0.19.1/
```

```
rm worker/elm.cabal # As suggested by the Dockerfile, appears to prevent Cabal from using this instead of the elm.cabal located in the top level of the project
```

```
cabal new-update
```

```
cabal new-configure --ghc-option=-optl=-pthread # Ignoring the other flags used for the Alpine Docker
```

```
cabal new-build
```

With any luck, you should have a new 'elm' compiler executable located in compiler-0.19.1/dist-newstyle/build/x86_64-linux/ghc-8.4.3/elm-0.19.1/x/build/elm/elm

## Now what?

The executable can be referenced with the complete path (eg: ${HOME}/compiler-0.19.1/dist-newstyle/build/x86_64-linux[...]/elm), but that is cumbersome, and not the typical way to execute programs. 

If we want to be able to omit the full filename, we need to add Elm's *path* to our $PATH environment variable. We can leave it in it's current location, although that might be tedious to manage should we wish to install a new version in the future, or be able to remove the source package from our system while keeping the executable.

On the other hand, it may not be wise to follow the [official install instructions](http://github.elm/compiler/blob/master/installers/linux/README.md), which suggest grabbing a random binary directly off the internet, making it executable, and moving it to /usr/local/bin with root permission. 

We can place the executable anywhere, but following standard practice these days would involve making a folder \${HOME}/.elm/bin, and placing it there, then adding \${HOME}/.elm/bin to our .bashrc:

```
mkdir -p ${HOME}/.elm/bin
```

```
cp ./dist-newstyle/build/x86_64-linux/ghc-8.4.3/elm-0.19.1/x/elm/build/elm/elm ${HOME}/.elm/bin/
```

```
chmod +x ${HOME}/.elm/bin/elm
```
### Editing .bashrc (or other environment file):

Add the following:

```shell
if ! $(echo "$PATH" | tr ":" "\n" | grep -qx "${HOME}\/.elm\/bin"); then
    PATH:${HOME}/.elm/bin";
fi
```

And source your .bashrc (or other environment file).

