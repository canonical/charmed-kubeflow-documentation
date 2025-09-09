.. _upgrade_1.8_1.9:

Upgrade from 1.8 to 1.9
=======================

This guide describes how to upgrade Charmed Kubeflow (CKF) from 1.8 to 1.9 version.

This requires upgrading each charm individually. 
New relations must be added separately. 
Most charms can be upgraded simply with ``juju refresh``, however certain components require additional steps to upgrade.

.. warning::
    
    CKF 1.9 is compatible with Charmed MLflow 2.15. If you have Charmed MLflow 2.1 deployed, you should upgrade it to 2.15 by following `this upgrade guide <https://documentation.ubuntu.com/charmed-mlflow/how-to/manage/upgrade/migrate-v21-v215/>`_ before upgrading Charmed Kubeflow.

---------------------
Before the upgrade
---------------------

Before upgrading CKF, you should do the following:

* Make sure:

  * All pipeline runs are completed and there are no recurring runs enabled.
  * Katib experiments, training jobs and notebooks are not in progress or pending.

* Back up any important data according to your organisation's policies. For databases, MinIO bucket pipelines and ML metadata, refer to the :ref:`backup guide <back_up>` for further details. For restoring that data, refer to the :ref:`restore guide <restore_control_plane>`.

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

As with the `1.8 latest update <https://discourse.charmhub.io/t/charmed-kubeflow-support-for-juju-3/14734>`_, 
Charmed Kubeflow 1.9 is supported on Juju 3.4 (>= 3.4.3). 
Make sure to use a compatible version. 
If needed, follow these `instructions <https://documentation.ubuntu.com/juju/3.6/howto/manage-your-juju-deployment/upgrade-your-juju-deployment/>`_ in order to upgrade the deployment.

~~~~~~~~~~~~~~~~~~~
Kubernetes
~~~~~~~~~~~~~~~~~~~

Due to Istio, CKF requires a Kubernetes cluster >=1.27 (see :ref:`Supported versions <supported_kubeflow_versions>`). 
Before upgrading to CKF 1.9, make sure this requirement is met.

---------------------
Upgrade charms
---------------------

To upgrade charms, you should follow the steps below in the proposed order.

.. note::

   Some charms may go to ``Blocked`` state during some steps of the upgrade process. Once the upgrade is completed, all charms should be green and in ``Active`` state.

~~~~~~~~~~~~~~~~~~~
Istio
~~~~~~~~~~~~~~~~~~~

Istio needs to be upgraded to version 1.22, assuming the deployed ``istio-pilot`` and ``istio-ingressgateway`` versions are 1.17.

1. Scale down the ``istio-ingressgateway`` application to 0:

.. code-block:: bash

    juju scale-application istio-ingressgateway 0

2. To make sure the ``istio-ingressgateway`` deployment is removed, run the following command. It should succeed by returning 0:

.. code-block:: bash

    kubectl -n kubeflow get deploy istio-ingressgateway-workload 2> >(grep -q "NotFound" && echo $?)

3. Upgrade ``istio-pilot`` charm to all intermediate versions. Thus, run each of the following commands separately and wait until it goes to ``Active`` state before running the next one:

.. code-block:: bash

    juju refresh istio-pilot --channel 1.18/stable
    juju refresh istio-pilot --channel 1.19/stable
    juju refresh istio-pilot --channel 1.20/stable
    juju refresh istio-pilot --channel 1.21/stable
    juju refresh istio-pilot --channel 1.22/stable

4. Upgrade and scale up ``istio-ingressgateway`` charm:

.. code-block:: bash

    juju refresh istio-ingressgateway --channel 1.22/stable
    juju scale-application istio-ingressgateway 1

If you encounter any issues during the upgrade, 
refer to `Istio upgrade troubleshooting <https://github.com/canonical/istio-operators/blob/main/charms/istio-pilot/README.md>`_ for more details.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
PodSpec to Sidecar charms
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

^^^^^^^^^^^^^^^^^^^
Mlmd
^^^^^^^^^^^^^^^^^^^

1. :ref:`Backup ML metadata <back_up>` for MLMD <= 1.14 and CKF 1.8.

2. Remove the relation with ``requirer`` charms (``envoy`` and ``kfp-metadata-writer``):

.. code-block:: bash

    juju remove-relation envoy mlmd
    juju remove-relation kfp-metadata-writer mlmd

.. note::

    ``grpc`` relations are restored once the ``requirer`` charms are upgraded. You'll do that in the "Add grpc relations" step of :ref:`Charms with refresh section <charms_with_refresh>`.

3. Remove the ``mlmd`` application:

.. warning::
    
    This wipes out the storage attached to the ``mlmd`` charm, that is, the database handled by this charm. Make sure you have performed the backup from step 1.

.. note::

    You must wait for the application to disappear. It takes less than a minute.

.. code-block:: bash

    juju remove-application mlmd --destroy-storage

4. Deploy ``mlmd`` from 1.9 corresponding channel:

.. code-block:: bash

    juju deploy mlmd --channel ckf-1.9/stable --trust

5. :ref:`Restore ML metadata <restore_mlmd_sqlite>` for MLMD > 1.14 and CKF 1.9.

^^^^^^^^^^^^^^^^^^^^^^^
Rest of PodSpec charms
^^^^^^^^^^^^^^^^^^^^^^^

Juju 3.4 requires to scale down the application, refresh it, and then scale it up.

.. warning::

    If CKF is deployed on AKS, skip this section and follow instead the `Rest of PodSpec charms on AKS`_ section.

1. Scale down applications:

.. note::

    You must wait for the units to disappear. It takes less than a minute.

.. code-block:: bash

    juju scale-application katib-controller 0
    juju scale-application kubeflow-volumes 0
    juju scale-application envoy 0

2. Refresh to the new charms:

.. code-block:: bash

    juju refresh katib-controller --channel 0.17/stable --trust
    juju refresh kubeflow-volumes --channel 1.9/stable --trust
    juju refresh envoy --channel 2.2/stable --trust

3. Scale up the applications:

.. code-block:: bash

    juju scale-application katib-controller 1
    juju scale-application kubeflow-volumes 1
    juju scale-application envoy 1

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Rest of PodSpec charms on AKS
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Due to `this bug <https://bugs.launchpad.net/juju/+bug/2073529>`_, 
the standard PodSpec charms upgrade path with ``juju refresh`` on AKS ends up with them being stuck in ``Unknown`` status, 
unable to spin up a new refreshed unit. Instead, you can apply the following workaround:

1. The commands below prevent the loss of your workloads created by Katib:

.. code-block:: bash

    kubectl annotate crd experiments.kubeflow.org controller.juju.is/id-
    kubectl annotate crd experiments.kubeflow.org model.juju.is/id-
    kubectl label crd experiments.kubeflow.org app.juju.is/created-by-
    kubectl label crd experiments.kubeflow.org app.kubernetes.io/managed-by-
    kubectl label crd experiments.kubeflow.org app.kubernetes.io/name-
    kubectl label crd experiments.kubeflow.org model.juju.is/name-

    kubectl annotate crd trials.kubeflow.org controller.juju.is/id-
    kubectl annotate crd trials.kubeflow.org model.juju.is/id-
    kubectl label crd trials.kubeflow.org app.juju.is/created-by-
    kubectl label crd trials.kubeflow.org app.kubernetes.io/managed-by-
    kubectl label crd trials.kubeflow.org app.kubernetes.io/name-
    kubectl label crd trials.kubeflow.org model.juju.is/name-

    kubectl annotate crd suggestions.kubeflow.org controller.juju.is/id-
    kubectl annotate crd suggestions.kubeflow.org model.juju.is/id-
    kubectl label crd suggestions.kubeflow.org app.juju.is/created-by-
    kubectl label crd suggestions.kubeflow.org app.kubernetes.io/managed-by-
    kubectl label crd suggestions.kubeflow.org app.kubernetes.io/name-
    kubectl label crd suggestions.kubeflow.org model.juju.is/name-

2. Remove PodSpec charms:

.. code-block:: bash

    juju remove-application katib-controller
    juju remove-application kubeflow-volumes
    juju remove-application envoy

3. Wait until the applications have been removed. To make sure all related resources are removed, run the following commands. They should succeed by returning 0:

.. code-block:: bash

    juju show-application katib-controller 2> >(grep -q "not found" && echo $?)
    kubectl -n kubeflow get deploy katib-controller 2> >(grep -q "NotFound" && echo $?)
    juju show-application kubeflow-volumes 2> >(grep -q "not found" && echo $?)
    kubectl -n kubeflow get deploy kubeflow-volumes 2> >(grep -q "NotFound" && echo $?)
    juju show-application envoy 2> >(grep -q "not found" && echo $?)
    kubectl -n kubeflow get deploy envoy 2> >(grep -q "NotFound" && echo $?)

4. Deploy the new charms and add the relations:

.. code-block:: bash

    juju deploy envoy --channel 2.2/stable --trust
    juju deploy kubeflow-volumes --channel 1.9/stable --trust
    juju deploy katib-controller --channel 0.17/stable --trust
    juju integrate kubeflow-dashboard:links kubeflow-volumes:dashboard-links
    juju integrate istio-pilot:ingress kubeflow-volumes:ingress
    juju integrate istio-pilot:ingress envoy:ingress

.. _charms_with_refresh:

~~~~~~~~~~~~~~~~~~~
Charms with refresh
~~~~~~~~~~~~~~~~~~~

1. Upgrade the rest of the charms with ``juju refresh``:

.. code-block:: bash

    juju refresh admission-webhook --channel 1.9/stable
    juju refresh argo-controller --channel 3.4/stable
    juju refresh dex-auth --channel 2.39/stable
    juju refresh jupyter-controller --channel 1.9/stable
    juju refresh jupyter-ui --channel 1.9/stable
    juju refresh katib-db-manager --channel 0.17/stable
    juju refresh katib-ui --channel 0.17/stable
    juju refresh kfp-api --channel 2.3/stable
    juju refresh kfp-metadata-writer --channel 2.3/stable
    juju refresh kfp-persistence --channel 2.3/stable
    juju refresh kfp-profile-controller --channel 2.3/stable
    juju refresh kfp-schedwf --channel 2.3/stable
    juju refresh kfp-ui --channel 2.3/stable
    juju refresh kfp-viewer --channel 2.3/stable
    juju refresh kfp-viz --channel 2.3/stable
    juju refresh knative-eventing --channel 1.12/stable
    juju refresh knative-operator --channel 1.12/stable
    juju refresh knative-serving --channel 1.12/stable
    juju refresh kserve-controller --channel 0.13/stable
    juju refresh kubeflow-dashboard --channel 1.9/stable
    juju refresh kubeflow-profiles --channel 1.9/stable
    juju refresh kubeflow-roles --channel 1.9/stable
    juju refresh metacontroller-operator --channel 3.0/stable
    juju refresh minio --channel ckf-1.9/stable
    juju refresh oidc-gatekeeper --channel ckf-1.9/stable
    juju refresh pvcviewer-operator --channel 1.9/stable
    juju refresh tensorboard-controller --channel 1.9/stable
    juju refresh tensorboards-web-app --channel 1.9/stable
    juju refresh training-operator --channel 1.8/stable

2. Add ``grpc`` relations to ``mlmd``:

.. code-block:: bash

    juju integrate envoy:grpc mlmd:grpc
    juju integrate kfp-metadata-writer:grpc mlmd:grpc

3. Add new relations:

.. code-block:: bash

    juju integrate katib-db-manager:k8s-service-info katib-controller:k8s-service-info
    juju integrate kubeflow-dashboard:links training-operator:dashboard-links
    juju integrate oidc-gatekeeper:dex-oidc-config dex-auth:dex-oidc-config

