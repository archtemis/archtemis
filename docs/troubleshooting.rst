.. include:: substitutions.rst

.. _troubleshooting-page:

***************
troubleshooting
***************

Systemd
=======

vconsole
--------

From ``dmesg``,  the followinf information was reported for the ``systemd-vconsole-setup`` unit:


::

    Aug 26 18:48:33 archlinux systemd-vconsole-setup[248]: /usr/bin/loadkeys failed with exit status 1.
    Aug 26 18:48:33 archlinux systemd-vconsole-setup[248]: KD_FONT_OP_GET failed while trying to get the font metadata: Invalid argument
    Aug 26 18:48:33 archlinux systemd-vconsole-setup[248]: Fonts will not be copied to remaining consoles
    Aug 26 18:48:33 archlinux systemd[1]: systemd-vconsole-setup.service: Main process exited, code=exited, status=1/FAILURE
    Aug 26 18:48:33 archlinux systemd[1]: systemd-vconsole-setup.service: Failed with result 'exit-code'.
    Aug 26 18:48:33 archlinux systemd[1]: Failed to start Virtual Console Setup.


The ``/etc/mkinitcpio.conf`` had the following ``HOOKS``:

::

    HOOKS=(base systemd plymouth autodetect modconf block filesystems keyboard fsck)


Adding ``sd-vconsole`` in the list fixed the problrem:

::

    HOOKS=(base systemd plymouth autodetect modconf block filesystems keyboard sd-vconsole fsck)


Then run ``sudo mkinitcpio -P``.
  

