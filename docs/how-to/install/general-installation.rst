.. _general_installation:

General installation
====================

This guide describes the general steps required to install Charmed Kubeflow (CKF).

CKF can be installed on any `CNCF certified <https://www.cncf.io/certification/software-conformance/#logos>`_ Kubernetes (K8s), 
including `Canonical Kubernetes <https://ubuntu.com/kubernetes>`_, AKS, EKS, GKE, Openshift, and any ``kubeadm``-deployed cluster.

---------------------
Requirements
---------------------

Charmed Kubeflow requires a running K8s cluster with the following:
* Kubernetes version depending on the CKF version you are deploying. See :ref:`Supported versions <supported_kubeflow_versions>` for more details.
* A default `storage class <https://kubernetes.io/docs/concepts/storage/storage-classes/>`_ configured.

---------------------
Bootstrap Juju
---------------------

CKF is deployed to Kubernetes with `Juju`_. 
Before deployment, Juju must be bootstrapped to the K8s cluster. 
For bootstrapping instructions, see `Get started with Juju <https://documentation.ubuntu.com/juju/latest/tutorial/>`_.

-------------------------------
Create the ``kubeflow`` model
-------------------------------

To create a Juju model for CKF, run:

.. code-block:: bash

   juju add-model kubeflow

.. note::
   The model name must be ``kubeflow``.

See `Juju OLM | add-model <https://juju.is/docs/olm/juju-add-model>`_ for more details.

---------------------
Deploy CKF
---------------------

To deploy the most recent stable version of CKF, run:

.. code-block:: bash

   juju deploy kubeflow --trust

.. note::
   It may take up to 20 minutes for all charms to become active.

You can monitor the deployment status with Juju as follows:

.. code-block:: bash

   juju status --watch 5s

.. note::
   You can install a different version of CKF by using the ``--channel`` option. 
   See :ref:`Supported versions <supported_kubeflow_versions>` for more details.
   
.. _access_ckf_dashboard:

------------------------
Access CKF dashboard
------------------------

You can interact with CKF using its central dashboard, accessed through an IP address. 
To access it, you must:

1. Configure Dex to authenticate with the Identity Provider of your choice.
2. Find the dashboard's IP address.

~~~~~~~~~~~~~~~~
Configure Dex
~~~~~~~~~~~~~~~~

To configure Dex's built-in connector, set credentials by configuring ``dex-auth`` with a username and a password:

.. code-block:: bash

   juju config dex-auth static-username=<new username>
   juju config dex-auth static-password=<new password>

.. note::
   To use an external Identity Provider, see :ref:`Integrate with identity providers <integrate_with_identity_providers>` for more details.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Find the dashboard IP address
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If the deployment uses a LoadBalancer, you can find the dashboard's IP by running the following command:

.. code-block:: bash

   kubectl -n kubeflow get svc istio-ingressgateway-workload -o jsonpath='{.status.loadBalancer.ingress[0].ip}'

See `kubectl get <https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get>`_ for more details.

If you have configured Istio Gateway to use a different gateway service type such as ClusterIP or NodePort, 
the dashboard should be accessible at that service's IP. 
See `Istio Gateway configurations <https://charmhub.io/istio-gateway/configurations?channel=1.22/stable>`_ for more information.

.. note::
   If DNS is required, use the resolvable address from ``istio-ingressgateway``.

.. tip::
   To access the dashboard remotely, you can obtain the IP over SSH and a SOCKS proxy. 
   See :ref:`How to set up SSH <how_to_set_up_ssh>` for more details.

~~~~~~~~
Log in
~~~~~~~~

Once you have accessed the dashboard IP address, log in using the credentials matching the identity provider you are using.
