.. _release_notes_1.9:

Charmed Kubeflow 1.9
=====================

.. note::
   Release date: July 31st, 2024

What's new
----------

* Support for Kubernetes from version 1.26 to 1.29.
* `Support for Juju 3.4 <https://discourse.charmhub.io/t/charmed-kubeflow-support-for-juju-3/14734>`_ (previously 3.1).
* Pipelines upgraded to version 2.2.0 (previously 2.0.3). `#483 <https://github.com/canonical/kfp-operators/pull/483>`_
* KServe upgraded to version 0.13 (previously 0.11). `#239 <https://github.com/canonical/kserve-operators/pull/239>`_
* Istio upgraded to version 1.22 (previously 1.17).
* Knative upgraded to version 1.12 (previously 1.10). `#182 <https://github.com/canonical/knative-operators/pull/182>`_
* Integration with Nvidia NGC containers via the `ngc-integrator-operator <https://github.com/canonical/ngc-integrator-operator>`_ charm. 
* Integration with Triton Inference Server for KServe. `#171 <https://github.com/canonical/knative-operators/issues/171>`_ 
* Charms state grafana dashboard. `#877 <https://github.com/canonical/bundle-kubeflow/issues/877>`_
* Enabled Istio CNI plugin. `#365 <https://github.com/canonical/istio-operators/pull/365>`_ 
* Reworked the way Charmed Kubeflow is monitored with the Canonical Observability stack. 
* Added ROCKs for following charms `#94 <https://github.com/canonical/pipelines-rocks/issues/94>`_, `#95 <https://github.com/canonical/pipelines-rocks/issues/95>`_, `#96 <https://github.com/canonical/pipelines-rocks/issues/96>`_, `#97 <https://github.com/canonical/pipelines-rocks/issues/97>`_, `#98 <98-1_>`__, `#99 <99-1_>`__, `#23 <https://github.com/canonical/argo-workflows-rocks/issues/23>`_, `#14 <https://github.com/canonical/dex-auth-rocks/issues/14>`_, `#67 <https://github.com/canonical/kserve-rocks/issues/67>`_, `#68 <https://github.com/canonical/kserve-rocks/issues/68>`_, `#69 <https://github.com/canonical/kserve-rocks/issues/69>`_, `#70 <https://github.com/canonical/kserve-rocks/issues/70>`_, `#71 <https://github.com/canonical/kserve-rocks/issues/71>`_, `#72 <https://github.com/canonical/kserve-rocks/issues/72>`_ `#73 <https://github.com/canonical/kserve-rocks/issues/73>`_, `#74 <https://github.com/canonical/kserve-rocks/issues/74>`_, `#99 <99-2_>`__, `#98 <98-2_>`__.

Bug fixes
---------

* Fixed non-working grafana dashboards for some charms (minio, notebooks, envoy, katib, argo). `#856 <https://github.com/canonical/bundle-kubeflow/issues/856>`_
* Fixed katib-ui charm trying to access the workload container by the wrong name. `#156 <https://github.com/canonical/katib-operators/issues/156>`_
* Introduced k8s-service-info relation between katib-controller and katib-db-manager. `#185 <https://github.com/canonical/katib-operators/pull/185>`_
* Refactored the kserve-controller charm to use an all-catch main event handler. `#197 <https://github.com/canonical/kserve-operators/pull/197>`_
* Introduced handling of relation-broken events in kfp-api. `#272 <https://github.com/canonical/kfp-operators/pull/272>`_
* Patched KFP Profile Controller service port. `#318 <https://github.com/canonical/kfp-operators/pull/318>`_
* Set explicitly the command for ``kfp-schedwf`` Pebble layer. `#347 <https://github.com/canonical/kfp-operators/pull/347>`_
* Fixed ``self._cni_config_changed`` call and handled exceptions in istio-pilot. `#396 <https://github.com/canonical/istio-operators/pull/396>`_
* Corrected null values for configurations in ``spawner_ui_config.yaml`` for ``jupyter-ui``. `#361 <https://github.com/canonical/notebook-operators/pull/361>`_

Enhancements
------------

* Istio ingress can be configured with TLS using either a TLS certificate provider or by passing the TLS key and cert directly to the ``istio-pilot`` charm as a Juju seceret. 
* Set ``meshConfig.accessLogFile`` configuration for exposing logs in istio-pilot. `#371 <https://github.com/canonical/istio-operators/pull/371>`_
* Added ``csr-domain-name`` config option for istio-pilot. `#381 <https://github.com/canonical/istio-operators/pull/381>`_
* Added config for queue sidecar image to knative-serving charm. `#186 <https://github.com/canonical/knative-operators/pull/186>`_
* Added configuration support for Kubeflow notebooks page in the UI. `#345 <https://github.com/canonical/notebook-operators/pull/345>`_
* Added ``cluster-domain``, ``cull-idle-time`` and ``idleness-check-period`` config options to notebook-operator. `#372 <https://github.com/canonical/notebook-operators/pull/372>`_
* Removed the need for ``public-url`` configuration for ``dex-auth`` and ``oidc-gatekeeper``, and at the same time allow better integration with Dex connectors by allowing the configuration of the Dex issuer URL `#209 <https://github.com/canonical/dex-auth-operator/pull/209>`_ and `#163 <https://github.com/canonical/oidc-gatekeeper-operator/pull/163>`_.

Deprecated
----------

* Removed ``seldon-core`` from the bundle definition. This does not affect the deployment upgrade from version 1.8.


.. Customized links

.. _98-1: https://github.com/canonical/pipelines-rocks/issues/98
.. _98-2: https://github.com/canonical/kubeflow-rocks/issues/98

.. _99-1: https://github.com/canonical/pipelines-rocks/issues/99
.. _99-2: https://github.com/canonical/kubeflow-rocks/issues/99