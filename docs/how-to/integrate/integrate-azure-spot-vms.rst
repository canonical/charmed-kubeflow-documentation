:tocdepth: 2

.. _integrate_azure_spot_vms:

Integrate with Azure spot virtual machines
==========================================

This guide describes how to use Azure Kubernetes Service (AKS) `Spot Virtual Machines (VMs) <https://learn.microsoft.com/en-us/azure/virtual-machines/spot-vms>`_ with Charmed Kubeflow (CKF). 
Spot virtual machines are an easy way to access extra computing on demand that can be leveraged for specific Machine Learning (ML) training.

You should use spot VMs for your workflows that are not time-sensitive and short tests and experiments where stability is less important than cost.
For example, data processing, distributed training and hyperparameter tuning, model training and batch inference.

It is not recommended to use spot VMs for the Kubernetes control plane, notebooks and dashboards, databases or datastores like Minio, and model serving for online inference.

.. note::

   Setting up nodes is intended for system admins. Configuring and running workloads is intended for end users.

---------------------
Requirements
---------------------

- CKF deployed on an AKS cluster. See :ref:`Install on AKS <install_aks>` for more details.

.. note::

   `GPU-accelerated <https://learn.microsoft.com/en-us/azure/virtual-machines/sizes/overview?tabs=breakdownseries%2Cgeneralsizelist%2Ccomputesizelist%2Cmemorysizelist%2Cstoragesizelist%2Cgpusizelist%2Cfpgasizelist%2Chpcsizelist#gpu-accelerated>`_ VMs are only available in certain regions which may differ from your chosen AKS cluster region.

----------------------------------
Add an Azure spot node pool to AKS
----------------------------------

1. Go to the Charmed Kubeflow AKS cluster overview page.
2. Click on ``Node pools`` under the ``Settings`` dropdown on the left-hand sidebar.
3. Click on ``Add node pool``.
4. In the next section, make sure to check the ``Enable Azure Spot instances`` check box.
5. Configure the Azure spot virtual machine by clicking on ``Configure…``.

-----------------------------------------
Configure your Azure spot virtual machine
-----------------------------------------

~~~~~~~~~~~~~~~~~~~~~~~~~~~
Eviction type and policy
~~~~~~~~~~~~~~~~~~~~~~~~~~~

You can specify when your VMs are evicted and how. 
If your workload has a maximum price over which it is not worth running it, you can define it in this section. 
See `Understand eviction <https://learn.microsoft.com/en-us/azure/architecture/guide/spot/spot-eviction#understand-eviction>`_ for more details.

~~~~~~~~~~~~~~~~~~~~~
VMs size and pricing
~~~~~~~~~~~~~~~~~~~~~

You can define the size and pricing of your VMs based on your workloads.

~~~~~~~~~~~~~~~~~~~
Labels
~~~~~~~~~~~~~~~~~~~

Labels identify the VM(s) that belong to a certain pool. It is recommended to add at least one label to better identify your VMs later on.

Click on ``Add`` on the bottom left side of the page once your spot node pool is configured.

--------------------------------------
Run workflows in spot virtual machines
--------------------------------------

To schedule workloads, you have to `assign their associated Pod(s) to the desired VM(s) <https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/>`_.

To do so, you have to configure your workloads. 
This configuration depends entirely on the workload type and the component that runs it. See the following examples.

.. note::

   Follow `Azure's suggestions <https://learn.microsoft.com/en-us/azure/aks/spot-node-pool#schedule-a-pod-to-run-on-the-spot-node>`_ for adding tolerations and affinities using the examples below.

~~~~~~~~~~~~~~~~~~~
Pipelines
~~~~~~~~~~~~~~~~~~~

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Using the ``kfp`` v2 SDK
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

You can use the ``kfp`` v2 SDK to add a `node selector constraint <https://kubeflow-pipelines.readthedocs.io/en/stable/source/dsl.html?h=node_selector#kfp.dsl.PipelineTask.add_node_selector_constraint>`_. 
To do so, use the following configuration:

.. code-block:: python

   import kfp
   from kfp import dsl

   def gpu_p100_op():
       return dsl.ContainerOp(
           name='check_p100',
           image='tensorflow/tensorflow:latest-gpu',
           command=['sh', '-c'],
           arguments=['nvidia-smi']
       ).add_node_selector_constraint('cloud.my-cloud.com/gpu-accelerator', 'accelerator-name').container.set_gpu_limit(1)

~~~~~~~~~~~~~~~~~~~
Training jobs
~~~~~~~~~~~~~~~~~~~

^^^^^^^^^^^^^^^^^^^
When using YAMLs
^^^^^^^^^^^^^^^^^^^

You can modify training job ``yaml`` representations to have node selectors or affinity. 
See `training jobs <https://www.kubeflow.org/docs/components/training/user-guides/>`_ for more details.

A training job usually has different processes, such as the “Chief”, “Worker”, or “Parameter Server”. 
Each of them has their own ``spec``, where node affinity or node selectors can be defined, provided that a spot virtual machine is labelled.

For example, you can define a ``TFJob yaml`` representation as follows:

.. code-block:: yaml

   apiVersion: kubeflow.org/v1
   kind: TFJob
   metadata:
     generateName: tfjob
     namespace: your-user-namespace
   spec:
     tfReplicaSpecs:
       PS:
         ...
         spec:
         ...
       Worker:
         spec:
         ...

^^^^^^^^^^^^^^^^^^^^^^^
Using ``nodeAffinity``
^^^^^^^^^^^^^^^^^^^^^^^

To add ``nodeAffinity`` to a training job you can edit the specific process or processes' spec field ensuring to match ``matchExpressions`` with the spot node pool:

.. code-block:: yaml

   Worker:
     spec:
       affinity:
         nodeAffinity:
           requiredDuringSchedulingIgnoredDuringExecution:
             nodeSelectorTerms:
             - matchExpressions:
               - key: a-custom-key.io/my-key
                 operator: In
                 values:
                 - a-custom-value

^^^^^^^^^^^^^^^^^^^^^^^
Using ``nodeSelector``
^^^^^^^^^^^^^^^^^^^^^^^

You can add a ``nodeSelector`` to ensure specific processes are scheduled in the desired node pool as follows:

.. code-block:: yaml

   Worker:
     spec:
       containers:
       - name: my-training-process
         image: training:0.1
       nodeSelector:
         a-custom-key.io/my-key: a-custom-value

^^^^^^^^^^^^^^^^^^^
When using the SDK
^^^^^^^^^^^^^^^^^^^

When using the ``kubeflow.training`` SDK in a Notebook, for example, the training job can be edited similar to how the pure ``yaml`` would be. 
That's because the training jobs use the `Python Kubernetes Client <https://github.com/kubernetes-client/python/tree/master>`_ to define each component.

Creating a TFJob requires you to do the following:

.. code-block:: python

   container = V1Container(...)
   worker = V1ReplicaSpec(..., spec=V1PodSpec(containers=container))
   tfjob = KubeflowOrgV1TFJob(
       api_version="kubeflow.org/v1",
       kind="TFJob",
       metadata=V1ObjectMeta(name="mnist", namespace=namespace),
       spec=KubeflowOrgV1TFJobSpec(
           clean_pod_policy="None",
           tf_replica_specs={"Worker": worker}
       )
   )

Based on the above, to schedule the ``worker`` process in a spot virtual machine, the `v1PodSpec <https://github.com/kubernetes-client/python/blob/master/kubernetes/docs/V1PodSpec.md>`_ object has to have either a ``nodeAffinity`` or ``nodeSelector``. 
For example:

.. code-block:: python

   worker = V1ReplicaSpec(
       spec=V1PodSpec(containers=container),
       affinity=V1Affinity(...)
   )

~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Katib hyperparameter tuning
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

^^^^^^^^^^^^^^^^^^^
When using YAMLs
^^^^^^^^^^^^^^^^^^^

You can schedule a Katib hyperparameter tuning experiment to a specific VM by adding ``nodeAffinity`` or ``nodeSelector``.

These fields should be added to the ``yaml`` representation of the experiment, at the ``trialSpec`` level.

^^^^^^^^^^^^^^^^^^^^^^
Using ``nodeAffinity``
^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: yaml

   trialSpec:
     apiVersion: batch/v1
     kind: Job
     spec:
       template:
         spec:
           containers:
           - name: training-container
             image: image-name
             command:
             - "a-command"
           affinity:
             nodeAffinity:
               requiredDuringSchedulingIgnoredDuringExecution:
                 nodeSelectorTerms:
                 - matchExpressions:
                   - key: a-custom-key.io/my-key
                     operator: In
                     values:
                     - a-custom-value
           restartPolicy: Never

^^^^^^^^^^^^^^^^^^^^^^
Using ``nodeSelector``
^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: yaml

   trialSpec:
     apiVersion: batch/v1
     kind: Job
     spec:
       containers:
       - name: my-training-process
         image: training:0.1
       nodeSelector:
         a-custom-key.io/my-key: a-custom-value

See `Trial Templates <https://www.kubeflow.org/docs/components/katib/user-guides/trial-template/>`_ for more information.

^^^^^^^^^^^^^^^^^^^
When using the SDK
^^^^^^^^^^^^^^^^^^^

When using the ``kubeflow.katib`` SDK in a Notebook, for example, the experiment can be edited similar to how the pure ``yaml`` would be. 
That's because the trial worker spec is usually defined as a JSON template of a Kubernetes ``Job``:

.. code-block:: python

   trial_spec={
       "apiVersion": "batch/v1",
       "kind": "Job",
       "spec": {
           "template": {
               "metadata": {
                   "annotations": {
                       "sidecar.istio.io/inject": "false"
                   }
               },
               "spec": {
                   "containers": [...],
                   "restartPolicy": "Never"
               }
           }
       }
   }

This is where ``nodeAffinity`` or a ``nodeSelector`` could be added to ensure this workload is scheduled in the desired spot virtual machine.

------------------------------------------------------------------
Best practices for evicting workloads from a spot virtual machine
------------------------------------------------------------------

These are a few recommendations for handling workloads eviction from spot VMs:

1. Set `terminationGracePeriodSeconds <https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle#pod-termination>`_ to allow the workloads to terminate gracefully. 
For instance, if you have access to the ``yaml`` representation of a workload, you can do the following:

.. code-block:: yaml

   spec:
     nodeSelector:
       my-key.io/key: "value"
     terminationGracePeriodSeconds: 25

2. Configure the processes to finalise gracefully by modifying the code. 
For example, the ``kfp`` SDK offers the `set_retry() <https://kubeflow-pipelines.readthedocs.io/en/stable/source/dsl.html?h=set_retry#kfp.dsl.PipelineTask.set_retry>`_ method for setting retries.
