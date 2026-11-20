# slackaudit 0.2.5

slackaudit is a Bash tool for local Slackware inventory and auditing. Its interactive interface uses dialog. It only collects and presents information; it does not change system settings or install or remove packages.

## Areas covered

The interactive interface is organized by audit area. Each area opens a submenu with separate screens for its topics, so detailed output is easier to inspect. The main menu includes a package-requirements check before the audit. A full text report remains available from the main menu and command line.

- **Audit summary:** local checks for duplicate package names, fstab structure, root filesystem usage, and kernel messages.
- **System:** Slackware release, architecture, kernel, uptime, and basic resource use.
- **Hardware:** separate views for CPU, memory and memory modules, motherboard and firmware, loaded kernel modules, PCI, USB, graphics, audio, input devices, sensors, and network adapters.
- **Packages:** package count and duplicate versions, a likely third-party package list, a complete installed package list, and a separate scan for manifest paths that are missing from the filesystem.
- **Storage:** separate views for disks and partitions, mounted filesystems, /etc/fstab validation, capacity and inode use, and per-disk SMART health checks.
- **Boot:** separate views for the running kernel and parameters, /boot contents, the detected boot manager configuration, and UEFI entries when available.
- **Network:** separate views for interfaces and addresses, routes, DNS, wireless and NetworkManager status, and listening sockets.
- **Services:** separate views for Slackware startup scripts, running processes, and listening sockets.
- **Local configuration:** separate views for accounts and groups, sensitive-file permissions, SSH settings, and firewall rules.
- **Logs:** kernel and system messages grouped by error, warning, success/OK, and informational patterns.

CVE analysis is not included.

## Packages and tools

For the **interactive interface**, install `dialog`. The command-line report (`--report`) does not need it. The script itself requires Bash and standard Slackware tools. On a minimal installation, make sure these packages are present:

- `bash`, `coreutils`, `hostname`, `gawk`, `grep`, and `sed` for the script and text processing.
- `procps-ng` for memory, process, and uptime information.
- `util-linux` for `lscpu`, `lsblk`, `findmnt`, and `dmesg`.
- `iproute2` for interface addresses, routes, and listening sockets.
- `dialog` for the interactive menus.

Install the following **optional packages** to enable the corresponding detailed screens. Some basic fallbacks use `/sys` or `/proc`, but installing these tools provides fuller, more readable output:

- `pciutils` (PCI devices and drivers), `usbutils` (USB device names and bus tree).
- `dmidecode` (memory modules, motherboard, BIOS/SMBIOS; run as root).
- `kmod` (formatted `lsmod` list; `/proc/modules` is used as a fallback).
- `smartmontools` (SMART health information; run as root for device queries).
- `lm_sensors` (temperature and sensor readings). Sensor results also depend on supported hardware, kernel drivers, and sensor configuration.
- `alsa-utils` (audio device enumeration).
- `libinput` (detailed input-device enumeration).
- `NetworkManager` (NetworkManager status), `iw` (wireless-device details).
- `iptables` or `nftables` (firewall rules; run as root for complete results).
- `efibootmgr` (UEFI boot entries, only for systems booted in UEFI mode; run as root).

The **Required tools and packages** screen in the main menu checks commands and reports the associated Slackware package names to install when a command is missing. For the SSH server configuration screen, install `openssh`. A normal full installation may already include some or most of these packages.

## Running

For the most complete results, run as root. Some details require elevated access even when all suggested packages are installed. If sudo is installed:

~~~sh
sudo slackaudit
~~~

Otherwise, start a root shell with su and run slackaudit from it. Non-root operation is supported, but some hardware, UEFI, firewall, and kernel-log details may be unavailable. The interactive interface asks whether to continue with limited access.

Make the script executable and start it:

~~~sh
chmod +x slackaudit
./slackaudit
~~~

To write a complete text report without opening the interface:

~~~sh
sudo ./slackaudit --report slackaudit-report.txt
~~~

To overwrite an existing report:

~~~sh
sudo ./slackaudit --report slackaudit-report.txt --force
~~~

The report is created with permissions restricted to its owner. It may contain hostnames, IP addresses, account names, and log excerpts. Review it before sharing.

## Optional installation

Copy the script to a directory in PATH:

~~~sh
install -m 0755 slackaudit /usr/local/sbin/slackaudit
~~~

Then run it:

~~~sh
sudo slackaudit
~~~

## Check limitations

The package check compares paths in /var/log/packages with files on disk. It detects missing paths but does not compare hashes or detect changed file contents. The third-party package view recognizes common repository or maintainer tags in package build names. Untagged local or third-party packages may not be detected, so the list is indicative rather than a complete provenance database.

Hardware details depend on kernel interfaces and installed tools. SMBIOS details generally require root and dmidecode. SMART checks run `smartctl -H` against detected whole disks. This reads health status and does not start a self-test; run as root. Boot manager detection uses the active UEFI `BootCurrent` entry when available. BIOS/Legacy firmware does not provide a standard interface for identifying which boot manager started the system, so configuration files found there are labeled as candidates. Log grouping uses text patterns and can misclassify messages; inspect context. Command output can vary with Slackware release and installed packages.
