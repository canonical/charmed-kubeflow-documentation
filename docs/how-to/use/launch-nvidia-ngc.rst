.. _launch_nvidia_ngc:

Launch NVIDIA NGC notebooks
===========================

This guide describes how you can create `Nvidia NGC <https://catalog.ngc.nvidia.com/containers?filters=platform%7Cpltfm_tensorflow%7CTensorFlow,platform%7Cpltfm_pytorch%7CPyTorch&orderBy=weightPopularDESC&query=>`_ Jupyter Notebooks with Charmed Kubeflow (CKF).

-----------------------
Deploy the dependencies
-----------------------

Deploy the charms required for notebook integration with NGC images:

.. code-block:: bash

   juju deploy ngc-integrator --channel=latest/edge --trust
   juju deploy resource-dispatcher --channel=latest/edge --trust

Add the required relation between the deployed charms:

.. code-block:: bash

   juju relate ngc-integrator:pod-defaults resource-dispatcher:pod-defaults

Wait until the charms are in ``active`` status, which can be monitored with:

.. code-block:: bash

   juju status --watch 5s

--------------------------
Create a notebook with NGC
--------------------------

From the Notebooks User Interface (UI):

1. Select ``New Notebook`` and click on ``Custom Notebook``.
2. Select ``Custom Image`` and type in the full name of the NGC image:

.. image:: https://assets.ubuntu.com/v1/82251793-ngc_notebook1.png

3. Scroll down to the bottom of the page and click on ``Advanced Options``.
4. From the ``Configurations`` dropdown, select ``Enable Nvidia NGC JupyterLab Notebook``:

.. image:: https://assets.ubuntu.com/v1/74fb6a7b-ngc_notebook2.png

Finally, launch the notebook and connect to it.
