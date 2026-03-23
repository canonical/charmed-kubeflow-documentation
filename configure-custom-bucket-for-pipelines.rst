.. _install_allowing_for_advanced_node_pool_scheduling:

Install allowing for advanced node-pool scheduling
==================================================

This guide describes how to set up your K8s cluster and how to install Charmed Kubeflow (CKF) to allow for the most advanced node-pool scheduling possible, so that:
- each Kubeflow Profile has a configirable node pool where respective user worklaods will be scheduled to by default
- each user workload can be selectively scheduled to different node pools than the default one of the respective Kubeflow Profile, among the node pools allocated to user workloads
- the following are mutually segregated to different node pools:
  - K8s-control-plane workloads
  - (optionally) Juju-system workloads
  - CKF-platform workloads
  - CKF-user workloads
- (optionally) different CKF-platform workloads are in turn selectively scheduled to different node pools, among the ones allocated to CKF platform workloads

-----------
Limitations
-----------

TODO

------------
Requirements
------------

TODO

---------
Procedure
---------

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Step 1: label and taint your node pools
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Set up your K8s cluster while labeling and tainting your node pools as described for the desired alternative, between the two that follow.

++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
Alternative 1: not segregating Juju system but keeping pools for general workloads
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

Label and taint node pools this way:

- It is implicitly assumed that, if the K8s cluster is set up with labels and taints for the node pool of the K8s control plane, the workloads of the K8s control plane are already deployed with the respective node affinity and tolerations for their node pool.

- For the node pool(s) of the Kubeflow platform:

  - When not scheduling different CKF-platform workloads to different node pools: add one specific label and one specific taint that do not conflict with any default ones. An example could be `kubeflow-platform=true` for the label and `kubeflow-platform=true:NoSchedule` for the taint.

  - When scheduling different CKF-platform workloads to different node pools: add one specific, different label for each such node pool and one same, specific taint for all such node pools, with all labels and taints not conflicting with any default ones. An example could be `kubeflow-platform-arch=amd64` for the label and `kubeflow-platform=true:NoSchedule` for the taint of a node pool and `kubeflow-platform-arch=arm64` for the label and `kubeflow-platform=true:NoSchedule` for the taint of another node pool.

- For each node pool meant to be used as the default one of some Kubeflow Profile(s), add one specific label - different for each node pool - that does not conflict with any default ones. An example could be `kubeflow-default-node-pool=a` for one of the default node pools and `kubeflow-default-node-pool=b` for another default one.

  .. note::

  These labels do not need to correspond to Kubeflow Profiles, but rather (zero, one or more) Kubeflow Profiles will later use these (and possibly the same) labels for the default node affinity of their workloads.

- For each node pool with special hardware meant to be used by workloads only when explicitly overriding the default node pool-allocation of their respective Kubeflow Profile(s), add one specific label and one specific taint - different for each node pool - that do not conflict with any default ones. An example could be `special-hardware=x` for the label and `special-hardware=x:NoSchedule` for the taint for one of the special-hardware node pools, and `special-hardware=y` for the label and `special-hardware=y:NoSchedule` for the taint for another special-hardware one.

- Feel free to keep (zero, one or more) node pools unlabeled and untainted, for general use.

.. warning::

  Make sure that all applied taints are of type `NoSchedule`, and not `NoExecute`, in order not to disrupt pre-existing cluster workloads in case of incorrect initial cluster settings, expectations and/or assumptions. Do not use `PreferNoSchedule` either, as scheduling guarantees are required.

++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
Alternative 2: segregating Juju system but not keeping pools for general workloads
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

Label and taint node pools as described above for alternative 1 but with the following changes:

- For each node pool meant to be used as the default one of some Kubeflow Profile(s), add not only one specific label but also one specific taint.

- Keep one and only node pool unlabeled and untainted, meant to be used for Juju-system workloads only and without foreseeing any general workloads ending up in the same pool.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Setp 2: set up your Juju controller
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

No additional, specific precautions required.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Step 3: set up a former `namespace-node-affinity-operator`
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

TODO

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Step 4: create the Kubeflow-platform Juju model
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

No additional, specific precautions required.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Step 5: label the Kubeflow-platform model's namespace(s)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Label with `namespace-node-affinity=enabled` the namespace of the Juju model for the Kubeflow platform. In case Knative is to be deployed, also label the namespaces of Knative, `knative-eventing` and `knative-serving`, in the same way after manually creating them.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Step 6: set up a latter `namespace-node-affinity-operator`
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

TODO

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Step 7: tear down the former `namespace-node-affinity-operator`
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Delete the former `namespace-node-affinity-operator`, that is the one deployed in the temporary Juju model, not the one in the Juju model for the Kubeflow platform.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Step 8: deploy Charmed Kubeflow
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

TODO

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Step 9: (re)configure your Kubeflow Profiles' default node pools
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
Alternative 1: not segregating Juju system but keeping pools for general workloads
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

TODO

++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
Alternative 2: segregating Juju system but not keeping pools for general workloads
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

TODO

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Step 10: create some (new) Kubeflow Profiles
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

No additional, specific precautions required.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Step 11: deploy some (new) Profiles' Workloads
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

TODO







~~~~~~~~~~~~~~~~~~~~~~~~
XXXXXXXXXXXXXXXXXXXXXXXX
~~~~~~~~~~~~~~~~~~~~~~~~

Update the `default pipeline root <https://www.kubeflow.org/docs/components/pipelines/concepts/pipeline-root>`__ for pipeline Runs across all Profiles, which includes the schema for, the name of and the path to the bucket, via its ``default_pipeline_root`` `configuration option <https://charmhub.io/kfp-profile-controller/configurations>`__, e.g.:

.. code-block:: bash

  juju config kfp-profile-controller default_pipeline_root=minio://${BUCKET_NAME}/v2/artifacts

.. note::

  In Charmed Kubeflow, Pipelines can only be configured with the MinIO schema.

This automatically creates a ``ConfigMap`` named ``kfp-launcher`` in each Profile

.. note::

  For this configuration change to take effect, existing ``ConfigMaps`` — if any — named ``kfp-launcher`` in the namespace of each Profile of interest have to be manually deleted after the configuration change is applied, so that they are automatically recreated with the updated default pipeline root. Refer to `this issue <https://github.com/canonical/metacontroller-operator/issues/193>`__ for details.
