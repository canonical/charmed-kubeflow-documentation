.. _serve_triton:

Serve a model using Triton Inference Server
===========================================

This guide describes how to serve a `BERT <https://www.nvidia.com/en-us/glossary/bert/>`_ model using 
`NVIDIA Triton Inference Server <https://developer.nvidia.com/triton-inference-server>`_.

--------------------------------------
Refresh the ``knative-serving`` charm
--------------------------------------

Upgrade the ``knative-serving`` charm to channel ``latest/edge``:

.. code-block:: bash

   juju refresh knative-serving --channel=latest/edge

Wait until the charm is in ``active`` status, you can check its status with:

.. code-block:: bash

   juju status --watch 5s

---------------------
Create a notebook
---------------------

Create a :ref:`Kubeflow notebook <kubeflow_notebooks>` to be used as your workspace. 
Leave the default notebook image, since you will only use the Command Line Interface (CLI) for running commands.

.. note::

   Running commands in this guide requires in-cluster communication, meaning instructions only work within the notebook environment.

Connect to the notebook, and start a new terminal from the launcher:

.. image:: https://assets.ubuntu.com/v1/bb2dac9a-notebook-screenshot.png

Use this terminal session to run the commands in the following sections.

----------------------------
Create the Inference Service
----------------------------

Define a new Inference Service ``YAML`` file for the BERT model as follows:

.. code-block:: bash

    cat <<EOF > "./isvc.yaml"
    apiVersion: "serving.kserve.io/v1beta1"
    kind: "InferenceService"
    metadata:
        name: "bert-v2"
        annotations:
        "sidecar.istio.io/inject": "false"
    spec:
        transformer:
        containers:
            - name: kserve-container      
            image: kfserving/bert-transformer-v2:latest
            command:
                - "python"
                - "-m"
                - "bert_transformer_v2"
            env:
                - name: STORAGE_URI
                value: "gs://kfserving-examples/models/triton/bert-transformer"
        predictor:
        triton:
            runtimeVersion: 20.10-py3
            resources:
            limits:
                cpu: "1"
                memory: 8Gi
            requests:
                cpu: "1"
                memory: 8Gi
            storageUri: "gs://kfserving-examples/models/triton/bert"
    EOF

.. note::

   In the ISVC ``YAML`` file, make sure to add the annotation ``"sidecar.istio.io/inject": "false"``.
   Due to issue `GH 216 <https://github.com/canonical/kserve-operators/issues/216>`_, you will not be able to reach the ISVC without disabling istio sidecar injection.

---------------------
Schedule GPUs
---------------------

For running on GPUs, specify the GPU resources in the ISVC YAML file. For example, you can run the predictor on NVIDIA GPUs as follows:

.. code-block:: bash

    cat <<EOF > "./isvc-gpu.yaml"
    apiVersion: "serving.kserve.io/v1beta1"
    kind: "InferenceService"
    metadata:
        name: "bert-v2"
    spec:
        transformer:
        containers:
            - name: kserve-container      
            image: kfserving/bert-transformer-v2:latest
            command:
                - "python"
                - "-m"
                - "bert_transformer_v2"
            env:
                - name: STORAGE_URI
                value: "gs://kfserving-examples/models/triton/bert-transformer"
        predictor:
        triton:
            runtimeVersion: 20.10-py3
            resources:
            limits:
                nvidia.com/gpu: 1
            requests:
                nvidia.com/gpu: 1
            storageUri: "gs://kfserving-examples/models/triton/bert"
    EOF

See `Schedule GPUs <https://kubernetes.io/docs/tasks/manage-gpus/scheduling-gpus/>`_ for more details.

Now you need to modify the ISVC ``YAML`` file to set the node selector, node affinity, or tolerations in the ISVC to match your GPU node.
For instance, this is an ISVC ``YAML`` file with node scheduling attributes:

.. code-block:: bash

    cat <<EOF > "./isvc.yaml"
    apiVersion: "serving.kserve.io/v1beta1"
    kind: "InferenceService"
    metadata:
        name: "bert-v2"
    spec:
        transformer:
        containers:
            - name: kserve-container      
            image: kfserving/bert-transformer-v2:latest
            command:
                - "python"
                - "-m"
                - "bert_transformer_v2"
            env:
                - name: STORAGE_URI
                value: "gs://kfserving-examples/models/triton/bert-transformer"
        predictor:
        nodeSelector:
            myLabel1: "true"
        tolerations:
            - key: "myTaint1"
            operator: "Equal"
            value: "true"
            effect: "NoSchedule"
        triton:
            runtimeVersion: 20.10-py3
            resources:
            limits:
                nvidia.com/gpu: 1
            requests:
                nvidia.com/gpu: 1
            storageUri: "gs://kfserving-examples/models/triton/bert"
    EOF

This example sets ``nodeSelector`` and ``tolerations`` for the ``predictor``. 
Similarly, you can set the ``affinity``.

Now apply the ISVC to your namespace with ``kubectl``:

.. code-block:: bash

   kubectl apply -f ./isvc.yaml -n <namespace>

.. note::

   Since you are using the CLI within a notebook, ``kubectl`` is using the Service Account credentials of the notebook pod.

Wait until the Inference Service is in ``Ready`` state. 
It can take up to few minutes. Check its status with:

.. code-block:: bash

   kubectl get inferenceservice bert-v2 -n <namespace>

You should see an output similar to this:

.. code-block:: bash

   NAME      URL                                           READY   AGE
   bert-v2   http://bert-v2.default.10.64.140.43.nip.io   True    71s

---------------------
Perform inference
---------------------

Get ISVC ``status.address.url``:

.. code-block:: bash

   URL=$(kubectl get inferenceservice bert-v2 -n <namespace> -o jsonpath='{.status.address.url}')

Make a request to this URL:

* Prepare the inference input:

  .. code-block:: bash

     cat <<EOF > "./input.json"
     {
       "instances": [
         "What President is credited with the original notion of putting Americans in space?"
       ]
     }
     EOF

* Make a prediction request:

  .. code-block:: bash

     curl -v -H "Content-Type: application/json" ${URL}/v1/models/bert-v2:predict -d @./input.json

The response contains the prediction output:

.. code-block:: bash

   {"predictions": "John F. Kennedy", "prob": 77.91851169430718}
