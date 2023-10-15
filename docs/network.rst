.. include:: substitutions.rst

.. _network-page:


*******
Network
*******

Parental controls
=================

In this use case, I configure my workstation such that my kids can browse internet but only on a controlled list of websites. The configuration relies on:

* the creation of :ref:`network-pc-linux` for my kids and a deidicated linux group to which my kids belong
* the configuration of :ref:`network-pc-firewall` using |nftables|_
* the configuration of a :ref:`network-pc-proxy` using |squid|_


.. _network-pc-linux:

Linux accounts and group
------------------------

First create the group ``children`` with ``gid`` 10000 (or whatever number):

::

    groupadd -g 10000 children


Create one linux account for each of your kid:

::

    useradd mykid1 -m -c 'Kid1 name'
    passwd mykid1
    useradd mykid2 -m -c 'Kid2 name'
    passwd mykid2

Then, change their primary group to ``children``:

::

    usermod -g children -G mykid1 mykid1
    usermod -g children -G mykid2 mykid2

.. _network-pc-firewall:

Firewall rules
--------------

Using |nftables|_, we will force the members of the ``children`` group to connect to the internet only through the :ref:`network-pc-proxy`. Install |nftables|_ as described in `Arch linux nftables wiki <https://wiki.archlinux.org/title/Nftables>`_ and start |nftables|_ by default at boot using ``systemd``:

::
    
    systemctl enable nftables


Then, the ``children`` group is allowed connection through port ``3128`` which is the listening port of the :ref:`network-pc-proxy`. Any other port connection is dropped by the firewall. This can be done by the following ``nft`` command lines:

::

    export CHILDREN_GID=10000
    export MY_WORKSTATION_IP='192.168.1.175, 127.0.0.1'
    nft add table inet filter
    nft add chain inet filter output '{ type filter hook output priority 0 ; policy accept ; }'
    nft add rule inet filter output tcp dport 3128 ip daddr "{ $MY_WORKSTATION_IP }" meta skgid $CHILDREN_GID counter log prefix \"[Nftables] accept children on proxy: \" flags all accept
    nft add rule inet filter output meta skgid $CHILDREN_GID counter log prefix \"[Nftables] deny children access: \" flags all drop


These new firewall rules must be saved in the default |nftables|_ configuration (``/etc/nftables.conf``) that is started at boot:

:: 

    nft -s list ruleset > /etc/nftables.conf

The ``/etc/nftables.conf`` configuration file should look like :download:`nftables.conf <../conf/etc/nftables.conf>`

.. _network-pc-proxy:

Proxy web
---------


Using |squid|_, we will restrict the members of the ``children`` group to access a white list of internet domain. Install |squid|_ as described in `Arch linux squid wiki <https://wiki.archlinux.org/title/Squid>`_ and start |squid|_ by default at boot using ``systemd``:


::
    
    systemctl enable squid


Create the file ``/etc/squid/domains.whitelist.txt`` with the list of internet domains that can be browsed by the members of the ``children`` group. This file should look like :download:`domains\.whitelist.txt <../conf/etc/squid/domains.whitelist.txt>`.


The |squid|_, will be configured with authentification such that we can know who is using the proxy. Authentification will be managed with the `Linux Pluggable Authentication Modules <https://wiki.archlinux.org/title/PAM>`_ (PAM). This way, the user is abble to connect to the proxy with standard linux password. To do so, it is necessary to create the ``/etc/pam.d/squid`` with the following lines such that |squid|_ can use PAM:

::

    auth            required        pam_unix.so
    account         required        pam_unix.so

The ``/etc/pam.d/squid`` file should look like :download:`squid <../conf/etc/pam.d/squid>`

Then, in the file ``/etc/squid/squid.conf``, add the following lines to require that any user should connect with linux login and password (which will be checked with PAM):

::

    auth_param basic program /usr/lib/squid/basic_pam_auth
    auth_param basic realm Squid proxy-caching web server
    auth_param basic children 5 startup=5 idle=1
    auth_param basic credentialsttl 2 hours
    acl pam_auth proxy_auth REQUIRED
    http_access deny !pam_auth


Define the ACL (access Control List) which contains the linux login of both kids. Here, it is important to add the directive ``proxy_auth`` with the list of users we want to control which internet domains can be browsed (note that other syntax should make it possible to directly based the ACL from the ``children`` group we have previously defined, without writing all users in this group).

::

    acl my_children proxy_auth mykid1 mykid2

Define the white list that the kids will be allowed to browse:

::

    acl whitelist_4_children dstdomain "/etc/squid/domains.whitelist.txt"

Finally, define the permission access (note that the ``all`` static ACL is required to avoid login popup loop, see `Squid authentification <https://wiki.squid-cache.org/Features/Authentication#how-do-i-prevent-authentication-loops>`_):

::

    http_access allow my_children whitelist_4_children
    http_access deny my_children all
    http_access allow pam_auth



The ``/etc/squid/squid.conf`` file should look like :download:`squid.conf <../conf/etc/squid/squid.conf>`

