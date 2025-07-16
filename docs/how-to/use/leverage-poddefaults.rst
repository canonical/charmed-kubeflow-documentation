.. _leverage_poddefaults:

Leverage PodDefaults
====================

This guide explains how to apply some common configurations to `Pods <https://kubernetes.io/docs/concepts/workloads/pods/>`_ 
with the use of `PodDefaults <https://github.com/kubeflow/kubeflow/blob/master/components/admission-webhook/README.md>`_.

PodDefaults in Charmed Kubeflow (CKF) automate the modification of PodSpecs requested by Kubeflow components at creation time. 
The PodDefault API resource is defined by a `CustomResourceDefinition <https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/#customresourcedefinitions>`_ (CRD) created by the ``admission-webhook`` charm.

-------------------------
Apply PodDefaults to Pods
-------------------------

PodDefaults configure Pods via the `PodDefaults Admission Controller <https://charmhub.io/admission-webhook>`_.

.. note::

   For attaching a PodDefault to a Pod, the Pod to be configured must have a label that matches the PodDefault's spec, specifically the ``spec.selector.matchLabels`` field.

----------------------------
Inject environment variables
----------------------------

Create a PodDefault to inject environment variables into a Pod as follows:

.. code-block:: bash

    apiVersion: kubeflow.org/v1alpha1
    kind: PodDefault
    metadata:
        name: env-poddefault
    spec:
        desc: short description of the PodDefault
        env:
        - name: ENV1
        value: value1
        - name: ENV2
        value: value2
        selector:
        matchLabels:
            env-poddefault: "true"

This PodDefault adds ``ENV1`` and ``ENV2`` with values ``value1`` and ``value2`` respectively to all containers of the Pod with the label ``env-poddefault``.

For example, `the PodDefault for integrating MinIO <https://charmed-kubeflow.io/docs/integrate-with-minio#configure-access>`_ adds the environment variables with S3 credentials.

.. _add_tolerations:

---------------------
Add tolerations
---------------------

Create a PodDefault to add tolerations to a Pod as follows:

.. code-block:: bash

    apiVersion: kubeflow.org/v1alpha1
    kind: PodDefault
    metadata:
        name: tolerations-poddefault
    spec:
        desc: short description of the PodDefault
        tolerations:
        - key: "myToleration"
        operator: "Equal"
        value: "true"
        effect: "NoSchedule"
        selector:
        matchLabels:
            tolerations-poddefault: "true"

This PodDefault adds to the Pod ``tolerations``, where the toleration allows Pods with the label ``toleration-poddefault`` to be scheduled on nodes that have a taint with the same key and value as mentioned in the toleration.

See `Taints and Tolerations <https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/>`_ for more details.

-------------------------
Set command and arguments
-------------------------

Create a PodDefault to set the commands and arguments of containers in the Pod as follows:

.. code-block:: bash

    apiVersion: kubeflow.org/v1alpha1
    kind: PodDefault
    metadata:
        name: command-poddefault
    spec:
        desc: short description of the PodDefault
        command: ["printenv"]
        args: ["HOSTNAME", "KUBERNETES_PORT"]
        selector:
        matchLabels:
            command-poddefault: "true"

This PodDefault sets the command and arguments to print the values of environment variables ``HOSTNAME`` and ``KUBERNETES_PORT``.

For example, the `allow-ngc-notebook PodDefault <https://github.com/canonical/ngc-integrator-operator/blob/main/src/templates/poddefault.yaml>`_ in the ``ngc-integrator`` charm sets the command and args to start an NGC notebook.

---------------------
Disable Istio sidecar
---------------------

By default, Istio sidecar injection for Pods in Kubeflow namespaces is enabled. 
To disable it, you can leverage the following PodDefault:

.. code-block:: bash

   apiVersion: kubeflow.org/v1alpha1
   kind: PodDefault
   metadata:
     name: disable-istio-poddefault
   spec:
     desc: Disable Istio sidecar injection
     annotations:
       "sidecar.istio.io/inject": "false"
     selector:
       matchLabels:
         disable-istio-sidecar: "true"

For example, to create a KServe ``InferenceService`` as shown in :ref:`Build your first ML model <build_your_first_ml_model>`, 
you need to disable Istio sidecar injection for the ``InferenceService`` Pod.

In the tutorial, it is done by adding the annotation ``"sidecar.istio.io/inject": "false"``. 
Instead, you can create the PodDefault above and apply the ``disable-istio-sidecar`` label to the ``InferenceService`` definition.

