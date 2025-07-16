.. _install_eks:

Install on EKS
==============

This guide describes how to install Charmed Kubeflow (CKF) on `AWS Elastic Kubernetes Service <https://aws.amazon.com/eks/>`_ (EKS).

You will spin up an EKS cluster on AWS cloud using the Amazon EKS Command Line Interface (CLI), `eksctl <https://eksctl.io/>`_, on your local machine. 
Then, you will interact with the cluster and deploy CKF using `kubectl <https://kubernetes.io/docs/reference/kubectl/>`_ and `Juju`_.

---------------------
Requirements
---------------------

* Local machine with Ubuntu 22.04 or later.
* An `AWS account <https://docs.aws.amazon.com/accounts/latest/reference/manage-acct-creating.html>`_.
* `eksctl <https://eksctl.io/>`_.
* ``kubectl``.

.. important::

   If you use IAM credentials for eksctl authentication, make sure they meet these `minimum IAM policies <https://eksctl.io/usage/minimum-iam-policies/>`_.

---------------------
Deploy EKS cluster
---------------------

First, clone the following repository containing the YAML file used to create the EKS cluster:

.. code-block:: bash

   git clone https://github.com/canonical/kubeflow-examples.git
   cd kubeflow-examples/eks-cluster-setup

Configure the deployment through the YAML file. The configuration set in the YAML file above provides the minimum requirements for deploying CKF:

- ``region``: the cluster is deployed by default to ``eu-central-1`` zone. Edit ``metadata.region`` and ``availabilityZones`` according to your needs.
- ``ssh key``: edit ``managedNodeGroups[0].ssh.publicKeyName`` with your key pair name to enable SSH access into the new EC2 instances.
- ``instance type``: the cluster is deployed with EC2 instances of type ``t2.2xlarge`` for worker nodes, according to the ``managedNodeGroups[0].instanceType`` field. See `Instance types <https://aws.amazon.com/ec2/instance-types/>`_ for more information.
- ``k8s version``: the cluster uses Kubernetes (K8s) 1.24 by default. See :ref:`Supported versions <supported_kubeflow_versions>` for more details about compatibility between CKF, K8s and Juju.
- ``worker nodes``: the cluster has two worker nodes. Edit ``maxSize`` and ``minSize`` under ``managedNodeGroups[0]`` according to your needs.
- ``volume size``: each worker node has gp2/gp3 disk of size 100 Gb. Edit ``managedNodeGroups[0].volumeSize`` for a different configuration.

You can now deploy the cluster as follows:

.. code-block:: bash

   eksctl create cluster -f cluster.yaml

.. note::

   The deployment may take up to 20 minutes.

.. warning::

   The deployment incurs charges for every hour the cluster is running.

-----------------------------
Verify access to the cluster
-----------------------------

Check the access to the cluster as follows:

.. code-block:: bash

   kubectl get nodes

.. note::

   See `Creating a kubeconfig file <https://docs.aws.amazon.com/eks/latest/userguide/create-kubeconfig.html>`_ in case the command above does not return the expected node output.

---------------------
Set up Juju
---------------------

1. Install Juju:

.. code-block:: bash

   sudo snap install juju --channel=3.4/stable

2. Add your EKS cluster as a cloud to Juju:

.. code-block:: bash

   /snap/juju/current/bin/juju add-k8s eks --client

3. Bootstrap a Juju controller:

.. code-block:: bash

   /snap/juju/current/bin/juju bootstrap eks eks-controller

.. note::
   :class: caution

   The command ``/snap/juju/current/bin/juju`` is currently used as a workaround for a `bug <https://bugs.launchpad.net/juju/+bug/2007575>`_.

See `Get started with Juju <https://juju.is/docs/juju/tutorial>`_ for more details.

---------------------
Deploy CKF
---------------------

To deploy CKF and access its dashboard, follow the steps provided in the :ref:`general installation <general_installation>` guide from creating the ``kubeflow`` model section.

---------------------
Clean up resources
---------------------

See `Delete a cluster <https://docs.aws.amazon.com/eks/latest/userguide/delete-cluster.html>`_ for information about removing the EKS cluster and related resources.

.. warning::

   The procedure above does not always delete the volumes that have been created during the cluster deployment. In that case, you can `delete them manually <https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-deleting-volume.html>`_.

To clean up Juju resources, run the following commands:

.. code-block:: bash

   juju unregister eks-controller
   juju remove-cloud eks --client
