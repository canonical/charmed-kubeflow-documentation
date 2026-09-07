.. _release_notes_1.11_ubuntu2:

Release notes for Charmed Kubeflow 1.11-ubuntu2
===============================================

Charmed Kubeflow 1.11-ubuntu2 promotes the ``track/1.11-rc`` release candidate into ``track/1.11``. It delivers a major update across Charmed Kubeflow solutions, introducing new product capabilities, upgrading core Kubeflow components, refactoring Terraform deployments, and hardening CI/UAT testing workflows.

Key changes
-----------

* **Kubeflow Pipelines (KFP) Upgraded:** Upgraded KFP charms to version **2.16**.
* **KServe Upgraded:** Upgraded KServe controller to version **0.17**.
* **Ambient Mesh & Proxy Enhancements:** Added ``kubeflow-ambient`` solution support, added capability to run Kubeflow behind a proxy, and enabled Ambient Mesh on the 1.11 release. As part of this enablement, Kubeflow Notebooks have been upgraded to the Canonical-built 1.11 version.
* **Spark Integration:** Added a new Kubeflow + Spark Terraform bundle for seamless Spark integration.
* **Kubeflow Trainer Repository Migration:** Migrated Kubeflow Trainer to the new repository and track layout.
* **Terraform Refactoring (CC008):** Underwent extensive Terraform refactoring to modularize components and integration interfaces.
* **UAT & CI Hardening:** Improved UAT and CI pipeline stability using image preloading, enhanced diagnostic logging, parallelized workflow execution, and updated deployment tests.

Upgrade instructions
--------------------

Refreshing charms (from 1.11 or 1.10)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To upgrade your charms to the ``1.11-ubuntu2`` revisions from previous 1.11 or 1.10 releases, execute the following commands:

.. code-block:: bash

   juju refresh admission-webhook --channel="2.0/stable"
   juju refresh argo-controller --channel="3.7/stable"
   juju refresh dex-auth --channel="2.41/stable"
   juju refresh envoy --channel="2.4/stable"
   juju refresh istio-ingressgateway --channel="1.28/stable"
   juju refresh istio-pilot --channel="1.28/stable"
   juju refresh jupyter-controller --channel="1.11/stable"
   juju refresh jupyter-ui --channel="1.11/stable"
   juju refresh katib-controller --channel="0.19/stable"
   juju refresh katib-db-manager --channel="0.19/stable"
   juju refresh katib-ui --channel="0.19/stable"
   juju refresh kfp-api --channel="2.16/stable"
   juju refresh kfp-metadata-writer --channel="2.16/stable"
   juju refresh kfp-persistence --channel="2.16/stable"
   juju refresh kfp-profile-controller --channel="2.16/stable"
   juju refresh kfp-schedwf --channel="2.16/stable"
   juju refresh kfp-ui --channel="2.16/stable"
   juju refresh kfp-viewer --channel="2.16/stable"
   juju refresh kfp-viz --channel="2.16/stable"
   juju refresh knative-eventing --channel="1.16/stable"
   juju refresh knative-operator --channel="1.16/stable"
   juju refresh knative-serving --channel="1.16/stable"
   juju refresh kserve-controller --channel="0.17/stable"
   juju refresh kubeflow-dashboard --channel="2.0/stable"
   juju refresh kubeflow-profiles --channel="2.0/stable"
   juju refresh kubeflow-roles --channel="1.10/stable"
   juju refresh kubeflow-trainer --channel="2.1/stable"
   juju refresh kubeflow-volumes --channel="1.11/stable"
   juju refresh metacontroller-operator --channel="4.11/stable"
   juju refresh minio --channel="1.10/stable"
   juju refresh mlmd --channel="ckf-1.10/stable"
   juju refresh mlflow-minio --channel="1.10/stable"
   juju refresh mlflow-server --channel="2.22/stable"
   juju refresh oidc-gatekeeper --channel="ckf-1.10/stable"
   juju refresh pvcviewer-operator --channel="1.11/stable"
   juju refresh resource-dispatcher --channel="2.0/stable"
   juju refresh tensorboard-controller --channel="1.11/stable"
   juju refresh tensorboards-web-app --channel="1.11/stable"
   juju refresh training-operator --channel="1.9/stable"

Upgrading from 1.10
~~~~~~~~~~~~~~~~~~~

If you are upgrading directly from **Charmed Kubeflow 1.10**, ensure you follow the official Istio upgrade procedure (upgrading Istio from version **1.24** to **1.28**) as described in the official upgrade documentation.

Important notes & troubleshooting
---------------------------------

Kubeflow Trainer (V2) migration (2.0 to 2.1)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Refreshing the ``kubeflow-trainer`` charm from ``2.0/edge`` to ``2.1/candidate`` may cause the unit to report an ``agent lost`` or degraded status. This is due to updates in Kubernetes labels and secret naming structures that previously conflicted.

If you encounter this issue, you can recover the application by running:

.. code-block:: bash

   kubectl delete deployment -n kubeflow kubeflow-trainer
   kubectl delete pod -n kubeflow kubeflow-trainer-0


Validation exclusion
~~~~~~~~~~~~~~~~~~~~~~

The current release has not been validated on a Kubernetes cluster running 1.34 with Ubuntu 20.04 as this is not a supported configuration. Also GPUs tests were not conducted on V100 GPUs as they are legacy hardware. 


Charm channels and revisions
----------------------------

The following table lists the charm channels and revisions included in this release:

.. list-table::
   :header-rows: 1
   :widths: 40 25 15

   * - Charm
     - Channel
     - Revision
   * - admission-webhook
     - 2.0/stable
     - 629
   * - argo-controller
     - 3.7/stable
     - 939
   * - dex-auth
     - 2.41/stable
     - 815
   * - envoy
     - 2.4/stable
     - 576
   * - istio-gateway
     - 1.28/stable
     - 1623
   * - istio-pilot
     - 1.28/stable
     - 1562
   * - jupyter-controller
     - 1.11/stable
     - 1540
   * - jupyter-ui
     - 1.11/stable
     - 1465
   * - katib-controller
     - 0.19/stable
     - 1360
   * - katib-db-manager
     - 0.19/stable
     - 1320
   * - katib-ui
     - 0.19/stable
     - 1322
   * - kfp-api
     - 2.16/stable
     - 2759
   * - kfp-metadata-writer
     - 2.16/stable
     - 1825
   * - kfp-persistence
     - 2.16/stable
     - 2768
   * - kfp-profile-controller
     - 2.16/stable
     - 2733
   * - kfp-schedwf
     - 2.16/stable
     - 2773
   * - kfp-ui
     - 2.16/stable
     - 2770
   * - kfp-viewer
     - 2.16/stable
     - 2794
   * - kfp-viz
     - 2.16/stable
     - 2716
   * - knative-eventing
     - 1.16/stable
     - 978
   * - knative-operator
     - 1.16/stable
     - 940
   * - knative-serving
     - 1.16/stable
     - 977
   * - kserve-controller
     - 0.17/stable
     - 1366
   * - kubeflow-dashboard
     - 2.0/stable
     - 1006
   * - kubeflow-profiles
     - 2.0/stable
     - 880
   * - kubeflow-roles
     - 1.10/stable
     - 423
   * - kubeflow-trainer
     - 2.1/stable
     - 148
   * - kubeflow-volumes
     - 1.11/stable
     - 652
   * - metacontroller-operator
     - 4.11/stable
     - 595
   * - minio
     - 1.10/stable
     - 686
   * - mlflow-server
     - 2.22/stable
     - 1395
   * - mlmd
     - ckf-1.10/stable
     - 441
   * - mysql-k8s
     - 8.0/stable
     - 423
   * - oidc-gatekeeper
     - ckf-1.10/stable
     - 642
   * - pvcviewer-operator
     - 1.11/stable
     - 485
   * - resource-dispatcher
     - 2.0/stable
     - 611
   * - tensorboard-controller
     - 1.11/stable
     - 715
   * - tensorboards-web-app
     - 1.11/stable
     - 700
   * - training-operator
     - 1.9/stable
     - 732
