libsecp256k1 <a target="_blank" href="https://gitter.im/bitprim/Lobby">![Gitter Chat][badge.Gitter]</a>
============

Optimized C library for EC operations on curve secp256k1.
This library is a work in progress and is being used to research best practices. Use at your own risk.

| **master(linux/osx)** | **dev(linux/osx)**   | **master(windows)**   | **dev(windows)** |
|:------:|:-:|:-:|:-:|
| [![Build Status](https://travis-ci.org/bitprim/secp256k1.svg)](https://travis-ci.org/bitprim/secp256k1)       | [![Build StatusB](https://travis-ci.org/bitprim/secp256k1.svg?branch=dev)](https://travis-ci.org/bitprim/secp256k1?branch=dev)  | [![Appveyor Status](https://ci.appveyor.com/api/projects/status/github/bitprim/secp256k1?svg=true)](https://ci.appveyor.com/project/bitprim/secp256k1)  | [![Appveyor StatusB](https://ci.appveyor.com/api/projects/status/github/bitprim/secp256k1?branch=dev&svg=true)](https://ci.appveyor.com/project/bitprim/secp256k1?branch=dev)  |

Table of Contents
=================

   * [libsecp256k1](#libsecp256k1)
      * [Features:](#features)
      * [Implementation details](#implementation-details)
      * [Installation](#installation)
        * [Using Conan](#using-conan)
        * [Manual build steps](#manual-build-steps)
            * [For Linux/Windows with MinGw/OSX](#for-linux-windows-with-mingw-and-osx)
            * [For Windows with Visual Studio](#for-windows-with-visual-studio)

## Features

* secp256k1 ECDSA signing/verification and key generation.
* Adding/multiplying private/public keys.
* Serialization/parsing of private keys, public keys, and signatures.
* Constant time, constant memory access signing and pubkey generation.
* Derandomized DSA (via RFC6979 or with a caller provided function).
* Very efficient implementation.

## Implementation details

* General
  * No runtime heap allocation.
  * Extensive testing infrastructure.
  * Structured to facilitate review and analysis.
  * Intended to be portable to any system with a C89 compiler and uint64_t support.
  * Expose only higher level interfaces to minimize the API surface and improve application security. ("Be difficult to use insecurely.")
* Field operations
  * Optimized implementation of arithmetic modulo the curve's field size (2^256 - 0x1000003D1).
    * Using 5 52-bit limbs (including hand-optimized assembly for x86_64, by Diederik Huys).
    * Using 10 26-bit limbs.
  * Field inverses and square roots using a sliding window over blocks of 1s (by Peter Dettman).
* Scalar operations
  * Optimized implementation without data-dependent branches of arithmetic modulo the curve's order.
    * Using 4 64-bit limbs (relying on __int128 support in the compiler).
    * Using 8 32-bit limbs.
* Group operations
  * Point addition formula specifically simplified for the curve equation (y^2 = x^3 + 7).
  * Use addition between points in Jacobian and affine coordinates where possible.
  * Use a unified addition/doubling formula where necessary to avoid data-dependent branches.
  * Point/x comparison without a field inversion by comparison in the Jacobian coordinate space.
* Point multiplication for verification (a*P + b*G).
  * Use wNAF notation for point multiplicands.
  * Use a much larger window for multiples of G, using precomputed multiples.
  * Use Shamir's trick to do the multiplication with the public key and the generator simultaneously.
  * Optionally (off by default) use secp256k1's efficiently-computable endomorphism to split the P multiplicand into 2 half-sized ones.
* Point multiplication for signing
  * Use a precomputed table of multiples of powers of 16 multiplied with the generator, so general multiplication becomes a series of additions.
  * Access the table with branch-free conditional moves so memory access is uniform.
  * No data-dependent branches
  * The precomputed tables add and eventually subtract points for which no known scalar (private key) is known, preventing even an attacker with control over the private key used to control the data internally.
  
## Installation

### Using Conan

Conan is a Python package for dependency management; it only requires Python and Pip.
With Conan, install can be performed on any OS. If there are no prebuilt binaries for a given
OS-compiler-arch combination, Conan will build from source.

```
pip install conan
conan remote add bitprim https://api.bintray.com/conan/bitprim/bitprim
conan install secp256k1/0.1@bitprim/stable
```

The last step will install binaries and headers in Conan's cache, a directory outside the usual
system paths. This will avoid conflict with system packages such as boost.
Also, notice it references the stable version 0.1. To see which versions are available,
please check [Bintray](https://bintray.com/bitprim/bitprim/secp256k1%3Abitprim).

### Manual build steps

#### For Linux, Windows with MinGw, and OSX

```
$ git clone https://github.com/bitprim/secp256k1.git
$ cd secp256k1
$ mkdir build
$ cd build
$ cmake .. -DENABLE_MODULE_RECOVERY=ON 
$ make -j2
$ sudo make install
```

#### For Windows with Visual Studio

From a [Visual Studio Developer Command Prompt](https://docs.microsoft.com/en-us/dotnet/framework/tools/developer-command-prompt-for-vs):

```
$ git clone https://github.com/bitprim/secp256k1.git
$ cd secp256k1
$ mkdir build
$ cd build
$ cmake .. -DENABLE_MODULE_RECOVERY=ON -DUSE_CONAN=OFF -DNO_CONAN_AT_ALL=ON
$ msbuild ALL_BUILD.vcxproj
```

[badge.Gitter]: https://img.shields.io/badge/gitter-join%20chat-blue.svg
