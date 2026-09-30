# Network-Based Multi-Boot System

A network-based Linux multi-boot system that enables multiple Linux environments to be booted over a network using PXE, TFTP, NFS, dnsmasq, and HTTP-based live boot.

## Overview

This project provides a network boot infrastructure where a client machine can obtain network configuration, retrieve boot files, and select different Linux environments from a multi-boot menu.

The system supports both NFS-based Linux environments and HTTP-based Live RAM boot.

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
              +--------------+--------------+
              |              |              |
              v              v              v
           Ubuntu         Lubuntu        TinyCore
             NFS            NFS
              |              |
              +------+-------+
                     |
                     v
                  NFS Server
                  /srv/nfs
