.. include:: substitutions.rst

.. _install-page:


******************
Install Arch Linux
******************

The `Installation guide <https://wiki.archlinux.org/title/Installation_guide>`_ from the |arch|_ wiki is very detailed. The different steps I have performed to install |arch|_ are summarized below. 

Bootable USB flash drive
========================

From the `Download <https://archlinux.org/download/>`_ page, get the ISO and the signatures files, for example:

::

    wget http://ftp.u-strasbg.fr/linux/distributions/archlinux/iso/2023.10.14/archlinux-2023.10.14-x86_64.iso
    wget https://archlinux.org/iso/2023.10.14/archlinux-2023.10.14-x86_64.iso.sig

With my existing existing Arch Linux installation, the signature is checked with:

::

    pacman-key -v archlinux-2023.10.14-x86_64.iso.sig

Different methods exist to create the `Bootable USB flash drive <https://wiki.archlinux.org/title/USB_flash_installation_medium>`_.  The copy of the ISO on the USB flash drive ws done with ``root``:

::

    cp archlinux-2023.10.14-x86_64.iso /dev/disk/by-id/usb-SMI_USB_DISK_AA000000000000000107-0:0

