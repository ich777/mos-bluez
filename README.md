# MOS BlueZ

mos-bluez provides a **MOS plugin** that integrates dbus and BlueZ
into the MOS ecosystem.

---

## Overview

This repository contains the **MOS plugin implementation**, optional helper
functions, and the web UI.

The plugin installs the official Debian `dbus` and `bluez` packages and allows
MOS to start and stop both services and monitor their status.

### Package Source

- dbus: [https://www.freedesktop.org/wiki/Software/dbus/](https://www.freedesktop.org/wiki/Software/dbus/)
- BlueZ: [http://www.bluez.org/](http://www.bluez.org/)

---

## Build & Automation

This repository includes a **GitHub Actions workflow** used to build and package
the plugin and its associated components for MOS.

The build process is fully automated and produces artifacts that can be
installed through the MOS Hub.

---

## Licensing

The contents of this repository (plugin code, build scripts, configuration,
and automation) are licensed under **GPL-3.0**.

`dbus` and `bluez` themselves are licensed under their respective upstream
licenses.

---

## Third-Party Software

This repository builds and packages third-party open-source software.
Packaged components remain licensed under their original upstream licenses.

Refer to `THIRD_PARTY.md` for details.
