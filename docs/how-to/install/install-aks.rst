.. _install_aks:

Install on AKS
==============

This guide describes how to install Charmed Kubeflow (CKF) on `Azure Kubernetes Service <https://azure.microsoft.com/en-us/products/kubernetes-service>`_ (AKS).

You will spin up an AKS cluster using the Azure CLI (Command Line Interface) on your local machine. 
Then, you will interact with the cluster and deploy CKF using `kubectl <https://kubernetes.io/docs/reference/kubectl/>`_ and `Juju`_.

---------------------
Requirements
---------------------

* Local machine with Ubuntu 22.04 or later.
* An `Azure account <https://azure.microsoft.com/en-us/free>`_.
* `Azure CLI <https://learn.microsoft.com/en-us/cli/azure/install-azure-cli-linux?pivots=apt>`_.
* ``kubectl``. 

.. important:: Make sure the Azure CLI has the following configuration:  
    
   - `Authentication permissions required <https://learn.microsoft.com/en-us/azure/aks/concepts-identity#azure-rbac-to-authorize-access-to-the-aks-resource>`_ for AKS.
   - ``Managed Application Contributor Role`` added.

---------------------
Deploy AKS cluster
---------------------

.. note::
    The deployment incurs charges for every hour the cluster is running.

First, create a resource group to deploy the cluster:

.. code-block:: bash

   az group create --name myResourceGroup --location westeurope

Regarding location, choose whichever suits best your needs. 
You can list all locations available using ``az account list-locations -o table``.

Now, spin up the cluster. 
The configuration below provides the minimum requirements for deploying CKF. 
For further customization, see `the full list of available parameters <https://learn.microsoft.com/en-us/cli/azure/aks?view=azure-cli-latest#az-aks-create>`_:

.. code-block:: bash

   az aks create \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --kubernetes-version 1.29 \
    --node-count 2 \
    --node-vm-size Standard_D8s_v3 \
    --node-osdisk-size 100 \
    --node-osdisk-type Managed \
    --os-sku Ubuntu \
    --ssh-key-value <path-to-public-key>

- ``kubernetes-version``: this example uses Kubernetes (K8s) ``1.29``. For using other versions, see :ref:`Supported versions <supported_kubeflow_versions>` for compatibility with K8s and Juju.
- ``node-count``: this cluster has two worker nodes, given that the cluster autoscaler option is disabled by default. You can also enable it and define instead ``max-count`` and ``min-count``.
- ``node-vm-size``: the cluster is deployed with Azure VM instances of size ``Standard_D8s_v3`` for worker nodes. See `VM sizes <https://learn.microsoft.com/en-us/azure/aks/hybrid/concepts-support#available-vm-sizes>`_ and `Sizes for cloud services <https://learn.microsoft.com/en-us/azure/cloud-services/cloud-services-sizes-specs>`_ for more details.
- ``node-osdisk-type``: ``Managed`` node disks are used since ``Ephemeral`` ones are better suited when applications are tolerant of individual VM failures, which is not the case for CKF.
- ``ssh-key-value``: public key path or key contents to access individual nodes using SSH. Its default value is ``~\.ssh\id_rsa.pub``.

.. note::

   Spinning up the cluster may take some time to complete.

---------------------------------
Verify access to the cluster
---------------------------------

Check if the AKS cluster has been added to ``kubeconfig`` as follows:

.. code-block:: bash

   kubectl config get-clusters

If you don't see it there, use the following command to add it:

.. code-block:: bash

   az aks get-credentials --resource-group myResourceGroup --name myAKSCluster --admin

.. note::

   You may need to remove ``--admin`` from the command above, depending on the type of ``kubeconfig`` that you have access to.

Now check your access to the cluster as follows:

.. code-block:: bash

   kubectl get nodes

You should expect an output like the following:

.. code-block:: bash

   NAME                                STATUS   ROLES   AGE     VERSION
   aks-nodepool1-18441560-vmss000000   Ready    agent   1m20s   v1.29.4
   aks-nodepool1-40664177-vmss000001   Ready    agent   1m20s   v1.29.4

---------------------
Set up Juju
---------------------

1. Install Juju:

.. code-block:: bash

   sudo snap install juju --channel=3.4/stable

2. Add your AKS cluster as a cloud to Juju:

.. code-block:: bash

   juju add-k8s aks --client

3. Bootstrap a Juju controller:

.. code-block:: bash

   juju bootstrap aks aks-controller

See `Get started with Juju <https://juju.is/docs/juju/tutorial>`_ for more details.

---------------------
Deploy CKF
---------------------

To deploy CKF and access its dashboard, follow the steps provided in the :ref:`general installation <general_installation>` guide from creating the ``kubeflow`` model section.

---------------------
Clean up resources
---------------------

You can delete the AKS cluster and related resources as follows:

.. code-block:: bash

   az aks delete --resource-group myResourceGroup --name myAKSCluster --yes
   az group delete --name myResourceGroup --yes

You can check if a resource group exists using:

.. code-block:: bash

   az group exists --name <resource-group-name>

See `Azure resource manager <https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/delete-resource-group?tabs=azure-powershell>`_ for further information.

To clean up Juju resources, run the following commands:

.. code-block:: bash

   juju unregister aks-controller
   juju remove-cloud aks --client
