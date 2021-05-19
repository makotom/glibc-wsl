# glibc-wsl

[The GNU C Library](https://www.gnu.org/software/libc/) for [Arch Linux](https://archlinux.org/) running on [Windows Subsystem for Linux 1](https://docs.microsoft.com/en-us/windows/wsl/).

## Why

- WSL 1 has [a known issue](https://github.com/microsoft/WSL/issues/3023#issuecomment-452339739) that its vDSO reports very old kernel version.
- In February 2021, Arch Linux [started](<(https://github.com/archlinux/svntogit-packages/commit/893b1c268abc8822332655865e3d4546025a9b4b#diff-37538beb61ff63edebbf735dfcf39e5d732f49183d6beb097169d971875ca422R54)>) to distribute a new build of GNU C Library, i.e., `glibc-2.33-4`, and stopped supporting Linux kernels < 4.4.
- As a result, running `pacman -Syu` on WSL 1 will break your Arch Linux setup, as `glibc` is no longer compatible with WSL 1.
- Fix for the issue is relatively easy, but actually there are other minor issues in packaging of `glibc`.
- Then, this repository intends to provide patches (and resulting binaries) which make the package:
  1. WSL 1 compatible,
  2. buildable with simple `makepkg`, with out-of-box `makepkg.conf`, and
  3. GPL-safer with plain-text-formatted licence terms included in the distributed tarballs.

### Why not AUR?

Because the author does not want to build it locally, as it would take some 30 - 60 minutes and eventually fail.
Especially, it is totally nonsense to quasi-automatically trigger a build of `glibc` on _each_ `yay` run.

### Why not an independent package in an independent repository?

**First of all, this distribution does not intend to permanently replace the official `glibc` package.**
These are just patches, and it is totally up to users and _at their own risk_ to apply them or take advantage of "stray" builds with the patches applied.

Then, since these are just patches, there is a motivation to minify patches and minimize differences in version numbers.
This is why the patches does not rename the package, and packages are provided as standalone ones.

### Why not building periodically?

**First of all, this distribution does not intend to permanently replace the official `glibc` package.**
These are just patches, and it is totally up to users and _at their own risk_ to apply them or take advantage of "stray" builds with the patches applied.

With that being said, the build job _can be_ a scheduled job, but it is ultimately meaningless because:

1. most runs would be rebuilds or redundant polling runs, and unneeded builds are costly.
2. any run which contains changes in the upstream is likely to fail and need human interventions, deteriorating benefits of periodical builds.

In other words, the author will try to manually track the changes in the upstream, as much as possible, and as fast as possible.
In fact that is somewhat feasible, since the author is likely to take a look on any change in the upstream and implement countermeasures, regardless whether there are any automated processes, to salvage his own WSL environments.

## How to?

You should be able to download the `.pkg.tar.zst` file for [the latest release](https://github.com/makotom/glibc-wsl/releases/latest), and install it with `pacman -U` command.
By downloading the corresponding `.sig` file as well, you should be able to verify your copy; follow [this instruction](https://wiki.archlinux.org/title/Pacman/Package_signing#Adding_unofficial_keys) to populate the GPG public key `2E6F2933950726A85333D84EE28A22F9AF2FEC65` used to sign the package.

Note that installing the package directly with `pacman -U << URL >>` would not work, as GitHub Releases redirects accesses to released assets towards another very-long URL, and it causes an error upon file downloads by `pacman`.

In order to avoid unexpected installation of future updates to the official `glibc`, add `glibc` to `IgnorePkg` of `/etc/pacman.conf`.
Note that official future updates in `glibc` can overwrite the patched package, and consequently break your Arch Linux setup.

## DIY?

By forking this repo and setting up your own CircleCI project, you are able build the package by yourself.

For your own CircleCI project, you need to setup 2 [contexts](https://circleci.com/docs/2.0/contexts/):

- Context `gpg-glibc-wsl`. This needs to contain the following env vars:
  1. `PACKAGER` ([your display name as a packager](https://wiki.archlinux.org/title/Makepkg#Packager_information), e.g., `Makoto Mizukami <comm@makotom.org>`),
  2. `GPG_SECRET_KEY` (the Base64-encoded GPG key to sign packages), and
  3. `GPG_PASSPHRASE` (the passphrase for the GPG key).
- Context `github`. This needs to contain [a GitHub personal access token](https://help.github.com/en/github/authenticating-to-github/creating-a-personal-access-token-for-the-command-line), which has a sufficient permission to create a new release, in `GITHUB_TOKEN` env var.
