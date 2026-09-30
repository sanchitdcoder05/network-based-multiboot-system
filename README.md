# Network-Based Multi-Boot System

A network-based Linux multi-boot system that enables multiple Linux environments to be booted over a network using PXE, TFTP, NFS, dnsmasq, and HTTP-based Live RAM boot.

## Overview

This project provides a network boot infrastructure where a client machine can obtain network configuration, retrieve boot files, and select different Linux environments from a multi-boot menu.

The system supports:

- PXE network boot
- DHCP using dnsmasq
- TFTP boot file delivery
- NFS-based Linux environments
- HTTP-based Live RAM boot
- Multiple Linux distributions
- QEMU-based testing

## Architecture

```text
                         PXE CLIENT
                             |
                             | DHCP
                             v
                    +------------------+
                    |     dnsmasq      |
                    |   DHCP + TFTP    |
                    +--------+---------+
                             |
                             | TFTP
                             v
                         /srv/tftp
                             |
                             v
                       Network Boot
                             |
                             v
                      Multi-Boot Menu
                             |
             +---------------+---------------+
             |               |               |
             v               v               v
          Ubuntu          Lubuntu         TinyCore
           NFS              NFS
             |               |
             +-------+-------+
                     |
                     v
                  NFS Server
                  /srv/nfs
Technologies
PXE / Network Boot
TFTP
NFS
dnsmasq
PXELINUX
DHCP
HTTP
Linux
QEMU
Supported Boot Options

The current network boot configuration provides:

Ubuntu Linux via NFS
Lubuntu Linux via NFS
TinyCore Linux
Ubuntu Linux via HTTP Live RAM
Lubuntu Linux via HTTP Live RAM
Components
DHCP and TFTP

dnsmasq provides DHCP and TFTP services.

Configuration:

configs/dnsmasq/sbl.conf

Current DHCP range:

192.168.122.50 - 192.168.122.150

TFTP root:

/srv/tftp
NFS Server

Ubuntu and Lubuntu filesystems are exported through NFS:

/srv/nfs/ubuntu
/srv/nfs/lubuntu

NFS configuration:

configs/nfs/exports
TFTP Boot Configuration

The SBL network boot configuration is stored in:

configs/tftp/boot.cfg

The PXELINUX multi-boot configuration is stored in:

configs/tftp/pxelinux-default
Repository Structure
network-based-multiboot-system/
├── configs/
│   ├── dnsmasq/
│   │   └── sbl.conf
│   ├── nfs/
│   │   └── exports
│   └── tftp/
│       ├── boot.cfg
│       └── pxelinux-default
├── docs/
├── qemu/
├── scripts/
├── .gitignore
└── README.md
Quick Setup Guide
1. Clone the Repository
git clone https://github.com/sanchitdcoder05/network-based-multiboot-system.git
cd network-based-multiboot-system
2. Install Required Packages

This project was developed and tested on Fedora Linux.

Install the required packages:

sudo dnf install dnsmasq nfs-utils syslinux-tftpboot qemu-system-x86

Depending on the Fedora version, package names may differ slightly.

3. Create Required Directories

Create the TFTP and NFS directories:

sudo mkdir -p /srv/tftp
sudo mkdir -p /srv/nfs/ubuntu
sudo mkdir -p /srv/nfs/lubuntu
4. Configure dnsmasq

Copy the project dnsmasq configuration:

sudo cp configs/dnsmasq/sbl.conf /etc/dnsmasq.d/sbl.conf

The current configuration provides:

DHCP Range: 192.168.122.50 - 192.168.122.150
Lease Time: 12 hours
TFTP Root:  /srv/tftp

The relevant configuration is:

bind-interfaces

dhcp-range=192.168.122.50,192.168.122.150,12h

enable-tftp
tftp-root=/srv/tftp
5. Configure NFS

Copy the NFS export configuration:

sudo cp configs/nfs/exports /etc/exports

The configuration exports:

/srv/nfs/ubuntu
/srv/nfs/lubuntu

Apply the exports:

sudo exportfs -rav

Verify:

sudo exportfs -v
6. Prepare the TFTP Directory

The TFTP directory should contain the required network boot files.

Example:

/srv/tftp/
├── boot.cfg
├── vmlinuz
└── initrd

Copy the project boot configuration:

sudo cp configs/tftp/boot.cfg /srv/tftp/boot.cfg

The required kernel and initramfs files must also be placed in:

/srv/tftp/

The exact kernel and initramfs files depend on the Linux distribution being used.

7. Prepare the NFS Filesystems

Place the extracted Linux filesystem contents in:

/srv/nfs/ubuntu/
/srv/nfs/lubuntu/

For example:

/srv/nfs/ubuntu/
├── .disk/
├── EFI/
├── boot/
├── casper/
├── dists/
└── pool/

/srv/nfs/lubuntu/
├── .disk/
├── EFI/
├── boot/
├── casper/
├── dists/
└── pool/

The operating-system files are intentionally not included in this Git repository because they are large binary/distribution files.

8. Start dnsmasq

Enable and start dnsmasq:

sudo systemctl enable --now dnsmasq

Check its status:

systemctl status dnsmasq

If another DHCP server is already active on the same network, make sure the test network is isolated or otherwise configured appropriately. Running two DHCP servers on the same network can cause conflicts.

9. Start the NFS Server

Enable and start NFS:

sudo systemctl enable --now nfs-server

Check the status:

systemctl status nfs-server

Verify the exported filesystems:

sudo exportfs -v
10. Network Configuration

The example configuration uses the network:

192.168.122.0/24

with DHCP addresses:

192.168.122.50 - 192.168.122.150

The NFS boot configuration currently references:

192.168.122.1

as the NFS server.

If the server uses a different IP address, update the nfsroot values in:

configs/tftp/pxelinux-default

For example:

nfsroot=<SERVER_IP>:/srv/nfs/ubuntu

and:

nfsroot=<SERVER_IP>:/srv/nfs/lubuntu
11. PXE Client

Connect a PXE-capable client to the same test network.

Enable network/PXE boot in the client's firmware.

The expected process is:

Client starts
     |
     v
DHCP request
     |
     v
dnsmasq assigns IP
     |
     v
TFTP boot files requested
     |
     v
Network boot configuration loaded
     |
     v
Multi-boot menu
     |
     +----------------------+
     |          |           |
     v          v           v
  Ubuntu    Lubuntu     TinyCore
    NFS        NFS
12. HTTP Live RAM Boot

The project also contains HTTP Live RAM boot entries.

These entries use:

iso-url=http://10.0.2.2:8080/ubuntu-26.04-desktop-amd64.iso

and:

iso-url=http://10.0.2.2:8080/lubuntu-26.04-desktop-amd64.iso

Therefore, an HTTP server must be running and serving the corresponding ISO files at the expected URLs.

The HTTP boot configuration can be found in:

configs/tftp/pxelinux-default

The IP address and ISO filenames should be changed if a different HTTP server is used.

13. QEMU Testing

The network boot environment can also be tested using QEMU instead of a physical PXE client.

The purpose of QEMU testing is to validate the network boot configuration in a virtualized environment before using physical hardware.

QEMU-specific commands and configurations can be added under:

qemu/
Configuration Files
dnsmasq
configs/dnsmasq/sbl.conf

Provides:

DHCP
TFTP
Network boot infrastructure
NFS
configs/nfs/exports

Defines the Ubuntu and Lubuntu NFS exports.

SBL Boot Configuration
configs/tftp/boot.cfg

Defines the SBL network boot configuration.

PXELINUX
configs/tftp/pxelinux-default

Contains the multi-boot menu and boot parameters.

Why Large Files Are Not Included

This repository intentionally stores configuration and documentation instead of complete operating-system images.

The following files are not included:

Linux kernel images
initramfs files
Ubuntu/Lubuntu ISO files
Extracted Ubuntu filesystem
Extracted Lubuntu filesystem
Virtual machine disk images

This keeps the Git repository lightweight and makes it easier to clone and maintain.

The required operating-system files can be obtained separately from their respective Linux distributions.

Troubleshooting
Check dnsmasq
systemctl status dnsmasq

View logs:

journalctl -u dnsmasq
Check NFS
systemctl status nfs-server

View exports:

sudo exportfs -v
Check TFTP Files
ls -lah /srv/tftp
Check NFS Filesystems
ls -lah /srv/nfs/ubuntu
ls -lah /srv/nfs/lubuntu
Check Network Address
ip addr
Check Listening Services
sudo ss -lunpt
Project Status

Working network-based multi-boot environment with:

DHCP and TFTP using dnsmasq
PXE/PXELINUX boot menu
Ubuntu NFS boot
Lubuntu NFS boot
TinyCore support
HTTP Live RAM boot
QEMU-based testing
Purpose

This project demonstrates practical experience with:

Network boot infrastructure
PXE
DHCP
TFTP
NFS
Linux system administration
Bootloaders
Network filesystems
Virtualized testing
Multi-OS deployment
License

This repository contains configuration and project documentation.

Linux distributions and other third-party components used by this project remain subject to their respective licenses.
