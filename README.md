# AlpineMobile - Alpine Linux On Mobile

> [!WARNING]
> This project only supports `arm64`. It will terminate immediately if run on a different architecture.

> [!WARNING]
> This project is intended for mobile devices running Linux kernel `4.4` or later (released January 2016). Older kernels are likely to cause major issues and are unsupported.

AlpineMobile provides a lightweight and minimal Alpine Linux chroot environment for mobile devices. It manages an isolated, native Alpine, Linux-like userland without depending on external terminal apps or additional user-space layers. It's ideal for power users, developers, and tinkerers who want to run a sandboxed Alpine environment as close to the actual mobile system.

![Alpine running on mobile](alpine.jpg)

## AlpineMobile is not standard terminal apps

Unlike standard terminal apps which run Linux tools via mobile-compiled binaries or user-space emulation, AlpineMobile runs a real Alpine chroot directly on mobile. It ships its own programs, libraries, patches and links key mobile filesystems into the Alpine environment, providing near-native Linux functionality and the ability to run almost any Alpine-compatible program.

## Features

- Extremely lightweight and portable
- Zero external dependencies
- Runs a minimal Alpine Linux userland in isolated chroot environment
- Mounts key mobile system paths to provide near-native Linux functionality
- Uses `unshare` to isolate mobile mountpoints from the chroot environment
- Supports `/dev` overlayfs, creating a writable layer over device files without modifying the real `/dev`

## Requirements

- A rooted mobile device
- A minimum 500MB-2GB of available storage space

## Disclaimer

The software is provided "as is", without warranty of any kind, express or implied, including but not limited to the warranties of merchantability, fitness for a particular purpose, and noninfringement. In no event shall the authors or copyright holders be liable for any claim, damages, or other liability, whether in an action of contract, tort, or otherwise, arising from, out of, or in connection with the software or the use or other dealings in the software.

## Security considerations

### Random Number Generation

This project patches certain system utilities, providing compatibility with the mobile system by overriding Linux's randomness mechanisms, such as the `getrandom` syscall, `getentropy` function and musl libc's random functions. It replaces the default cryptographic randomness with direct `/dev/urandom` reads. This is generally safe on modern Linux/mobile systems, because the kernel ensures `/dev/urandom` provides high-quality entropy.

## Installation Guide

This guide is written for novice users. You only need basic knowledge of mobile systems, like installing apps from alternative sources, extracting an archive, and a little experience using a terminal (command line).

### What You Need

Before starting, make sure you have these apps installed on your device:

- [Mobile Terminal](apk/MobileTerminal.apk) - lets you access mobile's command-line interface
- [Technical Keyboard](apk/TechnicalKeyboard.apk) - an advanced keyboard that makes typing commands easier
- [File Manager Pro](https://play.google.com/store/apps/details?id=com.estrongs.android.pro) - lets you browse and manage mobile files

Alternatively, you can use a free alternative to File Manager Pro, such as [Solid Explorer](https://play.google.com/store/apps/details?id=pl.solidexplorer2) or [X-plore](https://play.google.com/store/apps/details?id=com.lonelycatgames.Xplore).

### Step 1: Downloading and Extracting the Project

1. Download the project zip file: [AlpineMobile-main.zip](https://github.com/MOBILELINUX/AlpineMobile/archive/refs/heads/main.zip)
2. Locate `AlpineMobile-main.zip` in your `Download` folder (`/sdcard/Download`)
3. Extract the `AlpineMobile-main.zip` archive inside the `Download` folder. You can use an application such as `File Manager Pro` or `Solid Explorer` for this task

After extraction, you should have a folder named `AlpineMobile-main` in your `Download` folder.

### Step 2: Installing AlpineMobile

1. Open the `Mobile Terminal` app
2. Type the following commands, pressing Enter after each one:

```bash
su
cd /sdcard/Download/AlpineMobile-main
sh setup.sh
```

The script will automatically install AlpineMobile files in:

- `/data/local/alpinemobile`
- `/sdcard/alpinemobile`

After that, AlpineMobile is ready to use!

## Quickstart Guide

### Usage

To display AlpineMobile's help page, run:

```bash
/sdcard/alpinemobile/alpinemobile.sh help
```

```txt
Usage:
  alpinemobile.sh [OPTION] [SUBOPTION] [ARGUMENTS]

Options:
  run [COMMAND...]
      Runs the default Alpinemobile environment.
      If COMMAND is provided, it executes that command inside the environment.
      If no command is given, an interactive shell is started.

  list [patch|command]
      Lists available scripts.
      patch   - Lists all patch scripts in the patch directory.
      command - Lists all command scripts in the command directory.

  patch [PATCH_NAME]
      Applies the specified patch script from the patch directory.
      Example: alpinemobile.sh patch fix_network

  command [COMMAND_NAME]
      Executes the specified command script from the command directory.
      Example: alpinemobile.sh command setup_user

  resize (+|-)[SIZE]
      Resizes the alpine image relative to the specified size.
      Example: alpinemobile.sh resize +1G

Notes:
  - Unrecognized options are treated the same as the 'run' option.
```

### AlpineMobile Hierarchy

```txt
/sdcard/alpinemobile/
├── alpinemobile_env.sh   # Config script (paths, environment variables)
├── alpinemobile.sh       # Control script
├── alpinemobile_mgr.sh   # Backend script (mounts and manages chroot)
├── img/
│   └── alpine.img        # Alpine root filesystem
├── patch/                # Patch scripts
└── command/              # Command scripts
```

Scripts placed in `patch` provide easy-to-apply fixes and tweaks for the environment's misbehaving. For more information, check out the [patching section](#patching). The ones placed in the `command` folder can be easily executed to conduct specific tasks within the chroot environment, such as updating the system or managing ssh & vnc servers (coming soon!). Consult the [running commands section](#running-commands).

```txt
/data/local/alpinemobile/
├── bin   # External binaries
├── lib   # External libraries
└── mnt   # Alpine mount point
```

Programs stored in `bin` can be run directly inside the chroot as it is mounted and appended to `$PATH`. Libraries placed inside `lib` will be automatically preloaded, allowing for custom overrides and patches without any modifications to Linux system files.

### Running Interactive Sessions

1. Open the `Mobile Terminal` app
2. Type the following commands, pressing Enter after each one:

```bash
su
sh /sdcard/alpinemobile/alpinemobile.sh
```

This launches the environment and gives you a shell inside Alpine.

Inside the chroot shell, users can execute the `exit` command to leave the environment and automatically unmount the filesystems.

### Running Commands

You can run a specific command inside the chroot without starting an interactive shell. AlpineMobile provides pre-made command scripts located in `/sdcard/alpinemobile/command`, which automate common tasks such as maintenance, setup, or service management.

To list available command scripts, run:

```bash
su
sh /sdcard/alpinemobile/alpinemobile.sh list command
```

To execute a command script, run:

```bash
su
sh /sdcard/alpinemobile/alpinemobile.sh command <command-name>
```

To execute other commands directly inside the chroot, run:

```bash
su
sh /sdcard/alpinemobile/alpinemobile.sh apk update
# or (explicit version)
sh /sdcard/alpinemobile/alpinemobile.sh run apk update
```

This will execute the `apk update` command directly in the chroot environment.

## Patching

If certain utilities or packages don't work correctly inside the chroot, you can apply the provided patch scripts located in /sdcard/alpinemobile/patch. These scripts fix common issues, such as networking problems or other environment-specific quirks.

To list all available patches, run:

```bash
su
sh /sdcard/alpinemobile/alpinemobile.sh list patch
```

To apply a patch, run:

```bash
su
sh /sdcard/alpinemobile/alpinemobile.sh patch <patch-name>
```

## Notes

### Image Resizing

The Alpine root filesystem (alpine.img) in AlpineMobile has a fixed size. If you need more space, you can easily expand it using the built-in resize helper.

The following command expands the alpine environment by 500MB:

```bash
su
sh /sdcard/alpinemobile/alpinemobile.sh resize +500M
```

For additional usage instructions, run:

```bash
su
sh /sdcard/alpinemobile/alpinemobile.sh resize
```

- Make sure the image is **not mounted** when resizing
- Ensure you have **enough free storage** on your device to accommodate the new image size

### Patching Apk

The `_apk` user is responsible for managing package downloads and upgrades inside the chroot. By default, it does not have network access due to mobile group restrictions. This can cause errors during `apk update` or `apk upgrade`. Fix the issue by applying the `apk.sh` patch:

```bash
su
sh /sdcard/alpinemobile/alpinemobile.sh patch apk
```

After running this, the `_apk` user will have the necessary permissions to perform system upgrades without errors.

### User Networking

Due to mobile group restrictions, users inside the AlpineMobile chroot need to be added to the `inet` group to access networking tools.

```bash
addgroup -g 3003 inet
adduser <username> inet
```

After this, the user should be able to use networking commands like `ping`.

## Patched Programs

The following programs have been analyzed and patched to run properly within the AlpineMobile chroot environment:

- `gpg` – GNU Privacy Guard
- `sshd` – OpenSSH server
- `xfce4` - XFCE4 desktop environment (coming soon!)