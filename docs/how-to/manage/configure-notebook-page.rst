.. _configure_notebook_page:

Configure Kubeflow Notebook creation page
=========================================

This guide describes how to configure the Kubeflow Notebook creation page. 
This involves customising certain options available to users through the ``New Notebook`` User Interface (UI), such as suggested container images or GPU configurations.

.. note::

   This guide is intended for administrators.

------------------------------------
Modify the suggested notebook images
------------------------------------

The Notebook creation page includes dropdown lists of suggested container images for notebook servers:

.. image:: https://assets.ubuntu.com/v1/be2edbc1-kubeflow-notebook-creation-page1.png

By default, these include standard images built by Canonical or the Kubeflow project and are grouped into Jupyterlab, VisualStudio Code, and RStudio images. 
As an administrator, you can modify these lists.

~~~~~~~~~~~~~~~~~~~~~~
List available images
~~~~~~~~~~~~~~~~~~~~~~

You can list the available images for Jupyterlab, VisualStudio Code, and RStudio images using ``juju config`` with the ``jupyter-ui`` charm as follows:

.. code-block:: bash

   juju config jupyter-ui jupyter-images

   juju config jupyter-ui vscode-images

   juju config jupyter-ui rstudio-images

~~~~~~~~~~~~~~~~~~~~~
Configure images list
~~~~~~~~~~~~~~~~~~~~~

The ``jupyter-images``, ``vscode-images``, and ``rstudio-images`` configurations are YAML lists including image names. 
You can modify the default configuration by loading a new ``YAML`` file.

For example, define a file named ``images.yaml`` with the following contents:

.. code-block:: bash

   - kubeflownotebookswg/jupyter-pytorch-full:v1.9.0
   - kubeflownotebookswg/jupyter-tensorflow-full:v1.9.0
   - bitnami/jupyter-base-notebook:4.1.5

.. note::

   All image names must be available from where your notebook image deploys.

Now, pass this configuration file to Juju by running:

.. code-block:: bash

   juju config jupyter-ui jupyter-images=@images.yaml

You can customise ``vscode-images`` and ``rstudio-images`` in the same way.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Modify the existing list of images
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Setting ``juju config`` updates the entire configuration, rather than modifying the existing one. 
If you want to do so, for example, adding a new notebook image to your current configuration, you have to export the current config to a file, edit it, and import it back:

.. code-block:: bash

   juju config jupyter-ui jupyter-images > images_v1.yaml

   # edit images_v1.yaml, save as images_v2.yaml
   juju config jupyter-ui jupyter-images=@images_v2.yaml

---------------------
GPU configurations
---------------------

Kubeflow Notebooks can use any `GPU resources <https://kubernetes.io/docs/tasks/manage-gpus/scheduling-gpus/>`_ available in the Kubernetes cluster. 
You can configure the type and number of GPU resources available to users.

Charmed Kubeflow (CKF) exposes this configuration to administrators through the following items:

* ``gpu-vendors``: The GPU vendors that are selectable by users in the ``New Notebook`` UI when creating a notebook. The input can be in JSON or YAML format with key values. See the `upstream configuration file <https://github.com/kubeflow/notebooks/blob/notebooks-v1/components/crud-web-apps/jupyter/manifests/base/configs/spawner_ui_config.yaml>`_ for more details:
  
  * ``limitsKey``: the key that corresponds to the GPU vendor resource in Kubernetes.
  * ``uiName``: the name to be shown in the UI for this GPU.
* ``gpu-vendors-default``: The GPU vendor that is selected by default in the ``New Notebook`` creation page when creating a notebook. This must be one of the ``limitsKey`` values from the gpu-vendors config.
* ``gpu-number-default``: The number of GPUs that are selected by default in the ``New Notebook`` UI when creating a notebook.

.. note::

   ``gpu-vendors-default`` can be left as an empty string to select no GPU vendor by default.

Users see these in the dropdown menus:

.. image:: https://assets.ubuntu.com/v1/93b629ca-kubeflow-notebook-creation-page2.png

To set the list for available GPU resources, run:

.. code-block:: bash

   juju config jupyter-ui gpu-vendors='[{"limitsKey": "intel.com/gpu", "uiName": "Intel"}, {"limitsKey": "nvidia.com/gpu", "uiName": "NVIDIA"}, {"limitsKey": "amd.com/gpu", "uiName": "AMD"}]'

.. note::

   The command above overwrites the previous configuration, so all fields have to be specified.

For example, you can set the default notebook to use two NVIDIA GPUs as follows:

.. code-block:: bash

   juju config jupyter-ui gpu-number-default 2
   juju config jupyter-ui gpu-vendors-default nvidia.com/gpu

---------------------
Node Affinities
---------------------

You can configure Kubeflow Notebooks to use `Node Affinities <https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#affinity-and-anti-affinity>`_ 
when scheduling the notebook within the cluster. 
For instance, this can be used to assign notebooks to a specific node type, avoiding scheduling more than one notebook on a given node.

CKF exposes this configuration to administrators through the following items:

* ``affinity-options``: The Node Affinity configurations that are selectable by users in the ``New Notebook`` UI when creating a notebook. The input can be in JSON or YAML format with key values. See the `upstream configuration file <https://github.com/kubeflow/notebooks/blob/notebooks-v1/components/crud-web-apps/jupyter/manifests/base/configs/spawner_ui_config.yaml>`_ for more details:
  
  * ``configKey``: an arbitrary key for the configuration.
  * ``displayName``: the name shown in the ``New Notebook`` UI.
  * ``affinity``: the `affinity configuration <https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/>`_.
* ``affinity-options-default``: The ``configKey`` of the affinity to be chosen by default. Leave it as an empty string to select no affinity by default.

Users see these options from the dropdown menu:

.. image:: https://assets.ubuntu.com/v1/547dcab2-kubeflow-notebook-creation-page3.png

To change these settings, for example, if your cluster has east and west availability zones defined by node labels and you want users to be able to choose them, 
you have to modify the default configuration by creating a new ``YAML`` file and passing it to Juju.

First, create ``affinity_config.yaml`` file as follows:

.. code-block:: bash

    - configKey: "az_us-east1"
        displayName: "Availability Zone us-east1"
        affinity:
        nodeAffinity:
            requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
                - matchExpressions:
                - key: topology.kubernetes.io/zone
                    operator: In
                    values:
                    - us-east1
    - configKey: "az_us-west1"
        displayName: "Availability Zone us-west1"
        affinity:
        nodeAffinity:
            requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
                - matchExpressions:
                - key: topology.kubernetes.io/zone
                    operator: In
                    values:
                    - us-west1

Now, set the configuration, where the ``az_us-west1`` is chosen by default:

.. code-block:: bash

   juju config jupyter-ui affinity-options=@affinity_config.yaml
   juju config jupyter-ui affinity-options-default="az_us-west1"

---------------------
Use Pod tolerations
---------------------

You can configure Kubeflow Notebooks to use `Pod tolerations <https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/>`_ 
when scheduling the notebook within the cluster. 
For instance, this can be used to allow a notebook to be scheduled to a specific node, such as a node that uses GPU or some other special resource.

CKF exposes this configuration to administrators through the following items:

* ``tolerations-options``: The tolerations configurations that are selectable by users in the ``New Notebook`` UI when creating a notebook. The input can be JSON or YAML format with key values. See the `upstream configuration file <https://github.com/kubeflow/notebooks/blob/notebooks-v1/components/crud-web-apps/jupyter/manifests/base/configs/spawner_ui_config.yaml>`_ for more details:
  
  * ``groupKey``: an arbitrary key for the configuration.
  * ``displayName``: the name shown in the ``New Notebook`` UI.
  * ``tolerations``: the `toleration configuration <https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/>`_.
* ``tolerations-options-default``: The ``groupKey`` of the toleration to be chosen by default. Leave it as an empty string to select no toleration by default.

Users see these options from the dropdown menu:

.. image:: https://assets.ubuntu.com/v1/9b32edf3-kubeflow-notebook-creation-page4.png

To change the tolerations options for your cluster, you have to modify the default configuration by creating a new ``YAML`` file and passing it to Juju.

First, create the ``tolerations_config.yaml`` file:

.. code-block:: bash

    - groupKey: "group_1"
        displayName: "4 CPU 8GB at ~$0.50 USD per day"
        tolerations:
        - key: "dedicated"
            operator: "Equal"
            value: "kubeflow-c5.xlarge"
            effect: "NoSchedule"

    - groupKey: "group_2"
        displayName: "8 CPU 16GB at ~$1.20 USD per day"
        tolerations:
        - key: "dedicated"
            operator: "Equal"
            value: "kubeflow-c5.xxlarge"
            effect: "NoSchedule"

Use ``juju config`` to set the configuration for the toleration:

.. code-block:: bash

   juju config jupyter-ui tolerations-options=@tolerations_config.yaml
   juju config jupyter-ui tolerations-options-default=""

.. note::

   If the value ``tolerations-options-default`` is an empty string, then no toleration is selected by default.

----------------------------------------------
Create default configurations with PodDefaults
----------------------------------------------

You can use PodDefaults to inject common data and/or configuration to several notebooks at the same time. 
PodDefaults is a namespaced custom resource that defines the configuration to be overlaid on a Pod. 
Each user has access only to the PodDefaults defined in their own namespace. Users can create their own PodDefaults. 
Administrators can provide PodDefaults to users by adding them to the user's namespaces.

Kubeflow Notebooks can be configured to use PodDefaults through the ``New Notebook`` UI. 
This can be used, for example, to automatically inject credentials for an MLflow or S3 store. 
These configurations can be chosen by the user during the notebook creation.

CKF exposes this configuration to administrators through the following items:

* ``default-poddefaults``: The PodDefaults that are selected for the user by default in the ``New Notebook`` UI when creating a notebook. The input can be in ``JSON`` or ``YAML`` format as a list of PodDefaults names. See `upstream configuration file <https://github.com/kubeflow/notebooks/blob/notebooks-v1/components/crud-web-apps/jupyter/manifests/base/configs/spawner_ui_config.yaml>`_ for more details.

Users see these options in the dropdown menu:

.. image:: https://assets.ubuntu.com/v1/899cb7a9-kubeflow-notebook-creation-page5.png

To change this default configuration, use the ``juju config`` command:

.. code-block:: bash

   juju config jupyter-ui default-poddefaults='["add-s3-credentials", "add-mlflow-credentials"]'

.. note::
    To check the available PodDefaults in a specific namespace, you can use ``kubectl get poddefault -n <namespace>``.


