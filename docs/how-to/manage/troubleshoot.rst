.. _troubleshoot:

Troubleshoot
============

This guide outlines some common issues, general methods to find out their cause, and their solutions.

----------------------
Troubleshoot with Juju
----------------------

Juju tracks the state of all applications it deploys. 
Any issues detected by the applications is picked up by Juju. You can check their status as follows:

.. code-block:: bash

   juju status

See `Troubleshoot your Juju deployment <https://documentation.ubuntu.com/juju/3.6/howto/manage-your-juju-deployment/troubleshoot-your-juju-deployment/#troubleshoot-your-deployment>`_ for more details.

-------------------------
Troubleshoot with kubectl
-------------------------

The ``kubectl`` command provides information about the state of pods and services running on the cluster. 
To restrict the output to your kubeflow deployment, use the desired namespace, which is the name of the Juju model, ``kubeflow`` in this case. 
For example:

.. code-block:: bash

   kubectl get pods -n kubeflow

See `Debug running pods <https://kubernetes.io/docs/tasks/debug/debug-application/debug-running-pod/>`_ for more details.

----------------
Common issues
----------------

~~~~~~~~~~~~~~~~
Deployment
~~~~~~~~~~~~~~~~

- If some pods are stuck in ``pending`` state, the most common cause is lacking storage capacity. Check that enough storage is allocated to the cluster and examine the persistent volume claims made by the pods.

- If your Jupyter notebook server is stuck in ``creating`` state, the most common cause is insufficient CPU available in your cluster. By default, notebook servers are created using 0.5 CPU. If this is the case, you can set the CPU share to 0 when you create the notebook server instead.

~~~~~~~~~~~~~~~~
Dashboard
~~~~~~~~~~~~~~~~

- In case of forgotten password, you can check the ``dex-auth`` user and password details as follows:

.. code-block:: bash

   juju config dex-auth static-username

.. code-block:: bash

   juju config dex-auth static-password

You can also set a new username and password by running this:

.. code-block:: bash

   juju config dex-auth static-username=admin

.. code-block:: bash

   juju config dex-auth static-password=AxWiJjk2hu4fFga7

- If you are using dynamic hostname resolution and a hostname ending with ``nip.io`` to evaluate Charmed Kubeflow, you may encounter issues due to DNS caching. By default, Ubuntu Server uses ``systemd resolved`` for DNS caching. 

You can change its behaviour with the following commands:

.. code-block:: bash

   sudo apt install -y resolvconf
   sudo systemctl enable --now resolvconf.service
   echo "nameserver 8.8.8.8" | sudo tee -a /etc/resolvconf/resolv.conf.d/head
   sudo resolvconf -u

~~~~~~~~~~~~~~~~
Other
~~~~~~~~~~~~~~~~

In some cases, Istio Pilot and Istio Gateway pods may not start as expected. 
This might be caused by Internet connectivity issues. 
Verify your connection is stable and has enough bandwidth.
