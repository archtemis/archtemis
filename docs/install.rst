.. include:: substitutions.rst

.. _install-page:


******************
Install Arch Linux
******************

The `Installation guide <https://wiki.archlinux.org/title/Installation_guide>`_ from the |arch|_ wiki is very detailed. I recommend you follow the official documentation step by step. However, there are some choices to be made at several steps. The different choices I made to install |arch|_ are summarized below. 

Bootable USB flash drive
========================

From the `Download <https://archlinux.org/download/>`_ page, get the ISO and the signatures files, for example:

.. code-block:: bash

    wget http://ftp.u-strasbg.fr/linux/distributions/archlinux/iso/2023.10.14/archlinux-2023.10.14-x86_64.iso
    wget https://archlinux.org/iso/2023.10.14/archlinux-2023.10.14-x86_64.iso.sig

With my existing existing Arch Linux installation, the signature is checked with:

.. code-block:: bash

    pacman-key -v archlinux-2023.10.14-x86_64.iso.sig

Different methods exist to create the `Bootable USB flash drive <https://wiki.archlinux.org/title/USB_flash_installation_medium>`_.  The copy of the ISO on the USB flash drive was done with ``root``:

.. code-block:: bash

    cp archlinux-2023.10.14-x86_64.iso /dev/disk/by-id/usb-SMI_USB_DISK_AA000000000000000107-0:0



Pre-installation
================


Set the console keyboard layout and font: 

.. code-block:: bash

    loadkeys us-acentos

Update the system clock:

.. code-block:: bash

    timedatectl set-timezone Europe/Paris


Partition the disks:

.. code-block:: bash

    fdisk /dev/nvme0n1
    # n to add partition.
    #  -  EFI (1GB)
    #  -  Linux for the OS (128G)
    #  -  swap (16G)
    #  -  Linux for the user's home (remaining space)
    # w to write partition table
    mkfs.ext4 -L ARCH /dev/nvme0n1p2
    mkswap -L SWAP /dev/nvme0n1p3
    mkfs.fat -n EFI -F 32 /dev/nvme0n1p1
    mkfs.ext4 -L HOME /dev/nvme0n1p4


Configure the system
====================

I started by installing additional packages to edit file during the configuration process:

.. code-block:: bash

    pacman -Sy vim
    pacman -Sy sudo # to edit /etc/sudoers

Time:

.. code-block:: bash

    ln -sf /usr/share/zoneinfo/Europe/Paris /etc/localtime

Localization:

.. code-block:: bash

    echo "KEYMAP=us-acentos" > /etc/vconsole.conf


Boot loader:

Install `GRUB <https://wiki.archlinux.org/title/GRUB>`_:

.. code-block:: bash

    pacman -Sy grub efibootmgr

.. code-block:: bash

    grub-install --target=x86_64-efi --bootloader-id=GRUB --efi-directory=/boot/
    grub-mkconfig -o /boot/grub/grub.cfg

