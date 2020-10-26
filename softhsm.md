
# SoftHSM

*According of SoftHSM official [website](https://www.opendnssec.org/softhsm/):*

>SoftHSM is an implementation of a cryptographic store accessible through a PKCS #11 interface. You can use it to explore PKCS #11 without having a Hardware Security Module. It is being developed as a part of the OpenDNSSEC project. SoftHSM uses Botan for its cryptographic operations.

## Installation

SoftHSM can be installed using binarary package or from [source](https://github.com/opendnssec/SoftHSMv2). We use the following command to installed SoftHSM2 (version 2) in Ubuntu 18.04:
```bash
$ sudo apt update
$ sudo apt install softhsm
```

As part of the installation, the necessary command line tools (e.g. `softhsm2-util`) will be installed along with the shared object file `libsofthsm2.so` (possible location of this shared object is `/usr/lib/softhsm/`; keep this in mind, it will be needed later) which is the native implementation of PKCS#11 for SoftHSM.

## Configuration

Post-installation of `softhsm2`, it is important to configure the softhsm2 if you want to manage the token as non-root users. Otherwise, you might get the following error while trying to create a token using softhsm2-util or other utilities or scripts:

```bash
$ softhsm2-util --init-token --slot 0  --label "test" --so-pin 5462 --pin 8764329
ERROR: Could not initialize the library.
```

By default, the softhsm2 uses the following artifacts:
- `/etc/softhsm/softhsm2.conf`: softhsm2 configuration file
- `/var/lib/softhsm/tokens`: location to store the tokens

Use the manual page to find the system-wide `softhsm2.conf` location. A common location is `/etc/softhsm/softhsm2.conf`, but the location might be different on some systems.

```bash
$ man softhsm2.conf
```

Accesses to these files need root permission. As mention [here](https://stackoverflow.com/questions/53230852/error-creating-token-via-softhsm2-as-non-root-user-could-not-initialize-the-lib), changing ownership/permission of `/var/lib/softhsm/tokens` doesn't solve the problem as we cannot access `/etc/softhsm/softhsm2.conf` in the first place given the access limitation, so we should be doing this instead:

```bash
$ cd $HOME
$ mkdir -p $HOME/lib/softhsm/tokens
$ cd $HOME/lib/softhsm/
$ echo "directories.tokendir = $PWD/tokens" > softhsm2.conf

$ export SOFTHSM2_CONF=$HOME/lib/softhsm/softhsm2.conf
    NOTE: add this line in ~/.bashrc to avoid typing everytime.

$ softhsm2-util --init-token --slot 0 --label "test" --so-pin 1234 --pin 123456
The token has been initialized.
```

Now the token is initialized, and can be managmened and accessed using difirent PKCS11 clients, which we will discuss in the next sesction.

## Usage of softhsm2-util

This is a commandline uitility of softHSM for PKCS#11. The best way to explore this utility is with `-h` option. The source code of softhsm2-util can be found [here](https://github.com/opendnssec/SoftHSMv2/blob/develop/src/bin/util/softhsm2-util.cpp).

```bash
$softhsm2-util -h

Support tool for PKCS#11
Usage: softhsm2-util [ACTION] [OPTIONS]
Action:
  --delete-token    Delete the token at a given slot.
                    Use with --token or --serial.
                    WARNING: Any content in token will be erased.
  -h                Shows this help screen.
  --help            Shows this help screen.
  --import <path>   Import a key pair from the given path.
                    The file must be in PKCS#8-format.
                    Use with --slot or --token or --serial, --file-pin,
                    --label, --id, --no-public-key, and --pin.
  --init-token      Initialize the token at a given slot.
                    Use with --slot or --token or --serial or --free,
                    --label, --so-pin, and --pin.
                    WARNING: Any content in token will be erased.
  --show-slots      Display all the available slots.
  -v                Show version info.
  --version         Show version info.
Options:
  --file-pin <PIN>  Supply a PIN if the file is encrypted.
  --force           Used to override a warning.
  --free            Use the first free/uninitialized token.
  --id <hex>        Defines the ID of the object. Hexadecimal characters.
                    Use with --force if multiple key pairs may share
                    the same ID.
  --label <text>    Defines the label of the object or the token.
  --module <path>   Use another PKCS#11 library than SoftHSM.
  --no-public-key   Do not import the public key.
  --pin <PIN>       The PIN for the normal user.
  --serial <number> Will use the token with a matching serial number.
  --slot <number>   The slot where the token is located.
  --so-pin <PIN>    The PIN for the Security Officer (SO).
  --token <label>   Will use the token with a matching token label.
```

### Create a token

To create a token, we need to use `--init-token` action with associated options. We need need to provide security Officier PIN (option `--so-pin`) to add or remove token, and User PIN (option `--pin`) for accessing token from applications. 

#### Case 1: With commandline options for PIN

```bash
$ softhsm2-util --init-token --slot 0  --label "<token name>" --so-pin <so-pin value> --pin <pin value>
```

PIN can be sequence of UTF symbols. The problem with this approach is that one can see the PINs from command history! It should be avoided.

#### Case 2: Without commandline Options for PIN

```bash
$ softhsm2-util --init-token --slot 0 --label "<token name>"
```

This prompts you for providing PINs.

### Get slot details

```bash
$ softhsm2-util --show-slots
```

### Delete a token

#### Using token label

```bash
$ softhsm2-util --delete-token --label "<token name>"
```

#### Using token serial no.

```bash
$ softhsm2-util --delete-token --serial "<token serial no>"
```

Note that serial number of a token can be obtained using `softhsm2-util --show-slots` command.

# Useful Resource

- A dive into SoftHSM. [Link](https://medium.com/@clydecroix/a-dive-into-softhsm-e4be3e70c7bc)
- Install and Configure SoftHSM with Ubuntu. [Link](https://medium.com/faun/install-and-configure-softhsm-with-ubuntu-2375f32bb20a)
- SoftHSM2 on Ubuntu 18.04. [Link](https://drtfm.com/softhsm2-on-ubuntu-18-04/)
- SoftHSM Documentation v2 [Link](https://wiki.opendnssec.org/display/SoftHSMDOCS/SoftHSM+Documentation+v2)
- Module 7: Simulating hardware security integration. [Links](https://docs.aws.amazon.com/greengrass/latest/developerguide/console-mod7.html)
- An introduction to the use of HSM. NLnet Labs document 2008-draft. [source](https://www.dnssec.cz/files/nic/doc/hsm.pdf)
