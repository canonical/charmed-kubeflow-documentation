.. _release_notes_1.10:

Charmed Kubeflow 1.10
=====================

.. note::

   Release date: April 7th, 2025

This page contains the release notes for Charmed Kubeflow (CKF) 1.10.

See `Kubeflow 1.10.0 <https://github.com/kubeflow/manifests/releases/tag/v1.10.0>`_ for details on the upstream Kubeflow release notes. 
In comparison with the upstream Kubeflow project, CKF:

* Uses `OIDC-Authservice <https://github.com/arrikto/oidc-authservice>`_ for authentication.
* Does not include the `Model registry <https://github.com/kubeflow/model-registry>`_ and `Spark operator <https://github.com/kubeflow/spark-operator>`_ components.

What's new
----------

Highlights
~~~~~~~~~~

* Implemented automatic `profiles <https://www.kubeflow.org/docs/components/central-dash/profiles/#what-is-a-kubeflow-profile>`_ management using the new `GitHub Profiles Automator <https://charmhub.io/github-profiles-automator>`_ charm. This is an optional feature. See :ref:`Manage profiles <manage-profiles>` for more information.
* Enabled the configuration of High Availability for Istio gateway (`#553 <https://github.com/canonical/istio-operators/pull/553>`_).
* Improved application-health monitoring by exposing new metrics and providing new alert rules and Grafana dashboards for KServe, Istio and various other components.

Upgrades
~~~~~~~~

* Argo upgraded to 3.4.17 (previously 3.4.16).
* Dex upgraded to 2.41.1 (previously 2.39.1).
* Istio upgraded to 1.24 (previously 1.22).
* Knative upgraded to 1.16.0 (previously 1.12).
* KServe upgraded to 0.14.1 (previously 0.13.0).
* Metacontroller upgraded to 4.11.22 (previously 3.0).
* Pipelines upgraded to 2.4.1 (previously 2.3.0).
* Training-operator upgraded to 1.9 (previously 1.8).
* Support for Kubernetes 1.29-1.31.
* Support for Juju 3.6.

Features
~~~~~~~~

* Enhanced security by integrating the following `rocks <https://documentation.ubuntu.com/server/explanation/virtualisation/about-rock-images/index.html>`_:
    * Pipelines (`#158 <https://github.com/canonical/pipelines-rocks/pull/158>`_, `#159 <https://github.com/canonical/pipelines-rocks/pull/159>`_, `#160 <https://github.com/canonical/pipelines-rocks/pull/160>`_, `#3 <https://github.com/canonical/envoy-rock/pull/3>`_).
    * Kubeflow (`#87 <https://github.com/canonical/kubeflow-rocks/pull/87>`_, `#114 <https://github.com/canonical/kubeflow-rocks/pull/114>`_, `#154 <https://github.com/canonical/kubeflow-rocks/pull/154>`_).
    * Knative (`#4 <https://github.com/canonical/knative-rocks/pull/4>`_, `#5 <https://github.com/canonical/knative-rocks/pull/5>`_, `#6 <https://github.com/canonical/knative-rocks/pull/6>`_, `#10 <https://github.com/canonical/knative-rocks/pull/10>`_, `#18 <https://github.com/canonical/knative-rocks/pull/18>`_).
    * Training-operator (`#2 <https://github.com/canonical/training-operator-rock/pull/2>`_).
    * KServe (`#102 <https://github.com/canonical/kserve-rocks/pull/102>`_, `#103 <https://github.com/canonical/kserve-rocks/pull/103>`_, `#104 <https://github.com/canonical/kserve-rocks/pull/104>`_, `#106 <https://github.com/canonical/kserve-rocks/pull/106>`_).
* Enhanced observability by:
    * Enabling metrics for:
        * ``knative-operator`` (`#212 <https://github.com/canonical/knative-operators/pull/212>`_).
        * ``kserve-controller`` (`#261 <https://github.com/canonical/kserve-operators/pull/261>`_).
    * Enabling metrics and adding alert rules to:
        * ``istio-gateway`` (`#477 <https://github.com/canonical/istio-operators/pull/477>`_, `#514 <https://github.com/canonical/istio-operators/pull/514>`_).
        * ``tensorboard-controller`` (`#130 <130-1_>`__).
        * ``kubeflow-profiles`` (`#181 <https://github.com/canonical/kubeflow-profiles-operator/pull/181>`_).
    * Enabling metrics and adding alert rules and Grafana dashboard to ``kubeflow-dashboard`` (`#254 <https://github.com/canonical/kubeflow-dashboard-operator/pull/254>`_).
    * Adding basic alert rules regarding the charm's health to:
        * ``argo-controller`` (`#195 <https://github.com/canonical/argo-operators/pull/195>`_).
        * ``dex-auth`` (`#225 <https://github.com/canonical/dex-auth-operator/pull/225>`_).
        * ``envoy`` (`#130 <130-2_>`__). 
        * ``istio-pilot`` (`#515 <https://github.com/canonical/istio-operators/pull/515>`_).
        * ``katib-controller`` (`#231 <https://github.com/canonical/katib-operators/pull/231>`_).
        * ``knative-operator`` (`#215 <https://github.com/canonical/knative-operators/pull/215>`_).
        * ``kserve-controller`` (`#265 <https://github.com/canonical/kserve-operators/pull/265>`_).
        * ``metacontroller-operator`` (`#124 <https://github.com/canonical/metacontroller-operator/pull/124>`_).
        * ``minio`` (`#184 <https://github.com/canonical/minio-operator/pull/184>`_).
        * ``jupyter-controller`` (`#402 <https://github.com/canonical/notebook-operators/pull/402>`_).
        * ``pvcviewer-operator`` (`#55 <https://github.com/canonical/pvcviewer-operator/pull/55>`_).
        * ``training-operator`` (`#191 <https://github.com/canonical/training-operator/pull/191>`_).
    * Adding alert rules and an Istio control plane dashboard to ``istio-pilot`` (`#478 <https://github.com/canonical/istio-operators/pull/478>`_).

Bug fixes
---------

* Enabled the configuration of proxy environment variables in ``knative-serving`` controller (`#208 <https://github.com/canonical/knative-operators/pull/208>`_).
* Refactored ``knative-{eventing,serving}`` charms to block if the Knative `CRDs <https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/#customresourcedefinitions>`_ are not found (`#276 <https://github.com/canonical/knative-operators/pull/276>`_, `#281 <https://github.com/canonical/knative-operators/pull/281>`_).
* Refactored ``knative-{eventing,serving}`` charms to use a JSON file for handling custom images (`#304 <https://github.com/canonical/knative-operators/pull/304>`_).
* Enabled the configuration of proxy environment variables in the ``storage-initializer`` initContainer of ``kserve-controller`` (`#257 <https://github.com/canonical/kserve-operators/pull/257>`_).
* Removed unused `RBAC <https://kubernetes.io/docs/reference/access-authn-authz/rbac/>`_ proxy layer from ``kserve-controller`` (`#289 <https://github.com/canonical/kserve-operators/pull/289>`_).
* Added missing ``ClusterServingRuntime`` validation webhook (`#314 <https://github.com/canonical/kserve-operators/pull/314>`_).
* Added a namespace field to the ingress relation data in ``kubeflow-dashboard`` to fix cross-model ingresses (`#176 <https://github.com/canonical/kubeflow-dashboard-operator/pull/176>`_).
* Fixed the metrics collector for ``metacontroller-operator`` (`#101 <https://github.com/canonical/metacontroller-operator/pull/101>`_).
* Decreased the discovery interval of ``metacontroller-operator`` to update cached resources more quickly (`#117 <https://github.com/canonical/metacontroller-operator/pull/117>`_).
* Added missing `RBAC <https://kubernetes.io/docs/reference/access-authn-authz/rbac/>`_ rules to ``metacontroller-operator`` (`#158 <158-2_>`__).
* Set charm status to ``blocked`` when ``secret-key`` is too short in ``minio`` (`#178 <https://github.com/canonical/minio-operator/pull/178>`_).
* Removed an alert rule in ``jupyter-controller`` that was constantly firing due to an upstream bug (`#412 <https://github.com/canonical/notebook-operators/pull/412>`_).
* Used the same service name for its rock and charm for ``kfp-metadata-writer`` (`#167 <https://github.com/canonical/pipelines-rocks/pull/167>`_).

Deprecated
----------

* The use of Kubeflow Pipelines SDK v1 is deprecated. Please migrate your existing v1 pipelines to v2 following the `migration instructions <https://www.kubeflow.org/docs/components/pipelines/user-guides/migration/>`_. SDK v1 can still be used but Canonical does not provide support, patches or fixes related to its use.
* Removed ``create-profile`` and ``initialise-profile`` actions from ``kubeflow-profiles`` (`#210 <https://github.com/canonical/kubeflow-profiles-operator/pull/210>`_). For managing profiles in CKF 1.10, see :ref:`Manage profiles <manage-profiles>`.


.. Customized links

.. _130-1: https://github.com/canonical/kubeflow-tensorboards-operator/pull/130
.. _130-2: https://github.com/canonical/envoy-operator/pull/130

.. _158-1: https://github.com/canonical/pipelines-rocks/pull/158
.. _158-2: https://github.com/canonical/metacontroller-operator/pull/158
