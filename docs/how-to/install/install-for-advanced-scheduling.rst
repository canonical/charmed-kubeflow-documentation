.. _install_allowing_for_advanced_node_pool_scheduling:

Install allowing for advanced node-pool scheduling
==================================================

.. note::

  This guide can be combined with other installation methods described in :ref:`Install <index_install>`. In such a case, it is recommended to go through both this guide and the chosen installation method first, to then understand how to interleave the steps of this guide to the ones of the chosen installation method, which may also contain its own specificl cluster-setup and deployment instructions.

This guide describes how to set up your K8s cluster and how to install Charmed Kubeflow (CKF) to allow for the most advanced node-pool scheduling possible, so that:

- each Kubeflow Profile has a configurable node pool where respective user workloads will be scheduled to by default

- each user workload can be selectively scheduled to different node pools than the default one of the respective Kubeflow Profile, among the node pools allocated to user workloads

- the following are mutually segregated to different node pools:

  - K8s-control-plane workloads

  - (optionally) Juju-system workloads

  - CKF-platform workloads

  - CKF-user workloads

- (optionally) different CKF-platform workloads are in turn selectively scheduled to different node pools, among the ones allocated to CKF platform workloads

------------
Requirements
------------

Follow the same requirements as in the desired installation method among the other ones listed in :ref:`Install <index_install>`.

---------
Procedure
---------

Take the follwing steps in the same order.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Step 1: label and taint your node pools
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Set up your K8s cluster while labeling and tainting your node pools in one of two ways, either:

* Unsegregated Juju system and pools for general workloads, or
* Segregated Juju system and no pools for general workloads

Given that Juju-system workloads do not support specifying node affinities or tolerations, they will be randomly scheduled to any untainted node pools. Follow the latter approach when you prefer to segregate Juju-system workloads to a specific node pool, and the former when you prefer to use untainted node pools also for general workloads.

.. warning::

  It is assumed that the described node-pool setup, with nodes appropriately labeled and tainted, is either already in place or independently achievable, without expecting the described process to handle the migration and/or rescheduling of user workloads from previous clusters and/or cluster states that may differ in terms of node pools.

++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
Option 1: unsegregated Juju system and pools for general workloads
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

Label and taint node pools this way:

- It is implicitly assumed that, if the K8s cluster is set up with labels and taints for the node pool of the K8s control plane, the workloads of the K8s control plane are already deployed with the respective node affinity and tolerations for their node pool.

- For the node pool(s) of the Kubeflow platform:

  - When not scheduling different CKF-platform workloads to different node pools: add one specific label and one specific taint that do not conflict with any default ones. An example could be `platform=kubeflow` for the label and `platform=kubeflow:NoSchedule` for the taint.

  - When scheduling different CKF-platform workloads to different node pools: add one specific, different label for each such node pool and one same, specific taint for all such node pools, with all labels and taints not conflicting with any default ones. An example could be `kubeflow-platform-arch=i` for the label and `platform=kubeflow:NoSchedule` for the taint of a node pool and `kubeflow-platform-arch=j` for the label and `platform=kubeflow:NoSchedule` for the taint of another node pool.

- For each node pool meant to be used as the default one of some Kubeflow Profile(s), add one specific label - different for each node pool - that does not conflict with any default ones. An example could be `kubeflow-default-node-pool=a` for one of the default node pools and `kubeflow-default-node-pool=b` for another default one.

  .. note::

    These labels do not need to correspond to Kubeflow Profiles, but rather (zero, one or more) Kubeflow Profiles will later use these (and possibly the same) labels for the default node affinity of their workloads.

- For each node pool (with special hardware) meant to be used by workloads only when explicitly overriding the default node pool-allocation of their respective Kubeflow Profile(s), add one specific label and one specific taint - different for each node pool - that do not conflict with any default ones. An example could be `special-hardware=x` for the label and `special-hardware=x:NoSchedule` for the taint for one of the special-hardware node pools, and `special-hardware=y` for the label and `special-hardware=y:NoSchedule` for the taint for another special-hardware one.

- Feel free to keep (zero, one or more) node pools unlabeled and untainted, for general use.

.. warning::

  Make sure that all applied taints are of type `NoSchedule`, and not `NoExecute`, in order not to disrupt pre-existing cluster workloads in case of incorrect initial cluster settings, expectations and/or assumptions.

++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
Option 2: segregated Juju system and no pools for general workloads
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

Label and taint node pools as described above for option 1 but with the following changes:

- Keep one and only node pool unlabeled and untainted, meant to be used for Juju-system workloads only and without foreseeing any general workloads ending up in the same pool.

- For each node pool meant to be used as the default one of some Kubeflow Profile(s), add both specific label(s) and specific taint(s).

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Step 2: set up your Juju controller
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

For bootstrapping instructions, see `Get started with Juju <https://documentation.ubuntu.com/juju/latest/tutorial/>`_. No additional, specific precautions are required.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Step 3 (Optional): set up a former `namespace-node-affinity-operator`
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. note::

  This step is required only if it is desirable to have `namespace-node-affinity-operator` itself scheduled to Kubeflow-platform node pools instead of any untainted node pools such as Juju-system ones or (when available) general-workload ones.

Create a temporary Juju model and deploy `namespace-node-affinity-operator` into such a model, using a Juju application name different from the default one of the charm. For example:

  .. code-block:: bash

    juju add-model temp-namespace-node-affinity
    juju switch temp-namespace-node-affinity

    juju deploy --trust --channel 2.2/stable namespace-node-affinity temp-namespace-node-affinity
    juju wait-for application temp-namespace-node-affinity

Then, configure `namespace-node-affinity-operator` to inject workloads scheduled in the (not-yet-created) namespace of the Kubeflow platform with:

- When not scheduling different CKF-platform workloads to different node pools: both node affinity and tolerations, respectively matching the label and the taint of the Kubeflow-platform node pool. An example may be:

  .. code-block:: bash

    namespace_node_affinity_settings=$(cat << EOF
    kubeflow: |
      excludedLabels:
        exclude-me-from-namespace-node-affinity-operator: "true"
      nodeSelectorTerms:
        - matchExpressions:
          - key: platform
            operator: In
            values: [kubeflow]
      tolerations:
        - effect: NoSchedule
          key: platform
          operator: Equal
          value: [kubeflow]
    EOF
    )
    juju config temp-namespace-node-affinity settings_yaml="$namespace_node_affinity_settings"

- When scheduling different CKF-platform workloads to different node pools: only tolerations, matching the taint of the Kubeflow-platform node pool. An example may be:

  .. code-block:: bash

    namespace_node_affinity_settings=$(cat << EOF
    kubeflow: |
      excludedLabels:
        exclude-me-from-namespace-node-affinity-operator: “true”
      tolerations:
        - effect: NoSchedule
          key: platform
          operator: Equal
          value: kubeflow
    EOF
    )
    juju config temp-namespace-node-affinity settings_yaml="$namespace_node_affinity_settings"

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Step 4: create the Kubeflow-platform Juju model
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

No additional, specific precautions required. For instance:

.. code-block:: bash

  juju add-model kubeflow

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Step 5: label the Kubeflow-platform model's namespace(s)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Label with `namespace-node-affinity=enabled` the namespace of the Juju model for the Kubeflow platform. In case Knative is to be deployed, also label the namespaces of Knative, `knative-eventing` and `knative-serving`, in the same way, after manually creating them. For example:

.. code-block:: bash

  kubectl label namespaces kubeflow namespace-node-affinity=enabled

  kubectl create namespace knative-eventing
  kubectl label namespaces knative-eventing namespace-node-affinity=enabled

  kubectl create namespace knative-serving
  kubectl label namespaces knative-serving namespace-node-affinity=enabled

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Step 6: set up a latter `namespace-node-affinity-operator`
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Deploy (another instance of) `namespace-node-affinity-operator` into (a different model,) the model for the Kubeflow platform, using a Juju application name as the charm name or simply different from the name of the `namespace-node-affinity-operator` instance in the former namespace, and configure it with the same configurations as for the former `namespace-node-affinity-operator`. In case Knative is to be deployed, replicate the same configurations of the standard Kubeflow-platform namespace for the namespaces of Knative. An example of such configurations may be:

- When not scheduling different CKF-platform workloads to different node pools:

  .. code-block:: bash

    juju switch kubeflow

    juju deploy --trust --channel 2.2/stable namespace-node-affinity namespace-node-affinity
    juju wait-for application namespace-node-affinity

    namespace_node_affinity_settings=$(cat << EOF
    kubeflow: |
      excludedLabels:
        exclude-me-from-namespace-node-affinity-operator: "true"
      nodeSelectorTerms:
        - matchExpressions:
          - key: platform
            operator: In
            values: [kubeflow]
      tolerations:
        - effect: NoSchedule
          key: platform
          operator: Equal
          value: kubeflow
    knative-eventing: |
      excludedLabels:
        exclude-me-from-namespace-node-affinity-operator: "true"
      nodeSelectorTerms:
        - matchExpressions:
          - key: platform
            operator: In
            values: [kubeflow]
      tolerations:
        - effect: NoSchedule
          key: platform
          operator: Equal
          value: kubeflow
    knative-serving: |
      excludedLabels:
        exclude-me-from-namespace-node-affinity-operator: "true"
      nodeSelectorTerms:
        - matchExpressions:
          - key: platform
            operator: In
            values: [kubeflow]
      tolerations:
        - effect: NoSchedule
          key: platform
          operator: Equal
          value: kubeflow
    EOF
    )
    juju config namespace-node-affinity settings_yaml="$namespace_node_affinity_settings"

- When scheduling different CKF-platform workloads to different node pools:

  .. code-block:: bash

    juju switch kubeflow

    juju deploy --trust --channel 2.2/stable namespace-node-affinity namespace-node-affinity
    juju wait-for application namespace-node-affinity

    namespace_node_affinity_settings=$(cat << EOF
    kubeflow: |
      excludedLabels:
        exclude-me-from-namespace-node-affinity-operator: "true"
      tolerations:
        - effect: NoSchedule
          key: platform
          operator: Equal
          value: kubeflow
    knative-eventing: |
      excludedLabels:
        exclude-me-from-namespace-node-affinity-operator: "true"
      tolerations:
        - effect: NoSchedule
          key: platform
          operator: Equal
          value: kubeflow
    knative-serving: |
      excludedLabels:
        exclude-me-from-namespace-node-affinity-operator: "true"
      tolerations:
        - effect: NoSchedule
          key: platform
          operator: Equal
          value: kubeflow
    EOF
    )
    juju config namespace-node-affinity settings_yaml="$namespace_node_affinity_settings"

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Step 7 (Optional): tear down the former `namespace-node-affinity-operator`
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. note::

  This step is required if and only if step 3 above was followed.

Delete the former `namespace-node-affinity-operator`, that is the one deployed in the temporary Juju model, not the one in the Juju model for the Kubeflow platform. For example:

.. code-block:: bash

  juju switch temp-namespace-node-affinity

  juju remove-application --destroy-storage --no-prompt temp-namespace-node-affinity
  juju destroy-model --destroy-storage --no-prompt temp-namespace-node-affinity

  juju switch kubeflow

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Step 8: deploy CKF
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Deploy CKF following any installation method among :ref:`the supported ones <index_install>` and by also:

- When not scheduling different CKF-platform workloads to different node pools:

  - No additional, specific precautions required.

- When scheduling different CKF-platform workloads to different node pools:

  - Deploying Juju applications with `Juju constraints' tags <https://documentation.ubuntu.com/juju/3.6/reference/constraint/#tags>`__ defining specific node affinities as exemplified `in here <https://discourse.charmhub.io/t/pod-priority-and-affinity-in-juju-charms/4091>`__, one for each application to target the desired node pool (with the respective node label(s)) among the Kubeflow-platform ones. Here is an example: `--constraints="tags=node.kubeflow-platform-arch=j"`.

  - Deploying Juju applications that operate additional platform workloads (in addition to the ones defined in `metadata.yaml`) with charm configurations that add to such workloads node affinities as in the point above (not necessarily the same ones as the respective charms, at the user's own discretion).

.. note::

  Make sure that a charm revision of `kubeflow-profiles` greater than or equal to `839` is deployed, to include the changes required for it to label user Profiles' namespaces to enable `namespace-node-affinity-operator`.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Step 9: (re)configure your Kubeflow Profiles' default node pools
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Among the configurations of `namespace-node-affinity-operator`, add new configuration sections for (not-yet-created) Profiles' namespaces as described for the desired alternative, between the two ones that follow.

Moreover, for each namespace, add configurations to allow for some label(s) to disable the default injection by `namespace-node-affinity-operator`, to also be able to schedule specific workflows to different node pools than the default one — i.e. the “Customization” objective in Abstract. An example of such a label could be `exclude-me-from-namespace-node-affinity-operator=”true”`. Profiles whose namespaces are not configured with exclusion labels will not be able to override the default node-pool scheduling, therefore opting out of such a feature.

.. note::

  CKF admins can continuously reconfigure the default node-pool allocation and whether to enable customization for any Profiles. Nevertheless, Profile workloads deployed before the desired configuration changes are not expected to be rescheduled or migrated. For this reason, it is recommended to configure Profiles before their actual creation (knowing namespace names correspond to Profile names).

++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
Option 1: unsegregated Juju system and pools for general workloads
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

For each Profile's namespace, add configurations to inject workloads with node affinity matching labels of default node pools, to schedule Profiles' workloads to respective default node pools. Profiles whose namespaces are not configured with affinities for default node pools will see their workloads randomly scheduled in any node pools without taints, including not only all the default ones but also any other general ones that may exist, therefore opting out of such a feature.

An example of resulting overall configurations, where both Profiles `profile-i` and `profile-j` have the node pool labeled with `kubeflow-default-node-pool=a` as their default one and Profile `profile-k` has the node pool labeled with `kubeflow-default-node-pool=b` as its default one, may be:

.. code-block:: bash

    namespace_node_affinity_settings=$(cat << EOF
    kubeflow: |
      excludedLabels:
        exclude-me-from-namespace-node-affinity-operator: "true"
      nodeSelectorTerms:
        - matchExpressions:
          - key: platform
            operator: In
            values: [kubeflow]
      tolerations:
        - effect: NoSchedule
          key: platform
          operator: Equal
          value: kubeflow
    knative-eventing: |
      excludedLabels:
        exclude-me-from-namespace-node-affinity-operator: "true"
      nodeSelectorTerms:
        - matchExpressions:
          - key: platform
            operator: In
            values: [kubeflow]
      tolerations:
        - effect: NoSchedule
          key: platform
          operator: Equal
          value: kubeflow
    knative-serving: |
      excludedLabels:
        exclude-me-from-namespace-node-affinity-operator: "true"
      nodeSelectorTerms:
        - matchExpressions:
          - key: platform
            operator: In
            values: [kubeflow]
      tolerations:
        - effect: NoSchedule
          key: platform
          operator: Equal
          value: kubeflow
    profile-i: |
      excludedLabels:
        exclude-me-from-namespace-node-affinity-operator: "true"
      nodeSelectorTerms:
        - matchExpressions:
          - key: kubeflow-default-node-pool
            operator: In
            values:
              - a
    profile-j: |
      excludedLabels:
        exclude-me-from-namespace-node-affinity-operator: "true"
      nodeSelectorTerms:
        - matchExpressions:
          - key: kubeflow-default-node-pool
            operator: In
            values:
              - a
    profile-k: |
      excludedLabels:
        exclude-me-from-namespace-node-affinity-operator: "true"
      nodeSelectorTerms:
        - matchExpressions:
          - key: kubeflow-default-node-pool
            operator: In
            values:
              - b
    EOF
    )
    juju config namespace-node-affinity settings_yaml="$namespace_node_affinity_settings"

++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
Option 2: segregated Juju system and no pools for general workloads
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

For each Profile's namespace, add configurations to inject workloads with node affinity and tolerations respectively matching labels and taints of default node pools, to schedule Profiles' workloads to respective default node pools. Profiles whose namespaces are not configured with affinities and tolerations for default node pools will see their workloads scheduled to the node pool for Juju-system workloads.

An example of resulting overall configurations, where both Profiles `profile-i` and `profile-j` have the node pool labeled with `kubeflow-default-node-pool=a` and tainted with `kubeflow-default-node-pool=a:NoSchedule` as their default one and Profile `profile-k` has the node pool labeled with `kubeflow-default-node-pool=b` and tainted with `kubeflow-default-node-pool=b:NoSchedule` as its default one, may be:

.. code-block:: bash

    namespace_node_affinity_settings=$(cat << EOF
    kubeflow: |
      excludedLabels:
        exclude-me-from-namespace-node-affinity-operator: "true"
      nodeSelectorTerms:
        - matchExpressions:
          - key: platform
            operator: In
            values: [kubeflow]
      tolerations:
        - effect: NoSchedule
          key: platform
          operator: Equal
          value: kubeflow
    knative-eventing: |
      excludedLabels:
        exclude-me-from-namespace-node-affinity-operator: "true"
      nodeSelectorTerms:
        - matchExpressions:
          - key: platform
            operator: In
            values: [kubeflow]
      tolerations:
        - effect: NoSchedule
          key: platform
          operator: Equal
          value: kubeflow
    knative-serving: |
      excludedLabels:
        exclude-me-from-namespace-node-affinity-operator: "true"
      nodeSelectorTerms:
        - matchExpressions:
          - key: platform
            operator: In
            values: [kubeflow]
      tolerations:
        - effect: NoSchedule
          key: platform
          operator: Equal
          value: kubeflow
    profile-i: |
      excludedLabels:
        exclude-me-from-namespace-node-affinity-operator: "true"
      nodeSelectorTerms:
        - matchExpressions:
          - key: kubeflow-default-node-pool
            operator: In
            values:
              - a
      tolerations:
        - effect: NoSchedule
          key: kubeflow-default-node-pool
          operator: Equal
          value: a
    profile-j: |
      excludedLabels:
        exclude-me-from-namespace-node-affinity-operator: "true"
      nodeSelectorTerms:
        - matchExpressions:
          - key: kubeflow-default-node-pool
            operator: In
            values:
              - a
      tolerations:
        - effect: NoSchedule
          key: kubeflow-default-node-pool
          operator: Equal
          value: a
    profile-k: |
      excludedLabels:
        exclude-me-from-namespace-node-affinity-operator: "true"
      nodeSelectorTerms:
        - matchExpressions:
          - key: kubeflow-default-node-pool
            operator: In
            values:
              - b
      tolerations:
        - effect: NoSchedule
          key: kubeflow-default-node-pool
          operator: Equal
          value: b
    EOF
    )
    juju config namespace-node-affinity settings_yaml="$namespace_node_affinity_settings"

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Step 10: create some (other) Kubeflow Profiles
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

No additional, specific precautions required.

See `how to create a Profile <https://www.kubeflow.org/docs/components/central-dash/profiles/#create-a-profile>`_ for general instructions.

For instance, coherently with the examples used above, such Profiles could be created this way:

.. code-block:: bash

  for profile_name in profile-i profile-j profile-k;
  do
  kubectl apply -f - << EOF
  apiVersion: kubeflow.org/v1
  kind: Profile
  metadata:
    name: $profile_name
  spec:
    owner:
      kind: User
      name: admin@example.com
  EOF
  done

-----------
What's next
-----------

You can finally deploy some (other) Profiles' workloads by following :ref:`configure workloads for the most advanced node-pool scheduling possible <configure_advanced_scheduling#configure-workloads-for-the-most-advanced-node-pool-scheduling-possible>`.
