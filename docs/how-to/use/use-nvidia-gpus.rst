.. _use_nvidia_gpus:

Use NVIDIA GPUs
===============

This guide describes how to leverage NVIDIA GPU resources in your Charmed Kubeflow (CKF) deployment.

---------------------
Requirements
---------------------

* A CKF deployment and access to the Kubeflow dashboard. See :ref:`Get started <get_started>` for more details.
* An NVIDIA GPU accessible from the Kubernetes cluster that CKF is deployed on. Depending on your deployment, refer to one of the following guides for more details:
  
  * `MicroK8s add-ons <https://microk8s.io/docs/addon-gpu>`_
  * `AKS <https://learn.microsoft.com/en-us/azure/aks/gpu-cluster?tabs=add-ubuntu-gpu-node-pool>`_
  * `EKS <https://docs.aws.amazon.com/batch/latest/userguide/create-gpu-cluster-eks.html>`_

------------------------
Spin a Notebook on a GPU
------------------------

`Kubeflow Notebooks <https://www.kubeflow.org/docs/components/notebooks/overview/>`_ can use any `GPU resource <https://kubernetes.io/docs/tasks/manage-gpus/scheduling-gpus/>`_ available in the Kubernetes cluster. 
This is configurable during the Notebook's creation.

When `creating a Notebook <https://www.kubeflow.org/docs/components/notebooks/quickstart-guide/>`_, under GPUs, 
select the number of GPUs and ``NVIDIA`` as the GPU vendor. 
The GPUs number depends both on the cluster setup and your code demands.

If your Notebook uses a `Tensorflow <https://www.tensorflow.org/>`\-based image with `CUDA <https://developer.nvidia.com/cuda-toolkit>`, 
use the following code to confirm the notebooks have access to a GPU:

.. code-block:: python

   import tensorflow as tf
   gpus = tf.config.list_physical_devices("GPU")
   print(f"Congratz! The following GPUs are available to the notebook: {gpus}" if gpus else "There's no GPU available to the notebook")

.. note::

   In case your cluster setup uses `Taints and Tolerations <https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/>`_, see :ref:`Add Tolerations <add_tolerations>` for more details.

---------------------------
Run Pipeline steps on a GPU
---------------------------

`Kubeflow Pipelines <https://www.kubeflow.org/docs/components/pipelines/concepts/pipeline/>`_ provides `steps <https://www.kubeflow.org/docs/components/pipelines/concepts/step/>`_ to use GPU resources available in your Kubernetes cluster. 
You can enable this by adding the ``nvidia.com/gpu: 1`` `limit <https://kubernetes.io/docs/tasks/manage-gpus/scheduling-gpus/>`_ to a step during the Pipeline's definition. 
See the detailed steps below.

.. note::

   A GPU can be used by one `Pod <https://kubernetes.io/docs/concepts/workloads/pods/>`_ at a time. 
   Thus, a Pipeline can schedule Pods on a GPU only when available. 
   For advanced GPU sharing practices on Kubernetes, see `NVIDIA Multi-Instance GPU <https://www.nvidia.com/en-us/technologies/multi-instance-gpu/>`.

1. Open a notebook with your Pipeline. If you don't have one, use the following code as an example. It creates a Pipeline with a single `component <https://www.kubeflow.org/docs/components/pipelines/concepts/component/>`_ that checks GPU access:

.. code-block:: python

    # Import required objects
    from kfp import dsl

    @dsl.component(base_image="kubeflownotebookswg/jupyter-tensorflow-cuda:v1.9.0")
    def gpu_check() -> str:
        """Get the list of GPUs and print it. If empty, raise a RuntimeError."""
        import tensorflow as tf
        gpus = tf.config.list_physical_devices("GPU")
        print("GPU list:", gpus)
        if not gpus:
            raise RuntimeError("No GPU has been detected.")
        return str(len(gpus) > 0)

    @dsl.pipeline
    def gpu_check_pipeline() -> str:
        """Create a pipeline that runs code to check access to a GPU."""
        gpu_check_object = gpu_check()
        return gpu_check_object.output

Make sure the `KFP SDK <https://kubeflow-pipelines.readthedocs.io/en/master/>`_ is installed in the Notebook's environment:

.. code-block:: bash

   !pip install "kfp>=2.4,<3.0"

2. Ensure the step of the Pipeline's component ``gpu_check`` runs on a GPU by creating a function ``add_gpu_request(task)`` that uses the SDK's `add_node_selector_constraint() <https://kubeflow-pipelines.readthedocs.io/en/master/source/dsl.html#kfp.dsl.PipelineTask.add_node_selector_constraint>`_ and `set_accelerator_limit() <https://kubeflow-pipelines.readthedocs.io/en/master/source/dsl.html#kfp.dsl.PipelineTask.set_accelerator_limit>`_. This sets the required `step's Pod limit <https://kubernetes.io/docs/tasks/manage-gpus/scheduling-gpus/#using-device-plugins>`_:

.. code-block:: python

    def add_gpu_request(task: dsl.PipelineTask) -> dsl.PipelineTask:
        """Add a request field for a GPU to the container created by the PipelineTask object."""
        return task.add_node_selector_constraint(accelerator="nvidia.com/gpu").set_accelerator_limit(
            limit=1
        )

.. note::

   You can further control where the task's Pod will be scheduled by updating the `nodeSelector <https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#nodeselector>`_ of the Pod.
   You can do this with the `kfp.kubernetes.add_node_selector <https://kfp-kubernetes.readthedocs.io/en/kfp-kubernetes-1.4.0/source/kubernetes.html#kfp.kubernetes.add_node_selector>`_ method to add labels of your node pool that the task's Pod should be scheduled to.

3. Modify the Pipeline definition by calling ``add_gpu_request()`` to the component:

.. code-block:: python

    @dsl.pipeline
    def gpu_check_pipeline() -> str:
        """Create a pipeline that runs code to check access to a GPU."""
        gpu_check_object = add_gpu_request(gpu_check())
        return gpu_check_object.output

4. Submit and run the Pipeline:

.. code-block:: python

    from kfp.client import Client
    client = Client()
    run = client.create_run_from_pipeline_func(
        gpu_check_pipeline,
        experiment_name="Check access to GPU",
        enable_caching=False,
    )

5. Navigate to the output ``Run details``. In its logs, you can see the available GPU devices the step has access to.

-------------------------------------
Inference with a KServe ISVC on a GPU
-------------------------------------

KServe `inference services <https://kserve.github.io/website/master/get_started/first_isvc/>`_ (ISVC) can schedule their `Pods <https://kubernetes.io/docs/concepts/workloads/pods/>`_ on a GPU. 
To ensure the ISVC Pod is using a GPU, add the ``nvidia.com/gpu: 1`` `limit <https://kubernetes.io/docs/tasks/manage-gpus/scheduling-gpus/>`_ to the ISVC's definition.

You can do so by using the `kubectl Command Line Interface (CLI) <https://kubernetes.io/docs/reference/kubectl/>`_ or within a notebook.

~~~~~~~~~~~~~~~~~~~
Using kubectl CLI
~~~~~~~~~~~~~~~~~~~

Using the kubectl CLI, you can enable GPU usage in your ``InferenceService`` `Pod <https://kubernetes.io/docs/concepts/workloads/pods/>`_ by directly modifying its configuration ``YAML`` file. 
For example, the inference service YAML file from `this example <https://kserve.github.io/website/latest/get_started/first_isvc/>`_ would be modified to:

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
            resources:
            limits:
                nvidia.com/gpu: 1

~~~~~~~~~~~~~~~~~~~
Within a notebook
~~~~~~~~~~~~~~~~~~~

.. note::

   A GPU can be used by one `Pod <https://kubernetes.io/docs/concepts/workloads/pods/>`_ at a time. 
   Thus, an ISVC Pod can be scheduled on a GPU only when available. 
   For advanced GPU sharing practices on Kubernetes, see `NVIDIA Multi-Instance GPU <https://www.nvidia.com/en-us/technologies/multi-instance-gpu/>`.

1. Open a notebook with your ``InferenceService``. If you don't have one, use `this notebook <https://github.com/canonical/charmed-kubeflow-uats/blob/main/tests/notebooks/cpu/kserve/kserve-integration.ipynb>`_.

Make sure the `Kserve SDK <https://kserve.github.io/website/master/sdk_docs/sdk_doc/>`_ is installed in the Notebook's environment:

.. code-block:: bash

   !pip install kserve

2. Import `V1ResourceRequirements <https://github.com/kubernetes-client/python/blob/master/kubernetes/docs/V1ResourceRequirements.md>`_ from ``kubernetes.client`` package and add a ``resources`` field in the workload you want to run on a GPU. See the example for reference:

.. code-block:: python

    ISVC_NAME = "sklearn-iris"
    isvc = V1beta1InferenceService(
        api_version=constants.KSERVE_V1BETA1,
        kind=constants.KSERVE_KIND,
        metadata=V1ObjectMeta(
            name=ISVC_NAME,
            annotations={"sidecar.istio.io/inject": "false"},
        ),
        spec=V1beta1InferenceServiceSpec(
            predictor=V1beta1PredictorSpec(
                sklearn=V1beta1SKLearnSpec(
                    resources=V1ResourceRequirements(
                        limits={"nvidia.com/gpu": "1"}
                    ),
                    storage_uri="gs://kfserving-examples/models/sklearn/1.0/model"
                )
            )
        ),
    )

