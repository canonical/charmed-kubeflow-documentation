.. _index_security:

Security
========

This guide presents an overview of security features and guidance for hardening the security of Charmed Kubeflow (CKF) deployments.

.. toctree::
    :maxdepth: 1
    :hidden:

    Authentication <authentication>
    Authorisation <authorisation>
    Cryptography <cryptography>

Environment
-----------

The environment where CKF applications operate can be divided into two components:

1. Kubernetes (K8s).
2. `Juju`_.

Kubernetes
~~~~~~~~~~

CKF can be deployed on top of several K8s distributions.

The following table provides references to the security documentation for the main supported cloud platforms:

+--------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Cloud              | Security guide                                                                                                                                                                                                                                                                                                                           | 
+====================+==========================================================================================================================================================================================================================================================================================================================================+
| Charmed Kubernetes | `Security in Charmed Kubernetes <https://ubuntu.com/kubernetes/charmed-k8s/docs/security>`_                                                                                                                                                                                                                                              |
+--------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| AWS EKS            | `AWS best practices <https://aws.amazon.com/architecture/security-identity-compliance>`_, `AWS security credentials <https://docs.aws.amazon.com/IAM/latest/UserGuide/security-creds.html>`_, `Security in EKS <https://docs.aws.amazon.com/eks/latest/userguide/security.html>`_                                                        | 
+--------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Azure AKS          | `Azure best practices <https://learn.microsoft.com/en-us/azure/security/fundamentals/best-practices-and-patterns>`_, `Managed identities for Azure resource <https://learn.microsoft.com/en-us/entra/identity/managed-identities-azure-resources/>`_, `Security in AKS <https://learn.microsoft.com/en-us/azure/aks/concepts-security>`_ | 
+--------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Google GKE         | `Security overview <https://cloud.google.com/kubernetes-engine/docs/concepts/security-overview>`_                                                                                                                                                                                                                                        | 
+--------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

Juju
~~~~

Juju is the component responsible for orchestrating the entire lifecycle, from deployment to day-2 operations, of CKF applications. 
Therefore, it is critical to set it up securely.

See `Juju security <https://documentation.ubuntu.com/juju/3.6/explanation/juju-security/>`_ for more details.

Cloud credentials
^^^^^^^^^^^^^^^^^

When configuring cloud credentials to be used with Juju, ensure that users have correct permissions to operate at the required level on the K8s cluster.

Juju superusers responsible for bootstrapping and managing controllers require elevated permissions to manage several kinds of resources.
For this reason, the K8s user used for bootstrapping and managing deployments should have full permissions, including the ability to ``create``, ``delete``, ``patch``, and ``list``:

* Namespaces.
* Services.
* Deployments.
* Stateful sets.
* Pods.
* Persistent volume claims (PVCs).

.. note::
    In general, it is common practice to run Juju using the K8s admin role to ensure full permissions on the cluster.

Juju users
^^^^^^^^^^

It is essential that Juju users are granted only the minimum permissions required for their scope of operations. 
See `User access levels <https://juju.is/docs/juju/user-permissions>`_ for more details.

Juju user credentials must be stored securely and rotated regularly to reduce the risk of unauthorized access due to credential leakage.

Applications
------------

This section outlines methods to enhance the security of your CKF applications.

Base images
~~~~~~~~~~~

CKF runs on top of `Rock <https://documentation.ubuntu.com/rockcraft/en/latest/explanation/rocks/#rocks-explanation>`_ images, 
available in `these Github repositories <https://github.com/search?q=org%3Acanonical+topic%3Arocks+topic%3Akubeflow&type=repositories>`_, 
on top of Ubuntu 22.04.

Security upgrades
~~~~~~~~~~~~~~~~~

CKF uses pinned revisions of Rocks. 
`Charms <https://documentation.ubuntu.com/juju/3.6/reference/charm/#charm>`_ are refreshed to a newer revision periodically, 
but each revision is pointing to a specific `Rock hash <https://github.com/canonical/notebook-operators/blob/track/1.10/charms/jupyter-controller/metadata.yaml#L16>`_. 
This provides reproducible and secure environments.

New versions of these Rocks and Charms may be released to provide patching of vulnerabilities (CVEs).

It is important to refresh charms regularly to make sure the workload is as secure as possible.
See :ref:`Upgrade <upgrade>` for details on how to upgrade CKF.

Encryption
~~~~~~~~~~

Encryption for CKF is enabled via HTTPs, through a Transport Layer Security (TLS) certificate.
See :ref:`Enable HTTPS <enable-https>` for more details.

CKF uses `Istio Gateway <https://charmhub.io/istio-gateway>`_ as the ingress, 
and this gateway is configured via `Istio Pilot <https://charmhub.io/istio-pilot>`_ with the `TLS secret <https://charmhub.io/istio-pilot/configurations#tls-secret-id>`_.

Authentication
~~~~~~~~~~~~~~

See :ref:`Authentication <authentication>` to learn about how authentication works in CKF.

Authorisation
~~~~~~~~~~~~~

See :ref:`Authorisation <authorisation>` to learn about how authorisation works in CKF.

Cryptography
~~~~~~~~~~~~

See :ref:`Cryptography <cryptography>` for an overview of all cryptographic processes in CKF.

Monitoring and auditing
~~~~~~~~~~~~~~~~~~~~~~~

CKF provides integration with the `Canonical Observability Stack (COS) <https://charmhub.io/topics/canonical-observability-stack>`_.
To reduce the blast radius of infrastructure disruptions, it is generally recommended to deploy COS and the observed application in separate, isolated environments. 
See `COS production deployments best practices <https://charmhub.io/topics/canonical-observability-stack/reference/best-practices>`_ for more details.

See :ref:`Integrate with COS <integrate-with-cos>` to learn how CKF and COS can be integrated using Juju.