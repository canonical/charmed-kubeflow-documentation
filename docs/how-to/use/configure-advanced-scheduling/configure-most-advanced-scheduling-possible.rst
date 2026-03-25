.. _configure_most_advanced_scheduling_possible:

Configure workloads for the most advanced node-pool scheduling possible
=======================================================================

The following guide details how to schedule workloads to available, preconfigured node pools as selectively as possible.

.. warning::

  The following guide applies only if Charmed Kubeflow was installed with the additional precautions detailed in :ref:`Install allowing for advanced node-pool scheduling <install_allowing_for_advanced_node_pool_scheduling>`. Instead, refer to :ref:`Configure workloads for general advanced node-pool scheduling <configure_general_advanced_scheduling>` for alternative, more general scheduling guidelines.

.. warning::

  This guide does not allow for migration and/or rescheduling of Profile workloads. An effective workaround may be as simple as deleting and recreating workloads in any way suitable to the user.

.. warning::

  Not all user workloads support scheduling to a different node pool than the default one of their respective Profile: creating Kubeflow Notebooks programmatically, instead of using the Kubeflow Dashboard's UI, will not allow to select a different node pool than the Profile's default one. This is due to the fact that the API to create Notebooks programmatically does not allow for labels, while a label is necessary to prevent `namespace-node-affinity-operator` from injecting/overriding the Notebook request to K8s' API server with the default node affinity and tolerations, so that the Notebook could define any custom node affinities and tolerations. Given Notebooks are rarely created programmatically, though (but rather from the Dashboard's UI, preferentially), this limitation is negligible. Notebooks created from the Dashboard can rely on PodDefaults to add labels, instead, with no such problem - and can use the UI to define both node affinity and tolerations, in CKF. All other user workloads (KServe's, Pipelines', Katib's, Training's, Trainer's) fully support the objective “Customization”, as their APIs allow not only for node affinity and tolerations but also for labels.

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

No further action is required, since user workloads will already be injected with affinities — and tolerations, when segregating Juju-system components — for their Profile's default node pool.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Deploy workloads to some other, arbitrary node pools
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Ensure the specific workloads are defined with:

- The (set of) label(s) configured for disabling `namespace-node-affinity-operator` for the Profile — for instance, following the example set in :ref:`Install allowing for advanced node-pool scheduling <install_allowing_for_advanced_node_pool_scheduling>`, the label `exclude-me-from-namespace-node-affinity-operator=”true”`

- Node affinity (of type `requiredDuringSchedulingIgnoredDuringExecution` and not `preferredDuringSchedulingIgnoredDuringExecution`) matching the label of the target node pool

- Tolerations matching the taint of the target node pool

minding that Notebooks created programmatically are the only kind of workloads not supporting this — as described in :ref:`Install allowing for advanced node-pool scheduling <install_allowing_for_advanced_node_pool_scheduling>`.

Refer to :ref:`Configure workloads for general advanced node-pool scheduling <configure_general_advanced_scheduling>` for further details about configuring some of the points above.
