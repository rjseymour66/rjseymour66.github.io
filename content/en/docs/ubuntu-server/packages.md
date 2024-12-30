---
title: "Software Packages"
linkTitle: "Packages"
weight: 20
# description:
---

Linux uses software repositories to distribute software:
- Available as mirrors, which are copies of the repos in different geographical locations so you can download software from whichever is closest
- Some packages have dependencies, and Linux provides software to help you manage dependencies
- There is a maintainer or group of maintainers that packages and ships new versions to repositories for approval, which are eventually distributed to mirrors
- New package versions are usually because of security updates or new features
  - Updates are usually security patches rather than new features
  - If there are major feature changes, then the new version is held until the upcoming major release
- New features are called **Stable Release Updates** and go through rigorous approval before they are available in default repositories

## Debian packages

Ubuntu uses Debian packages (`.deb`) because Ubuntu is forked from Debian and uses the same installation commands
- Kernel, system pacakges, libraries, and security updates are all Debian packages
- Other apps - such as Apache - use .deb files too, and their dependency versions might conflict with those used by the system
  - Maintainers go through great pains to ensure there are no conflicts
- New major versions are usually not available until the next distro version release
- **Universal packages** are a new and intended to be a single package that multiple distro versions can recognize
  - Dev creates a single package, instead of one package per distro, and users download that one package, irregardless of their distro

## Snap packages

Snap packages is a **universal package** type, which is a single package format that any distro can install
- Universal packages have their dependencies built in, so there shouldn't be conflicts
- Example universal packages include Flatpak, AppImage, and Snap
  - Flatpak and AppImage only work for desktop GUI systems
  - Snap works for both GUI and server installations
- Snap packages are called _snaps_ for short
- Snaps have no dependencies on your system packages
- Snaps are larger than regular Debian packages because they include all dependencies
- Snap packages are just now catching on, we usually still install Debian packages