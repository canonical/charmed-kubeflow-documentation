.. _upgrade_1.10_1.11:

Upgrade from 1.10 to 1.11
=========================

This guide describes how to upgrade Charmed Kubeflow (CKF) from 1.10 to 1.11.
Upgrading the solution by more than one minor version to another is not supported, nor is it recommended.

.. warning::
    
    CKF 1.11 is compatible with Charmed MLflow 2.15.
    If you have Charmed MLflow 2.1 deployed, `upgrade it to 2.15 <https://documentation.ubuntu.com/charmed-mlflow/how-to/manage/upgrade/migrate-v21-v215/>`_ before upgrading CKF.

---------------------
Before the upgrade
---------------------

Before upgrading CKF, you should do the following:

* Make sure:
  
  * All pipeline runs are completed and there are no recurring runs enabled.
  * Katib experiments, training jobs and notebooks are not in progress or pending.

* Back up any important data according to your organisation's policies. For databases, MinIO bucket pipelines and ML metadata, refer to the :ref:`backup guide for further details <back_up>`. For restoring that data, refer to the :ref:`restore guide <restore>`.

.. warning::
    
    The backup guide above does not guarantee the backup of all Kubeflow resources, such as notebooks and profiles. 
    Make sure to take the appropriate actions to avoid accidental data loss.

* Record all charm versions, including revisions, in your existing CKF deployment. This can be done by running ``juju export-bundle``.

---------------------
Upgrade environment
---------------------

~~~~~~~~~~~~~~~~~~~
Juju
~~~~~~~~~~~~~~~~~~~

The current stable version of CKF is supported on Juju 3.6. 
If needed, follow the `instructions <https://documentation.ubuntu.com/juju/3.6/howto/manage-your-juju-deployment/upgrade-your-juju-deployment/>`_ in order to upgrade the deployment. 
See :ref:`Supported versions <supported_kubeflow_versions>` for more information.

~~~~~~~~~~~~~~~~~~~
Kubernetes
~~~~~~~~~~~~~~~~~~~

The current stable version of CKF requires a Kubernetes cluster of at least version 1.29. 
For a full list of supported versions, refer to :ref:`Supported versions <supported_kubeflow_versions>`. 
Before upgrading to CKF, make sure this requirement is met.

---------------------
Upgrade charms
---------------------

To upgrade charms, you should follow the steps below in the proposed order.

.. note::

   Some charms may go to ``Blocked`` state during some steps of the upgrade process. 
   Once the upgrade is completed, all charms should be green and in ``Active`` state.

~~~~~~~~~~~~~~~~~~~
Istio
~~~~~~~~~~~~~~~~~~~

The current stable version of CKF requires Istio 1.28, please follow the instructions below to upgrade the ``istio-pilot`` and ``istio-gateway`` charms to that version:

1. Scale down the ``istio-ingressgateway`` application to 0:

.. code-block:: bash

    juju scale-application istio-ingressgateway 0

2. To make sure the ``istio-ingressgateway`` deployment is removed, run the following command. 
It should succeed by returning 0:

.. code-block:: bash

    kubectl -n kubeflow get deploy istio-ingressgateway-workload 2> >(grep -q "NotFound" && echo $?)

3. ``istio-pilot`` must be upgraded one minor version at a time. 

Upgrade ``istio-pilot`` charm to all intermediate versions until getting to the desired one.
Run each of the refresh commands separately and wait until the charm goes to ``Active`` state before running the next one:

.. code-block:: bash

    juju refresh istio-pilot --channel 1.25/stable
    juju refresh istio-pilot --channel 1.26/stable
    juju refresh istio-pilot --channel 1.27/stable
    juju refresh istio-pilot --channel 1.28/stable

.. warning::
    
    Failing to upgrade the ``istio-pilot`` charm one minor version at a time may result in the deployment being in an unrecoverable state.

4. Upgrade and scale up ``istio-ingressgateway`` charm:

.. code-block:: bash

    juju refresh istio-ingressgateway --channel 1.28/stable
    juju scale-application istio-ingressgateway 1

If you encounter any issues during the upgrade, 
refer to `Istio upgrade troubleshooting <https://github.com/canonical/istio-operators/blob/main/charms/istio-pilot/README.md>`_ for more details.

~~~~~~~~~~~~~~~~~~~~~~~~~~
Upgrade rest of the charms
~~~~~~~~~~~~~~~~~~~~~~~~~~

Upgrade the rest of the charms to their current stable versions with ``juju refresh``:

.. code-block:: bash

   juju refresh admission-webhook --channel 1.10/stable
   juju refresh argo-controller --channel 3.7/stable
   juju refresh dex-auth --channel 2.41/stable
   juju refresh envoy --channel 2.4/stable
   juju refresh jupyter-controller --channel 1.10/stable
   juju refresh jupyter-ui --channel 1.10/stable
   juju refresh katib-controller --channel 0.18/stable
   juju refresh katib-db-manager --channel 0.18/stable
   juju refresh katib-ui --channel 0.18/stable
   juju refresh kfp-api --channel 2.15/stable
   juju refresh kfp-metadata-writer --channel 2.15/stable
   juju refresh kfp-persistence --channel 2.15/stable
   juju refresh kfp-profile-controller --channel 2.15/stable
   juju refresh kfp-schedwf --channel 2.15/stable
   juju refresh kfp-ui --channel 2.15/stable
   juju refresh kfp-viewer --channel 2.15/stable
   juju refresh kfp-viz --channel 2.15/stable
   juju refresh knative-eventing --channel 1.16/stable
   juju refresh knative-operator --channel 1.16/stable
   juju refresh knative-serving --channel 1.16/stable
   juju refresh kserve-controller --channel 0.15/stable
   juju refresh kubeflow-dashboard --channel 1.10/stable
   juju refresh kubeflow-profiles --channel 1.10/stable
   juju refresh kubeflow-roles --channel 1.10/stable
   juju refresh kubeflow-volumes --channel 1.10/stable
   juju refresh metacontroller-operator --channel 4.11/stable
   juju refresh mlmd --channel ckf-1.10/stable
   juju refresh minio --channel ckf-1.10/stable
   juju refresh oidc-gatekeeper --channel ckf-1.10/stable
   juju refresh pvcviewer-operator --channel 1.10/stable
   juju refresh tensorboard-controller --channel 1.10/stable
   juju refresh tensorboards-web-app --channel 1.10/stable
   juju refresh training-operator --channel 1.9/stable
