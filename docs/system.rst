.. include:: substitutions.rst

.. _system-page:

******
System
******

System logs
===========

dmesg
-----

To print kernel log use `dmesg <https://man.archlinux.org/man/dmesg.1>`_, for example:

.. code-block:: bash

    # Print log with human-readable output and no pager
    dmesg -H -P

journalctl
----------

For `Systemd <https://wiki.archlinux.org/title/Systemd>`_ services, use `Systemd/Journal <https://wiki.archlinux.org/title/Systemd/journal>`_  with command such as:

.. code-block:: bash

    # See log of a systemd unit for the current boot:
    journalctl -u systemd-vconsole-setup -b
    
    # Follow new messages:
    journalctl -f


Hardware information
====================

lshw, dmidecode
---------------

Use ``lshw`` or ``dmidecode``. For example, to get hardware information about RAM memory:

.. code-block:: bash

    sudo dmidecode -t memory
    sudo lshw -C memory

lsblk
-----

List information about block devices. For example, to get the partition list with their UUID:

.. code-block:: bash

    lsblk -f

fdisk
-----

Display the partition table:

.. code-block:: bash

    fdisk -l

To modify the partition table:

.. code-block:: bash

    fdisk /dev/nvme0n1
