CentOS Installation
=======================

RackHD CentOS installation support multiple versions. Please refer to :ref:`os-installation-workflows-label` to see which versions are supported. We'll take CentOS 7 as the example below. If you want to install another version's CentOS, please replace with corresponding version's image, mirror, payload, etc.


Setup Mirror
-------------

A mirror should be setup firstly before installation.

* **Local ISO mirror**: Download CentOS ISO image, mount ISO image in a local server as the repository, http service for this repository is provided so that a node could access without proxy.
* **Local sync mirror**: Sync public site's mirror repository to local, http service for this repository is provided so that a node could access without proxy.
* **Public mirror**: The node could access a public or remote site's mirror repository with proxy.

.. include:: mirror-notes.rst

.. tabs::

    .. tab:: Local ISO Mirror

        .. code-block:: shell

            mkdir ~/iso && cd !/iso

            # Download iso file
            # You can choose a mirror in this site:
            # http://isoredirect.centos.org/centos/7/isos/x86_64/CentOS-7-x86_64-DVD-1708.iso
            # There are three type of ISOs (DVD ISO, Everything ISO, Minimal ISO), Minimal ISO is not supported

            wget http://mirror.math.princeton.edu/pub/centos/7/isos/x86_64/CentOS-7-x86_64-DVD-1708.iso

            # Create mirror folder
            mkdir -p /var/mirrors/centos

            # Replace {on-http-dir} with your own
            mkdir -p {on-http-dir}/static/http/mirrors

            # Mount iso
            sudo mount CentOS-7-x86_64-DVD-1708.iso /var/mirrors/centos

            # Replace {on-http-dir} with your own
            sudo ln -s /var/mirrors/centos {on-http-dir}/static/http/mirrors/


    .. tab:: Local Sync Mirror

        For CentOS local mirror, the mirrors are easily made by syncing public CentOS mirror site, on any recent distribution of CentOS:

        .. code::

            # Replace x with your own version

            sudo rsync --progress -av --delete --delete-excluded --exclude "local*" \
            --exclude "i386" rsync://centos.eecs.wsu.edu/x/ /var/mirrors/centos/x

    .. tab:: Public Mirror

        Add following block into httpProxies in ``/opt/monorail/config.json``, and restart on-http service.

        .. code::

            {
              "localPath": "/centos",
              "server": "http://mirror.centos.org/",
              "remotePath": "/centos/"
            },


Call API to Install OS
-----------------------

After the mirror is setup, We could download payload and call workflow API to install OS.

Get payload example.

.. code-block:: shell

    wget https://raw.githubusercontent.com/RackHD/RackHD/master/example/samples/install_centos_7_payload_minimal.json

Call OS installation workflow API to install OS. ``127.0.0.1:9090`` is according to the configuration ``address`` and ``port`` of ``httpEndPoints`` -> ``northbound-api-router`` in ``/opt/monorail/config.json``

.. code-block:: shell

    curl -X POST -H 'Content-Type: application/json' \
        -d @install_centos_payload_minimal.json \
        127.0.0.1:9090/api/current/nodes/{node-id}/workflows?name=Graph.InstallCentos | jq '.'

Please record the API's returned result, it's this workflow's Id (like ``342cce19-7385-43a0-b2ad-16afde072715``), it will be used to check result later.

.. include:: install-notes.rst

.. include:: check-result.rst
