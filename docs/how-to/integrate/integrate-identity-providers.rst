.. _integrate_identity_providers:

Integrate with identity providers
=================================

.. note::

   This content is intended for system admins.

This guide describes how you can authenticate in Charmed Kubeflow (CKF) via different Identity Providers (IdP) by configuring `Dex <https://charmhub.io/dex-auth>`_.

When authenticating through Dex, your identity data is stored using an external user-management system, such as a LDAP directory or a GitHub organisation. 
Dex uses `connectors <https://dexidp.io/docs/connectors/>`_ to authenticate a user against an identity provider.

You can integrate the supported IdPs with ``dex-auth`` charm following these steps:

1. `Add a connector <#add-a-connector-1>`_.
2. `Configure Dex issuer URL <#configure-dex-issuer-url-2>`_.

---------------------
Add a connector
---------------------

Each connector has its own configuration in YAML format, which is best described in each connector's documentation.

To add a new connector, pass the configuration to ``dex-auth`` via the ``connectors`` `configuration option <https://charmhub.io/dex-auth/configuration?channel=2.39/stable>`_:

``juju config dex-auth connectors=@connectors.yaml``

Where ``connectors.yaml`` is a ``.yaml`` file with a list of connector(s) configuration.

As an example of connector configuration, this is what you might use for ``connectors.yaml`` to configure Dex to authenticate against a Microsoft IdP:

.. code-block:: yaml

   - type: microsoft
     id: microsoft
     name: Microsoft
     config:
       clientID: $MICROSOFT_APPLICATION_ID
       clientSecret: $MICROSOFT_CLIENT_SECRET
       redirectURI: http://127.0.0.1:5556/dex/callback

------------------------
Configure Dex issuer URL
------------------------

When using a connector, fields like the ``redirectURI`` from the connector configuration must match the ``issuer-url`` configuration option in the ``dex-auth`` charm. 
To make sure that is the case, you can:

1. Verify the current value of Dex issuer URL as follows:

.. code-block:: bash

   juju config dex-auth issuer-url

2. Set it to match your deployment configuration:

.. code-block:: bash

   juju config dex-auth issuer-url=http://<domain-name>.cloudname.com/dex

For example, when using a cloud service like Azure it could look like this:

.. code-block:: bash

   juju config dex-auth issuer-url=https://my-charmed-kubeflow.uksouth.cloudapp.azure.com/dex

.. warning::

   After configuring it, connectors configurations must use it as Dex issuer URL all where it applies. 
