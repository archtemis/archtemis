.. include:: substitutions.rst

.. _system-page:

******
System
******

system logs
===========

dmesg
-----

To print kenel log use `dmesg <https://man.archlinux.org/man/dmesg.1>`_, for example:

::

    # Print log with human-readable output and no pager
    dmesg -H -P

journalctl
----------

For `Systemd <https://wiki.archlinux.org/title/Systemd>`_ services, use `Systemd/Journal <https://wiki.archlinux.org/title/Systemd/journal>`_  with command such as:

::

    # See log of a systemd unit for the current boot:
    journalctl -u systemd-vconsole-setup -b
    
    # Follow new messages:
    journalctl -f

