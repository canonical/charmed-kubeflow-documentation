.. _configure_advanced_scheduling:

Configure workloads for advanced node-pool scheduling
=====================================================

The following guide illustrates how to schedule workflows targeting specific node pools, both in general and when Charmed Kubeflow is set up for the most advanced scheduling capabilities as possible.

.. _configure_general_advanced_scheduling:

-------------------------------------------------------------
Configure workloads for general advanced node-pool scheduling
-------------------------------------------------------------

This guide describes how to configure different Charmed Kubeflow (CKF) workloads, such as Notebooks, Pipeline steps, and distributed jobs, 
to align with specific :ref:`scheduling patterns <workload_scheduling_patterns>` that might be required.

~~~~~~~~~~~~
Requirements
~~~~~~~~~~~~

1. A CKF deployment and access to the Kubeflow dashboard. See :ref:`Get started <get_started>` for more details.
2. An underlying Kubernetes (K8s) cluster with multiple nodes and labels.

~~~~~~~~~
Notebooks
~~~~~~~~~

You can configure Notebooks to be scheduled on specific nodes via the Notebooks page in the Kubeflow dashboard when creating a new Notebook.

.. note::

   Configuring the Notebook creation page is intended only for admins. See :ref:`this guide <configure_notebook_page>` for more details.

To do so, configure the Affinity and Toleration settings during Notebook creation by:

1. Clicking on ``+ Create Notebook``.
2. Scrolling to the bottom and expanding ``Advanced Options``.
3. Configuring the ``Affinity`` and ``Tolerations`` sections.

.. note::

   In case your cluster setup uses `Taints and Tolerations <https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/>`_, see :ref:`Add Tolerations <add_tolerations>` for more details.

~~~~~~~~~~~~~~
Pipeline steps
~~~~~~~~~~~~~~

K8s specific configurations, such as ``nodeSelectors`` and ``Tolerations``, in a Kubeflow pipeline `step <https://www.kubeflow.org/docs/components/pipelines/concepts/step/>`_ can be configured via the `kfp-kubernetes <https://kfp-kubernetes.readthedocs.io/en/kfp-kubernetes-1.4.0/index.html>`_ Python package.
The following example sets both in a pipeline step:

.. code-block:: python

    from kfp.kubernetes import add_node_selector, add_toleration

    @dsl.component(base_image="python:3.12")
    def print_node_name():
        """Print the Node's hostname."""
        import socket

        print("Node name: %s" % socket.gethostname())

    @dsl.pipeline
    def node_scheduling_pipeline():
        print_node_task = print_node_name()
        task = add_node_selector(print_node_task, "sku", "pool-1")
        task = add_toleration(task, key="sku", operator="Exists", effect="NoSchedule")

~~~~~~~~~~~~~~~~~~~~
Distributed training
~~~~~~~~~~~~~~~~~~~~

Distributed training in CKF is achieved via the `Katib <https://v1-9-branch.kubeflow.org/docs/components/katib/overview/>`_ and `Training Operator <https://v1-9-branch.kubeflow.org/docs/components/training/overview/>`_ components.
Katib Trials can be implemented with different job types, which may use default settings defined in `Trial Templates <https://v1-9-branch.kubeflow.org/docs/components/katib/user-guides/trial-template/>`_. 
These can include standard K8s Jobs or `Distributed training jobs <https://v1-9-branch.kubeflow.org/docs/components/training/user-guides/>`_ via the Training Operator.

All Trial definitions ultimately configure a ``PodSpec`` for the Trial's Pods. 
To accommodate the above scheduling use cases, you need to configure the ``nodeSelector`` and Tolerations of the ``PodSpec``.

Below is an example of a ``TFJob`` that can be used in a Trial definition and satisfies all the above criteria:

.. code-block:: yaml

    apiVersion: kubeflow.org/v1
    kind: TFJob
    metadata:
        generateName: tfjob
        namespace: your-user-namespace
    spec:
        tfReplicaSpecs:
        PS:
            replicas: 1
            restartPolicy: OnFailure
            template:
            metadata:
                annotations:
                sidecar.istio.io/inject: "false"
            spec:
                nodeSelector:  # Scheduling
                pool: pool1
                tolerations:   # Scheduling
                - effect: NoSchedule
                    key: sku
                    operator: Equal
                    value: pool1
                containers:
                - name: tensorflow
                    image: gcr.io/your-project/your-image
                    command:
                    - python
                    - -m
                    - trainer.task
                    - --batch_size=32
                    - --training_steps=1000
        Worker:
            replicas: 3
            restartPolicy: OnFailure
            template:
            metadata:
                annotations:
                sidecar.istio.io/inject: "false"
            spec:
                nodeSelector:  # Scheduling
                pool: pool1
                tolerations:   # Scheduling
                - effect: NoSchedule
                    key: sku
                    operator: Equal
                    value: pool1
                containers:
                - name: tensorflow
                    image: gcr.io/your-project/your-image
                    resources:
                    limits:
                        nvidia.com/gpu: 1
                    command:
                    - python
                    - -m
                    - trainer.task
                    - --batch_size=32
                    - --training_steps=1000

~~~~~~~~~~~~~~~~~~~~~~~~
KServe InferenceServices
~~~~~~~~~~~~~~~~~~~~~~~~

KServe ``InferenceServices`` expose PodSpec attributes.
that can be used for configuring advanced scheduling scenarios. 
See the example below for more details:

.. code-block:: yaml

    apiVersion: "serving.kserve.io/v1beta1"
    kind: "InferenceService"
    metadata:
        name: "sklearn-iris"
    spec:
        predictor:
        model:
            modelFormat:
            name: sklearn
            storageUri: "gs://kfserving-examples/models/sklearn/1.0/model"
            nodeSelector:  # Scheduling
            sku: pool-1
            tolerations:   # Scheduling
            - key: "sku"
            operator: "Exists"
            effect: "NoSchedule"

.. _configure_most_advanced_scheduling_possible:

-----------------------------------------------------------------------
Configure workloads for the most advanced node-pool scheduling possible
-----------------------------------------------------------------------

The following guide shows how to schedule workloads to available, preconfigured node pools as selectively as possible.

.. warning::

  While Notebooks created from the Dashboard can rely on PodDefaults to add labels to disable `namespace-node-affinity-operator`, in order to target a different node pool than the Profile's default one, `the API to create Kubeflow Notebooks programmatically does not allow for that <https://www.kubeflow.org/docs/components/notebooks/api-reference/notebook-v1/#kubeflow.org/v1.NotebookTemplateSpec>`__.

.. note::

  This guide does not support migrating or rescheduling Profile workloads. A practical workaround may be to delete and recreate workloads using whichever method best suits your needs.

~~~~~~~~~~~~
Requirements
~~~~~~~~~~~~

Charmed Kubeflow installed using the specific :ref:`Install allowing for advanced node-pool scheduling <install_allowing_for_advanced_node_pool_scheduling>` guide. If Charmed Kubeflow was not installed with such additional precautions, refer to :ref:`configure workloads for general advanced node-pool scheduling <configure_general_advanced_scheduling>` for alternative, more general scheduling guidelines.

~~~~~~~~~
Procedure
~~~~~~~~~

Workloads can either be scheduled to their respective Profiles' default node pools, which is the default behavior, or deployed to other arbitrary node pools.

+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
Deploy workloads to their respective Profiles' default node pools
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

Create workloads without extra precautions, since by default they will already be injected with affinities — and tolerations, when segregating Juju-system components — for their Profile's default node pool.

++++++++++++++++++++++++++++++++++++++++++++++++++++
Deploy workloads to some other, arbitrary node pools
++++++++++++++++++++++++++++++++++++++++++++++++++++

Ensure the specific workloads are defined with:

- The (set of) label(s) configured for disabling `namespace-node-affinity-operator` for the Profile — for instance, following the example set in :ref:`Install allowing for advanced node-pool scheduling <install_allowing_for_advanced_node_pool_scheduling>`, the label `exclude-me-from-namespace-node-affinity-operator=”true”`

- Node affinity (of type `requiredDuringSchedulingIgnoredDuringExecution` and not `preferredDuringSchedulingIgnoredDuringExecution`) matching the label of the target node pool

- Tolerations matching the taint of the target node pool

Refer to :ref:`configure workloads for the most advanced node-pool scheduling possible <configure_most_advanced_scheduling_possible>` for further details about configuring some of the points above.
