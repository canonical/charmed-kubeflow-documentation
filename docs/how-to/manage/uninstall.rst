.. _uninstall:

Uninstall
=========

This guide describes how to uninstall Charmed Kubeflow (CKF).

---------------------
Remove Kubeflow model
---------------------

Remove the Juju model containing your Kubeflow deployment as follows:

.. code-block:: bash

   juju destroy-model kubeflow --destroy-storage

This will remove all the applications under that Kubernetes namespace, and then remove the namespace itself.

If needed, you can force the deletion of the model as follows:

.. code-block:: bash

   juju destroy-model kubeflow -no-prompt --destroy-storage --force

Alternatively, you can also simply release storage instead of deleting the model by using the following flag:

.. code-block:: bash

   juju destroy-model kubeflow --release-storage

----------------------
Remove Juju controller
----------------------

You may want to remove the Juju controller to free up resources. 
To do so, run the following command including the controller name, ``my-controller`` in this example:

.. code-block:: bash

   juju destroy-controller my-controller
