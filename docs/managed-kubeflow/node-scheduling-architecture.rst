:orphan:

.. _node_scheduling_architecture:

Node scheduling architecture
============================

This guide provides an overview of the different `Node pools <https://learn.microsoft.com/en-us/azure/aks/create-node-pools>`_, and their configurations, 
created with :ref:`Managed Kubeflow on Azure <index_managed_kubeflow>` deployments.

---------------------
System node pool
---------------------

By default, a system node pool is created for the Managed Kubeflow installation. 
This node pool hosts all the `Pods <https://kubernetes.io/docs/concepts/workloads/pods/>`_ required for the core control plane of Charmed Kubeflow (CKF).
These nodes, along with their charmed workloads, are managed by Canonical, ensuring CKF control plane is always up and running.

While a default recommended size is preconfigured, you can still configure Virtual Machine (VM) sizes using the ``Management instance size`` section.

.. note::

   The system node pool does not include any `Taints <https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/>`_. 
   The control-plane charms allow ``NodeAffinities`` to be scheduled on those nodes, but other (user) workloads are deployed in the system node pool by default.

---------------------
Workload node pools
---------------------

Workload node pools, also known as worker pools, are deployed alongside the system node pool. 
They are used to schedule workload Pods, such as Notebooks, Pipeline steps, Inference Services, and Katib Jobs.

Every worker pool provides Taints, allowing workload Pods that are explicitly configured to be scheduled on them.

.. note::

   If a workload Pod is not configured with the appropriate Tolerations and Affinities, it could be scheduled on the system node pool.

~~~~~~~~~~~~~~~~~~~~~~~~~
List workload node pools
~~~~~~~~~~~~~~~~~~~~~~~~~

You can list the available workload node pools using the Notebooks User Interface (UI). 
To do so, follow these steps:

1. Navigate to ``Notebooks`` using the left sidebar of the Kubeflow dashboard.
2. Click on ``+ New Notebook``.
3. Scroll down.
4. Click on ``Advanced Options > Affinity / Tolerations``.

There, the list of Affinities provides an entry for every viable node group in the cluster.

.. note::

   All workload node pools include a `Taint <https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/>`_ with ``key: sku`` and ``value: <name-of-pool>``.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Schedule workload node pools
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Workload node pools are configured with default Taints, meaning any Pods intended for these pools must specify matching Tolerations.

By default, Notebooks are automatically configured with Affinities and Tolerations, targeting the first created workload node pool, 
ensuring Notebook Pods are scheduled correctly.
However, workloads initiated from Notebooks must explicitly define appropriate Affinities and Tolerations. 
Without these settings, such workloads will default to the system node pool, which is intentionally resource-constrained. 
As a result, workloads without the correct configuration may remain in a pending state indefinitely.

.. image:: https://lh7-rt.googleusercontent.com/docsz/AD_4nXc86CuLTHkqAXLXd6uJkZVCw1vYQSJP7svM7pEcLZwRHGjgqMVnYLReuTTOQR3YgiDGTYREOASZ_jBOFVpWziLumjZ_hOqeow9U-WZPZUyHFDRYTMu3GUzPNMX-EIEP8k5dshX4Nw?key=VDlSMFpcrq3z3bKXaVGgJg

For more details on how workload Pods can be configured, see the following guides:

1. :ref:`Schedule Pods on GPU nodes <use_nvidia_gpus>`.
2. :ref:`Scheduling patterns in Charmed Kubeflow <workload_scheduling_patterns>`.
3. :ref:`Configure scheduling for Kubeflow workloads <configure_advanced_scheduling>`.
