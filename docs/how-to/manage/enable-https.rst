.. _enable_https:

Enable HTTPS
============

This guide describes how to enable HTTPS on Charmed Kubeflow (CKF) through a Transport Layer Security (TLS) certificate.

TLS is a cryptographic protocol designed to provide secure communication over a computer network. 
It ensures privacy, data integrity, and authenticity between communicating applications. 
By enabling HTTPS, which uses TLS, in your Charmed Kubeflow setup, you protect the data transmitted between clients and the Kubeflow services from being intercepted or tampered with. 
See `Transport Layer Security <https://en.wikipedia.org/wiki/Transport_Layer_Security>`_ for more details.

---------------------
Requirements
---------------------

Before you can enable HTTPS in your CKF deployment, you need to obtain a domain and create a TLS certificate for that domain. 
This process varies depending on the cloud provider or certificate authority you choose. 
Below are some resources to help you generate certificates and set up domain names on different cloud platforms:

~~~~~~~~~~~~~~~~~~~~
Set up a domain name
~~~~~~~~~~~~~~~~~~~~

* `AWS Route 53: Registering a New Domain <https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/domain-register.html>`_.
* `Azure DNS: Create a DNS Zone and Record <https://docs.microsoft.com/en-us/azure/dns/dns-getstarted-portal>`_.
* `Google Cloud DNS: Creating a Managed Zone <https://cloud.google.com/dns/docs/zones>`_.

~~~~~~~~~~~~~~~~~~~~~~~~~
Obtain a TLS certificate
~~~~~~~~~~~~~~~~~~~~~~~~~

* `AWS Certificate Manager <https://aws.amazon.com/certificate-manager/>`_.
* `Azure Key Vault Certificates <https://docs.microsoft.com/en-us/azure/key-vault/certificates/>`_.
* `Google Cloud Certificate Manager <https://cloud.google.com/certificate-manager/docs/overview>`_.
* `Self-Signed Certificates <https://www.openssl.org/>`_.

-----------------------------------
Pass TLS certificate and key values
-----------------------------------

To pass the TLS certificate and key values, do the following:

1. `Create a user secret <https://documentation.ubuntu.com/juju/3.6/howto/manage-secrets/#add-a-secret>`_ to hold the TLS certificate and key values (as strings):

.. code-block:: bash

   juju add-secret istio-tls-secret tls-crt="$(cat CERT_FILE)" tls-key="$(cat KEY_FILE)"

where:

* ``tls-crt`` holds the value of the TLS certificate file as a string.
* ``tls-key`` holds the value of the TLS key file as a string.

2. `Grant istio-pilot access <https://documentation.ubuntu.com/juju/3.6/howto/manage-secrets/#grant-access-to-a-secret>`_ to the secret:

.. code-block:: bash

   juju grant-secret istio-tls-secret istio-pilot

3. Pass the secret ID as configuration:

.. code-block:: bash

   juju config istio-pilot tls-secret-id=secret:<secret ID resulting from step 1>

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Configure self-signed certificates
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you are using self-signed certificates, you need to configure the ``oidc-gatekeeper`` with the Certificate Authority (CA) bundle to ensure proper validation.

.. note::

   This step is typically unnecessary for public cloud certificates, as their CAs are already trusted by browsers.

To configure ``oidc-gatekeeper`` with the CA, follow these steps:

1. Create a CA bundle file that includes your self-signed CA certificate:

.. code-block:: bash

   cat <<EOF > /path/to/ca-bundle.crt

   -----BEGIN CERTIFICATE-----

   # Your CA certificate content

   -----END CERTIFICATE-----
   EOF

2. Update the ``oidc-gatekeeper`` configuration to use the CA bundle:

.. code-block:: bash

   juju config oidc-gatekeeper ca-bundle="$(cat /path/to/ca-bundle.crt)"

------------------------------------
Migrate from configuration to action
------------------------------------

.. note::

   These instructions should only run if migrating from ``istio-operators`` < 1.17 rev1197

Passing the TLS certificate and key using Juju secrets means replacing the ``ssl-*`` configuration options for the ``istio-pilot``. 
This migration is as simple as doing:

.. code-block:: bash

   juju refresh istio-pilot

It implies the following considerations:

1. The ``ssl-key`` and ``ssl-crt`` values passed as configuration options will be lost. It is recommended to save them before upgrading.
2. A downtime is expected while upgrading to newer versions of ``istio-pilot`` as the Ingress Gateway has to be reconfigured. This is expected to happen between the ``juju refresh`` command and the time after running the ``set-tls`` action.
3. Migrating and not setting the TLS certificate and private key values can lead to unexpected results. Make sure they are set.

To upgrade and re-configure, do the following:

1. Get existing configuration values and save them:

.. code-block:: bash

   juju config istio-pilot ssl-crt
   juju config istio-pilot ssl-key

2. Refresh the ``istio-operators`` charms to the desired version:

.. code-block:: bash

   juju refresh istio-pilot --channel $ISTIO_PILOT_CHANNEL
   juju refresh istio-ingressgateway --channel $ISTIO_INGRESSGATEWAY_CHANNEL
   juju status istio-pilot/<unit-number> --wait 5s # Wait for the unit to go to active and idle

3. `Pass TLS certificate and key <#pass-tls-certificate-and-key-values>`_ using Juju secrets.
