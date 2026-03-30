.. _configure_most_advanced_scheduling_possible:

Configure workloads for the most advanced node-pool scheduling possible
=======================================================================

The following guide shows how to schedule workloads to available, preconfigured node pools as selectively as possible.

.. warning::

  While Notebooks created from the Dashboard can rely on PodDefaults to add labels to disable `namespace-node-affinity-operator`, in order to target a different node pool than the Profile's default one, `the API to create Kubeflow Notebooks programmatically does not allow for that <https://www.kubeflow.org/docs/components/notebooks/api-reference/notebook-v1/#kubeflow.org/v1.NotebookTemplateSpec>`__.

.. note::

  This guide does not support migrating or rescheduling Profile workloads. A practical workaround may be to delete and recreate workloads using whichever method best suits your needs.

------------
Requirements
------------

Charmed Kubeflow installed using the specific :ref:`Install allowing for advanced node-pool scheduling <install_allowing_for_advanced_node_pool_scheduling>` guide. If Charmed Kubeflow was not installed with such additional precautions, refer to :ref:`Configure workloads for general advanced node-pool scheduling <configure_general_advanced_scheduling>` for alternative, more general scheduling guidelines.

---------
Procedure
---------

Workloads can either be scheduled to their respective Profiles' default node pools, which is the default behavior, or deployed to other arbitrary node pools.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Deploy workloads to their respective Profiles' default node pools
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Create workloads without extra precautions, since by default they will already be injected with affinities — and tolerations, when segregating Juju-system components — for their Profile's default node pool.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Deploy workloads to some other, arbitrary node pools
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Ensure the specific workloads are defined with:

- The (set of) label(s) configured for disabling `namespace-node-affinity-operator` for the Profile — for instance, following the example set in :ref:`Install allowing for advanced node-pool scheduling <install_allowing_for_advanced_node_pool_scheduling>`, the label `exclude-me-from-namespace-node-affinity-operator=”true”`

- Node affinity (of type `requiredDuringSchedulingIgnoredDuringExecution` and not `preferredDuringSchedulingIgnoredDuringExecution`) matching the label of the target node pool

- Tolerations matching the taint of the target node pool

Refer to :ref:`Configure workloads for general advanced node-pool scheduling <configure_general_advanced_scheduling>` for further details about configuring some of the points above.
