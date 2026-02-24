.. _release_notes_1.11:

Charmed Kubeflow 1.11
=====================

.. note::

   Release date: To Be Defined

This page contains the release notes for Charmed Kubeflow (CKF) 1.11.

See `Kubeflow 1.11.0 <https://github.com/kubeflow/manifests/releases/tag/v1.11.0>`_ for details on the upstream Kubeflow release notes. 
In comparison with the upstream Kubeflow project, CKF:

* Uses `OIDC-Authservice <https://github.com/arrikto/oidc-authservice>`_ for authentication.
* Does not include the `Model registry <https://github.com/kubeflow/model-registry>`_ and `Spark operator <https://github.com/kubeflow/spark-operator>`_ components.

What's new
----------

Highlights
~~~~~~~~~~

* Enabled the configuration of user workloads security policies. See :ref:`Configure user workloads to run unprivileged <configure_unprivileged_workloads>` for more information.
* Charms and workloads now run as unprivileged in the cluster.
* Introduced Kubeflow Trainer V2, working alongside Kubeflow Trainer V1.

Upgrades
~~~~~~~~

* Argo upgraded to 3.7.3 (previously 3.5.14).
* Istio upgraded to 1.28 (previously 1.24).
* KServe upgraded to 0.15.2 (previously 0.14.1).
* Pipelines upgraded to 2.15.0 (previously 2.5.0).
* Support for Kubernetes 1.29-1.33.

Features
~~~~~~~~

* Enhanced security by running the following charms and workloads as unprivileged:

  * Admission webhook (`#230 <https://github.com/canonical/admission-webhook-operator/pull/230>`_, `#239 <https://github.com/canonical/admission-webhook-operator/pull/239>`_)
  * Argo (`#300 <https://github.com/canonical/argo-operators/pull/300>`_)
  * Dashboard (`#307 <https://github.com/canonical/kubeflow-dashboard-operator/pull/307>`_)
  * Dex (`#291 <https://github.com/canonical/dex-auth-operator/pull/291>`_)
  * Envoy (`#192 <https://github.com/canonical/envoy-operator/pull/192>`_, `#200 <https://github.com/canonical/envoy-operator/pull/200>`_)
  * Istio (`#665 <https://github.com/canonical/istio-operators/pull/665>`_)
  * Jupyter Notebooks (`#522 <https://github.com/canonical/notebook-operators/pull/522>`_, `#537 <https://github.com/canonical/notebook-operators/pull/537>`_)
  * Katib (`#362 <https://github.com/canonical/katib-operators/pull/362>`_, `#390 <https://github.com/canonical/katib-operators/pull/390>`_)
  * Knative (`#393 <https://github.com/canonical/knative-operators/pull/393>`_)
  * Metacontroller (`#201 <https://github.com/canonical/metacontroller-operator/pull/201>`_)
  * Minio (`#279 <https://github.com/canonical/minio-operator/pull/279>`_)
  * ML Metadata (`#159 <https://github.com/canonical/mlmd-operator/pull/159>`_, `#167 <https://github.com/canonical/mlmd-operator/pull/167>`_)
  * OIDC-Authservice (`#232 <https://github.com/canonical/oidc-gatekeeper-operator/pull/232>`_)
  * Pipelines (`#809 <https://github.com/canonical/kfp-operators/pull/809>`_, `#851 <https://github.com/canonical/kfp-operators/pull/851>`_, `#852 <https://github.com/canonical/kfp-operators/pull/852>`_, `#853 <https://github.com/canonical/kfp-operators/pull/853>`_, `#856 <https://github.com/canonical/kfp-operators/pull/856>`_)
  * Profiles (`#277 <https://github.com/canonical/kubeflow-profiles-operator/pull/277>`_)
  * PVC Viewer (`#119 <https://github.com/canonical/pvcviewer-operator/pull/119>`_)
  * Tensorboards (`#226 <https://github.com/canonical/kubeflow-tensorboards-operator/pull/226>`_)
  * Trainer (`#284 <https://github.com/canonical/training-operator/pull/284>`_)
  * Training operator (`#282 <https://github.com/canonical/training-operator/pull/282>`_)
  * Volumes (`#227 <https://github.com/canonical/kubeflow-volumes-operator/pull/227>`_, `#235 <https://github.com/canonical/kubeflow-volumes-operator/pull/235>`_)

Bug fixes
---------

* Fixed an issue in `argo` charm alert rules causing an alert to be always fired in transient conditions (`#310 <https://github.com/canonical/argo-operators/pull/310>`_)
* Fixed an issue in `kserve` charm preventing its graceful removal (`#414 <https://github.com/canonical/kserve-operators/pull/414>`_)

Deprecated
----------

* The use of Kubeflow Pipelines SDK v1 is deprecated. Please migrate your existing v1 pipelines to v2 following the `migration instructions <https://www.kubeflow.org/docs/components/pipelines/user-guides/migration/>`_. SDK v1 can still be used but Canonical does not provide support, patches or fixes related to its use.
* Removed ``create-profile`` and ``initialise-profile`` actions from ``kubeflow-profiles`` (`#210 <https://github.com/canonical/kubeflow-profiles-operator/pull/210>`_). For managing profiles in CKF 1.11, see :ref:`Manage profiles <manage_profiles>`.
