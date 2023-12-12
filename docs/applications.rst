.. include:: substitutions.rst

.. _applications-page:


************
Applications
************

Multimedia
==========


.. _upnp-applications:

UPnP/DNLA
---------

To stream media I use `Rygel <https://wiki.archlinux.org/title/Rygel>`_ which is compatible with UPnP/DNLA. Any user can start ``rygel`` to share content from the home directory, it will scan the folders `Pictures`, `Videos` and `Music` by default.

If you run ``rygel`` behind a firewall, ensure that your firewall allows connections from your clients on the listening port. By default, whenever you start ``rygel``, the default configuration assigns a port dynamically which may change from one session to another if the ``/etc/rygel.conf`` sets ``port=0``.  Therefore, I recommend that the user starts ``rygel`` using always the same port, let's assume for example ``50000``. To choose a port which is available, you can run the following command:


.. code-block:: bash

    sudo lsof -nP -iTCP -sTCP:LISTEN

As I use |nftables|_ firewall, I have to allow the connection to port ``50000`` from clients  on my local network using the following ``nft`` commands:


.. code-block:: bash

    # list the different handles in the exiting nftables
    sudo nft -a list ruleset
    # add the rule after the appropriate handle number, e.g. 10
    sudo nft add rule inet filter input position 10 tcp dport 50000 ip saddr { 192.168.1.1-192.168.1.255 } accept comment \"allow rygel UPnP/DLNA\"
    # permanently save your nftables rules
    sudo bash -c 'nft -s list ruleset > /etc/nftables.conf'

Finally, start rygel as follows:

.. code-block:: bash

    rygel -g 5 -p 50000
