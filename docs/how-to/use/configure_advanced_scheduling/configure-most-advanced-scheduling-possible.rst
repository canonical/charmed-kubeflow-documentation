.. _configure_most_advanced_scheduling_possible:

Configure workloads for the most advanced node-pool scheduling possible
=======================================================================

The following guide details how to schedule workloads to available, preconfigured node pools as selectively as possible.

.. warning::

  The following guide applies only if Charmed Kubeflow was installed with the additional precautions detailed in :ref:`Install allowing for advanced node-pool scheduling <install_allowing_for_advanced_node_pool_scheduling>`. Instead, refer to :ref:`Configure workloads for general advanced node-pool scheduling <configure_general_advanced_scheduling>` for alternative, more general scheduling guidelines.

------------
Requirements
------------

Having followed :ref:`Install allowing for advanced node-pool scheduling <install_allowing_for_advanced_node_pool_scheduling>`.

---------
Procedure
---------

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Deploy workloads to their respective Profiles' default node pools
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

No further action is required, since user workloads will already be injected with affinities — and tolerations, when segretating Juju-system components — for their Profile's default node pool.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Deploy workloads to some other, arbitrary node pools
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Ensure the specific workloads are defined with:

- The (set of) label(s) configured for disabling `namespace-node-affinity-operator` for the Profile — for instance, following the example set in `Install allowing for advanced node-pool scheduling <install_allowing_for_advanced_node_pool_scheduling>`, the label `exclude-me-from-namespace-node-affinity-operator=”true”`

- Node affinity (of type `requiredDuringSchedulingIgnoredDuringExecution` and not `preferredDuringSchedulingIgnoredDuringExecution`) matching the label of the target node pool

- Tolerations matching the taint of the target node pool

minding that Notebooks created programmatically are the only kind of workloads not supporting this — as described in `Install allowing for advanced node-pool scheduling <install_allowing_for_advanced_node_pool_scheduling>`.

Refer to :ref:`Configure workloads for general advanced node-pool scheduling <configure_general_advanced_scheduling>` for further details about configuring some of the points above.
