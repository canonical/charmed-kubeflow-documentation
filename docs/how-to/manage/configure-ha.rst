.. _configure_ha:

Configure High Availability for Istio Gateway
=============================================

By default, `Istio <https://charmhub.io/istio>`_ is deployed with a single replica of the Gateway workload pod.

This guide shows how to deploy multiple replicas of the pod and spread them across different nodes by configuring High Availability (HA) 
for `Istio Gateway <https://charmhub.io/istio-gateway?channel=1.22/stable>`_. 
The configuration uses `Kubernetes inter-pod anti-affinity <https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#inter-pod-affinity-and-anti-affinity>`_.

Configuring HA makes your Istio deployment more resilient, reducing the risk of a single point of failure.
That is, this ensures that if an Istio Gateway workload pod is down, the rest of Kubeflow can still be accessed.

---------------------
Requirements
---------------------

* A multi-node Kubernetes cluster.
* `istio-pilot <https://charmhub.io/istio-pilot?channel=1.22/stable>`_ and ``istio-gateway`` 1.22 version or above.
* `kubectl <https://snapcraft.io/kubectl>`_.

.. note::

   The Istio HA configuration is only available in version ``1.22/*`` or above. 
   To upgrade to a higher version, see `Istio instructions <https://discourse.charmhub.io/t/how-to-upgrade-kubeflow-from-1-8-to-1-9/14913#istio-6>`_.

---------------------------
Configure High Availability
---------------------------

You can enable the Istio HA by setting the ``replicas`` configuration value for the ``istio-ingressgateway`` charm. 
You can do so as follows:

.. code-block:: bash

   juju config istio-ingressgateway replicas=<desired number of replicas>

.. note::

   The number of replicas must be less than or equal to the number of available nodes in your cluster. 
   Otherwise, the additional pods will remain in ``Pending`` status.

------------------------
Verify High Availability
------------------------

Once the ``istio-ingressgateway`` charm is configured, you can verify it's running with HA by checking the pods with the ``istio-ingressgateway`` label:

.. code-block:: bash

   kubectl get po -n kubeflow -l app=istio-ingressgateway -o wide

For example, consider your cluster consists of two or more nodes. If you set the ``replicas`` config to ``2``, you should see 2 running pods:

.. code-block:: bash

   NAME                                                 READY   STATUS    RESTARTS   AGE   IP             NODE
   istio-ingressgateway-workload-86d4dd6dff-84g6l       1/1     Running   0          6m    10.1.58.136    node1
   istio-ingressgateway-workload-86d4dd6dff-j9fhv       1/1     Running   0          4m    10.1.179.133   node2

Each pod is always scheduled on a different node due to `inter-pod anti-affinity <https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#inter-pod-affinity-and-anti-affinity>`_ being set in the Istio Gateway deployment.
