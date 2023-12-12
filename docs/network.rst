.. include:: substitutions.rst

.. _network-page:


*******
Network
*******

parental controls
=================

In this use case, I configure my workstation such that my kids can browse internet but only on a controlled list of websites. The configuration relies on:

* the creation of :ref:`network-pc-linux` for my kids and a deidicated linux group to which my kids belong
* the configuration of :ref:`network-pc-firewall` using |nftables|_
* the configuration of a :ref:`network-pc-proxy` using |squid|_


.. _network-pc-linux:

Linux accounts and group
------------------------

First create the group ``children`` with ``gid`` 10000 (or whatever number):

.. code-block:: bash

    groupadd -g 10000 children


Create one linux account for each of your kid:

.. code-block:: bash

    useradd mykid1 -m -c 'Kid1 name'
    passwd mykid1
    useradd mykid2 -m -c 'Kid2 name'
    passwd mykid2

Then, change their primary group to ``children``:

.. code-block:: bash

    usermod -g children -G mykid1 mykid1
    usermod -g children -G mykid2 mykid2

.. _network-pc-firewall:

Firewall rules
--------------

Using |nftables|_, we will force the members of the ``children`` group to connect to the internet only through the :ref:`network-pc-proxy`. Install |nftables|_ as described in `Arch linux nftables wiki <https://wiki.archlinux.org/title/Nftables>`_ and start |nftables|_ by default at boot using ``systemd``:

.. code-block:: bash
    
    systemctl enable nftables


Then, the ``children`` group is allowed connection through port ``3128`` which is the listening port of the :ref:`network-pc-proxy`. Any other port connection is dropped by the firewall. This can be done by the following ``nft`` command lines:

.. code-block:: bash

    export CHILDREN_GID=10000
    export MY_WORKSTATION_IP='192.168.1.175, 127.0.0.1'
    nft add table inet filter
    nft add chain inet filter output '{ type filter hook output priority 0 ; policy accept ; }'
    nft add rule inet filter output tcp dport 3128 ip daddr "{ $MY_WORKSTATION_IP }" meta skgid $CHILDREN_GID counter log prefix \"[Nftables] accept children on proxy: \" flags all accept
    nft add rule inet filter output meta skgid $CHILDREN_GID counter log prefix \"[Nftables] deny children access: \" flags all drop


These new firewall rules must be saved in the default |nftables|_ configuration (``/etc/nftables.conf``) that is started at boot:

.. code-block:: bash

    nft -s list ruleset > /etc/nftables.conf

The ``/etc/nftables.conf`` configuration file should look like :download:`nftables.conf <../conf/etc/nftables.conf>`

.. _network-pc-proxy:

Proxy web
---------


Using |squid|_, we will restrict the members of the ``children`` group to access a white list of internet domain. Install |squid|_ as described in `Arch linux squid wiki <https://wiki.archlinux.org/title/Squid>`_ and start |squid|_ by default at boot using ``systemd``:


.. code-block:: bash
    
    systemctl enable squid


Create the file ``/etc/squid/domains.whitelist.txt`` with the list of internet domains that can be browsed by the members of the ``children`` group. This file should look like :download:`domains\.whitelist.txt <../conf/etc/squid/domains.whitelist.txt>`.


The |squid|_, will be configured with authentification such that we can know who is using the proxy. Authentification will be managed with the `Linux Pluggable Authentication Modules <https://wiki.archlinux.org/title/PAM>`_ (PAM). This way, the user is abble to connect to the proxy with standard linux password. To do so, it is necessary to create the ``/etc/pam.d/squid`` with the following lines such that |squid|_ can use PAM:

.. code-block:: bash

    auth            required        pam_unix.so
    account         required        pam_unix.so

The ``/etc/pam.d/squid`` file should look like :download:`squid <../conf/etc/pam.d/squid>`

Then, in the file ``/etc/squid/squid.conf``, add the following lines to require that any user should connect with linux login and password (which will be checked with PAM):

.. code-block:: bash

    auth_param basic program /usr/lib/squid/basic_pam_auth
    auth_param basic realm Squid proxy-caching web server
    auth_param basic children 5 startup=5 idle=1
    auth_param basic credentialsttl 2 hours
    acl pam_auth proxy_auth REQUIRED
    http_access deny !pam_auth


Define the ACL (access Control List) which contains the linux login of both kids. Here, it is important to add the directive ``proxy_auth`` with the list of users we want to control which internet domains can be browsed (note that other syntax should make it possible to directly based the ACL from the ``children`` group we have previously defined, without writing all users in this group).

.. code-block:: bash

    acl my_children proxy_auth mykid1 mykid2

Define the white list that the kids will be allowed to browse:

.. code-block:: bash

    acl whitelist_4_children dstdomain "/etc/squid/domains.whitelist.txt"

Finally, define the permission access (note that the ``all`` static ACL is required to avoid login popup loop, see `Squid authentification <https://wiki.squid-cache.org/Features/Authentication#how-do-i-prevent-authentication-loops>`_):

.. code-block:: bash

    http_access allow my_children whitelist_4_children
    http_access deny my_children all
    http_access allow pam_auth



The ``/etc/squid/squid.conf`` file should look like :download:`squid.conf <../conf/etc/squid/squid.conf>`


Wi-Fi
=====

Eduroam
-------

`Eduroam <https://eduroam.org>`_  is an international Wi-Fi internet access roaming service for users in research. 
Since version v2.10, `wpa_supplicant <https://wiki.archlinux.org/title/Wpa_supplicant>`_ uses `OpenSSL <https://wiki.archlinux.org/title/OpenSSL>`_ 3 (see `CHANGELOG <https://w1.fi/cgit/hostap/plain/wpa_supplicant/ChangeLog>`_) which seems not being compatible with Eduroam connections. I managed to connect to Eduroam by downgrading wpa_supplicant to version v2.9:

.. code-block:: bash

    # download wpa_supplicant v2.9 from Arch Linux archives
    wget https://archive.archlinux.org/packages/w/wpa_supplicant/wpa_supplicant-2%3A2.9-8-x86_64.pkg.tar.zst

    # install the package
    pacman -U 'wpa_supplicant-2 3A2.9-8-x86_64.pkg.tar.zst'

    # install OpenSSL 1 required by version v2.9
    pacman -Sy openssl-1.1

    # restart the wpa_supplicant systemd service
    systemctl restart wpa_supplicant

In order to avoid wpa_supplicant being ugraded by ``pacman -Syu``, edit the file ``/etc/pacman.conf`` and add:

.. code-block:: bash

    IgnorePkg = wpa_supplicant


After this modification, I was able to connect using the Network Manager widget in KDE plasma.


Note that similar issue and solution were described from Ubuntu 22.04 (see `here <https://forum.ubuntu-fr.org/viewtopic.php?id=2072098>`_).
