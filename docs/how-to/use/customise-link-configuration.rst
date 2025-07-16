.. _customise_link_configuration:

Customise link configuration
============================

The `Kubeflow Central Dashboard <https://www.kubeflow.org/docs/components/central-dash/overview/>`_ provides access to Kubeflow's components. 
This guide describes how to customise the link configuration of your Kubeflow dashboard.

---------------------
Configure links
---------------------

To define your own links on the Kubeflow dashboard, use the charm config fields:

* ``additional-menu-links``.
* ``additional-external-links``.
* ``additional-quick-links``.
* ``additional-dashboard-links``.

For example, to create external links, define ``my_external_links.yaml`` as follows:

.. code-block:: yaml

    - text: My Container Registry
        link: http://my.company.com/registry
        icon: apps

Where:

* ``text``: the text shown on the dashboard.
* ``link``: the full link.
* ``icon``: any icon from `here <https://kevingleason.me/Polymer-Todo/bower_components/iron-icons/demo/index.html>`_.

Now pass this config to `Juju`_ by:

.. code-block:: bash

   juju config kubeflow-dashboard additional-external-links=@my_external_links.yaml

This results in your link(s) being included on the dashboard. 
See the bottom of the left-hand sidebar:

.. image:: https://assets.ubuntu.com/v1/f637eab8-link%20config1.png

---------------------
Add to existing links
---------------------

You can add links to an existing set of them by modifying the existing ``juju config`` value. 
For example, add documentation links as follows:

.. code-block:: bash

   juju config kubeflow-dashboard additional-documentation-links | yq -P > links_to_edit.yaml

Where ``links_to_edit.yaml`` contains:

.. code-block:: yaml

    - text: My Awesome Tutorial
        link: http://my.company.com/tutorial1
    - text: My Awesomer Tutorial
        link: http://my.company.com/tutorial2

Now push the new settings back to Juju with:

.. code-block:: bash

   juju config kubeflow-dashboard additional-documentation-links=@links_to_edit.yaml

.. image:: https://assets.ubuntu.com/v1/dfd990b6-link%20config2.png

-----------------------------
Customise the link order
-----------------------------

Links are shown by default in alphabetical order. 
To modify this, use the following config fields:

* ``menu-link-order``.
* ``external-link-order``.
* ``quick-link-order``.
* ``dashboard-link-order``.

The config options ``*-link-order`` accept a ``YAML`` list of linked text that defines the link order at the top of that category. 
Any link not specified in this config is included at the end in alphabetical order. 
For example, setting the following:

.. code-block:: bash

   juju config kubeflow-dashboard documentation-link-order='["My Awesomer Tutorial", "My Awesome Tutorial"]'

Would change the previous dashboard to:

.. image:: https://assets.ubuntu.com/v1/b737d26f-link%20config3.png



