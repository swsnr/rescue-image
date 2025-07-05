# Rescue USI

Build a minimal and opinionated rescue USI (unified kernel image with embedded root filesystem),
i.e. a single EFI executable containing a minimal yet complete system for recovery.

## Why?

You tinker, things break. They say every Arch Linux user carries a USB stick with the installation disk around, just in case.

With this image, you don't need to. Put the image on your ESP, and boot it rescue your Linux installation.

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
$ mkosi build
```

Note: If you did not set up user namespaces, you have to run the above command as root.

Then, put the image on your EFI partition (or on the XBOOTLDR partition if your EFI system partition is too small):

```console
# install -m644 -t /efi/EFI/Linux mkosi.output/*.efi
```

If you place it in `EFI/Linux` systemd-boot will discover it automatically without further configuration.

### Image version

By default, mkosi reads the version of this image from the `mkosi.version` file; together with the distribution release
version this helps you identify what rescue image you have.

However, for a rolling release distribution which has no distribution release version, i.e. Archlinux specifically,
you can chose an explicit image version to help identify the image contents:

```console
$ mkosi --image-version="$(git rev-parse --short=10 HEAD)-$(date --utc +%Y%m%d%H%M)" build
```

### Secure boot

After installing the rescue image to `/efi` you can sign it for secure boot, e.g. with `sbctl sign`.

`mkosi` can also sign the image by itself, using `sbsigntools`.
For this you need to set `SecureBootKey=` and `SecureBootCertificate=`, e.g in `mkosi.local.conf`.

### Customization

By default, the image builds for the same distribution as the host, i.e. if you're running an Arch system it build an Arch image.
You can customize this by setting the `Distribution` key in `mkosi.local.conf`.

You can put additional options for `mkosi` into `mkosi.local.conf` which is ignored by git.
You can also fork the repository and freely adapt the configuration to your own needs.

Refer to `mkosi(1)` for more information.

## License

Copyright Sebastian Wiesner <sebastian@swsnr.de>

The code in this repository is licensed under the EUPL, see <https://interoperable-europe.ec.europa.eu/collection/eupl/eupl-text-eupl-12>.

Packages inside the generated rescue image are covered by their respective licenses;
as a result the final rescue image may be covered by a different license.
