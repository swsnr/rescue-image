# Arch Rescue image

A mostly minimal and opinionated `mkosi` UKI image for use as rescue image for Arch Linux.

See https://swsnr.de/archlinux-rescue-image-with-mkosi

## Why?

Every once in a while Arch breaks.  They say every Arch Linux user carries a USB stick with the installation disk around, just in case.

With this image, you don't need to.  Put the image on your EFI, and boot it to chroot into any unbootable Arch Linux installation on the system.

## How?

Make sure you have recent versions of `mkosi` and `systemd-ukify` installed.

First, set a root password:

```console
$ printf "hashed:%s\n" "$(openssl passwd -6)" > mkosi.rootpw
$ chmod 600 mkosi.rootpw
```

Note: Without this step you **cannot** log in on the rescue image, so don't forget it.
Alternatively,

- write `hashed:` to `mkosi.rootpw` to make `root` have an empty password, or
- pass `--autologin` when building the image (see below) to have root automatically login on the first console (at your own risk!).

Then, build the image:

```console
$ mkosi --image-version="$(git rev-parse --short=10 HEAD)-(date --utc +%Y%m%d%H%M)"
```

You can pick whatever `image-version` you want as long as it helps you understand how and when you built the image.

Note: If you did not set up user namespace, you have to run the above command as root.

Then, put the image on your EFI partition (or on the XBOOTLDR partition if your EFI system partition is too small):

```console
# install -m644 -t /efi/EFI/Linux mkosi.output/*.efi
```

The image requires about 215 MiB of space.

### Secure boot

After installing the rescue image to `/efi` you can sign it for secure boot, e.g. with `sbctl sign`.

`mkosi` can also sign the image by itself, using `sbsigntools`.
For this you need to set `SecureBootKey=` and `SecureBootCertificate=`, e.g in `mkosi.local.conf`.

### Customization

You can put additional options for `mkosi` into `mkosi.local.conf` which is ignored by git.
You can also fork the repository and freely adapt the configuration to your own needs.

Refer to `mkosi(1)` for more information.

## License

This code in this repository is subject to the terms of the Mozilla Public
License, v. 2.0. If a copy of the MPL was not distributed with this
file, You can obtain one at http://mozilla.org/MPL/2.0/.

Packages inside the generated rescue image are covered by their respective licenses.
