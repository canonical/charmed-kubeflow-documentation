.. _deploy_nvidia_nims:

Deploy NVIDIA NIMs
==================

This guide describes how to deploy `NVIDIA Inference Microservices (NIMs) <https://developer.nvidia.com/nim>`_ on Charmed Kubeflow (CKF) 
and serve a model with `KServe <https://kserve.github.io/website/0.13/>`_, a component of CKF. 
A NIM is a containerized inference microservice for running LLMs, distributed from `NVIDIA container registry (NGC) <https://www.nvidia.com/en-us/gpu-cloud/>`_.

The guide uses Kubeflow v1.9, Kubernetes v1.29, and Juju v3.4. 
See :ref:`Supported versions <supported_kubeflow_versions>` for more details on compatibility among these.

---------------------
Requirements
---------------------

* An active CKF deployment and access to the Kubeflow dashboard. See :ref:`Get started <get_started>` for more details.
* A GPU compatible with the model downloaded from NGC. See `the model details <https://docs.nvidia.com/nim/large-language-models/1.1.0/support-matrix.html#meta-llama-3-8b-instruct>`_ for further information. This guide is tested using an NVIDIA A100 GPU.
* An NVIDIA NGC API key. See `create an NVIDIA account <https://ngc.nvidia.com/signin>`_ and `create an API key <https://org.ngc.nvidia.com/setup/api-key>`_ for more details.

.. note::
   When creating an API key, ensure that ``NGC Catalog`` is selected from the Services Included dropdown.

-----------------------------
Configure MicroK8s GPU add-on
-----------------------------

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Install NVIDIA GPU drivers
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

See `MicroK8s Add-on: gpu documentation <https://canonical.com/microk8s/docs/addon-gpu#p-20002-use-host-nvidia-container-runtime>`_ to install the NVIDIA drivers and verify that drivers are loaded.

.. important::
   This step is necessary due to `this MicroK8s issue <https://github.com/canonical/microk8s-core-addons/issues/303>`_.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Enable MicroK8s GPU add-on
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Enable `MicroK8s GPU add-on <https://canonical.com/microk8s/docs/addon-gpu>`_ to enable running NVIDIA GPU workloads.

.. code-block:: bash

   microk8s enable gpu

Check MicroK8s status as follows:

.. code-block:: bash

   microk8s status --wait-ready

Wait until the output shows microk8s is running and the gpu add-on is listed under enabled.

--------------------------
Create a Kubeflow notebook
--------------------------

Create a :ref:`Kubeflow notebook <kubeflow_notebooks>`. 
This notebook is the workspace from which you run commands.

Choose the default notebook image since you will be only using the Command Line Interface (CLI).

.. note::
   Running commands in this guide requires in-cluster communication and instructions won't work outside of the notebook environment.

Connect to the notebook and start a new terminal from the launcher as shown below:

.. image:: https://assets.ubuntu.com/v1/bb2dac9a-notebook-screenshot.png

Use this terminal session to run the commands in the next sections.

-------------------------
Create Kubernetes secrets
-------------------------

From the terminal, export your NGC API key. 
It will be used in the next steps to create the required `K8s secrets <https://kubernetes.io/docs/concepts/configuration/secret/>`_:

.. code-block:: bash

   export NGC_API_KEY=<your_key>

Create a `Docker config K8s secret <https://kubernetes.io/docs/concepts/configuration/secret/#docker-config-secrets>`_ with the NGC API key to pull the image from the NGC private Docker registry as follows:

.. code-block:: bash

   kubectl create secret docker-registry ngc-secret --docker-server=nvcr.io --docker-username='$oauthtoken' --docker-password=$NGC_API_KEY

Create an `opaque K8s secret <https://kubernetes.io/docs/concepts/configuration/secret/#opaque-secrets>`_ with the NGC API key to launch NIMs:

.. code-block:: bash

   kubectl create secret generic nvidia-nim-secret --from-literal=NGC_API_KEY=$NGC_API_KEY

----------------------
Create Serving Runtime
----------------------

Create the KServe `Serving Runtime <https://kserve.github.io/website/latest/modelserving/servingruntimes/>`_ ``YAML`` to be used as the runtime for the NIMs as follows:

.. code-block:: bash

    cat <<EOF > "./runtime.yaml"
    apiVersion: serving.kserve.io/v1alpha1
    kind: ServingRuntime
    metadata:
        name: nvidia-nim-llama3-8b-instruct-1.0.0
    spec:
        annotations:
        prometheus.kserve.io/path: /metrics
        prometheus.kserve.io/port: "8000"
        serving.kserve.io/enable-metric-aggregation: "true"
        serving.kserve.io/enable-prometheus-scraping: "true"
        containers:
        - env:
        - name: NIM_CACHE_PATH
            value: /tmp
        - name: NGC_API_KEY
            valueFrom:
            secretKeyRef:
                name: nvidia-nim-secret
                key: NGC_API_KEY
        image: nvcr.io/nim/meta/llama3-8b-instruct:1.0.0
        name: kserve-container
        ports:
        - containerPort: 8000
            protocol: TCP
        resources:
            limits:
            cpu: "12"
            memory: 32Gi
            requests:
            cpu: "12"
            memory: 32Gi
        volumeMounts:
        - mountPath: /dev/shm
            name: dshm
        imagePullSecrets:
        - name: ngc-secret
        protocolVersions:
        - v2
        - grpc-v2
        supportedModelFormats:
        - autoSelect: true
        name: nvidia-nim-llama3-8b-instruct
        priority: 1
        version: "1.0.0"
        volumes:
        - emptyDir:
            medium: Memory
            sizeLimit: 16Gi
        name: dshm
    EOF

Apply the ``YAML`` file to your namespace:

.. code-block:: bash

   kubectl apply -f runtime.yaml

The runtime above is inspired by the runtimes published in the `NVIDIA/nim-deploy <https://github.com/NVIDIA/nim-deploy/tree/main/kserve/runtimes>`_ repository. 
Check it out for more details on available NIM runtimes.

.. note::
   This guide deviates from NVIDIA's runtime YAMLs by setting the ``NIM_CACHE_PATH`` to ``/tmp``. 
   This enforces the NIM container to download the model in memory instead of using a PVC and avoids `this KServe issue <https://github.com/kserve/kserve/issues/3687>`_.

------------------------
Create Inference Service
------------------------

Define a new Inference Service YAML file using the LLama3 runtime created in the previous step:

.. code-block:: bash

    cat <<EOF > "./isvc.yaml"
    apiVersion: serving.kserve.io/v1beta1
    kind: InferenceService
    metadata:
        annotations:
        autoscaling.knative.dev/target: "10"
        name: llama3-8b-instruct-1xgpu
    spec:
        predictor:
        minReplicas: 1
        model:
            modelFormat:
            name: nvidia-nim-llama3-8b-instruct
            resources:
            limits:
                nvidia.com/gpu: "1"
            requests:
                nvidia.com/gpu: "1"
            runtime: nvidia-nim-llama3-8b-instruct-1.0.0
    EOF

Apply the ``YAML`` file to your namespace:

.. code-block:: bash

   kubectl apply -f ./isvc.yaml

Wait until Inference Service is in ``Ready`` state.

.. note::
   This process can take up to 10 minutes because of pulling the large-size NIMs image and model.

You can check its state with:

.. code-block:: bash

   kubectl get inferenceservice llama3-8b-instruct-1xgpu

You should expect an output similar to this:

::

   NAME                       URL                                                         READY   PREV   LATEST   PREVROLLEDOUTREVISION   LATESTREADYREVISION                        AGE
   llama3-8b-instruct-1xgpu   http://llama3-8b-instruct-1xgpu.admin.10.64.140.43.nip.io   True           100                              llama3-8b-instruct-1xgpu-predictor-00001   16m

---------------------------------------
Make a request to the Inference Service
---------------------------------------

Get the Inference Service ``status.address.url`` and save it to a variable:

.. code-block:: bash

   URL=$(kubectl get inferenceservice llama3-8b-instruct-1xgpu -o jsonpath='{.status.address.url}')

Make a request to the Inference Service URL:

.. code-block:: bash

   curl $URL/v1/chat/completions -H "Content-Type: application/json" -d '{
   "model": "meta/llama3-8b-instruct",
   "messages": [{"role":"user","content":"What is Kubeflow?"}]
   }'

You should expect an output similar to this:

::

   {"id":"cmpl-c2ec0a9bf1d64172975992f8236fc166","object":"chat.completion","created":1729157204,"model":"meta/llama3-8b-instruct","choices":[{"index":0,"message":{"role":"assistant","content":"Kubeflow is an open-source platform for Machine Learning (ML) on Kubernetes. It allows data scientists and developers to easily build, deploy, and manage machine learning workloads on Kubernetes, a container orchestration system...."},"logprobs":null,"finish_reason":"stop","stop_reason":128009}],"usage":{"prompt_tokens":16,"total_tokens":426,"completion_tokens":410}}

