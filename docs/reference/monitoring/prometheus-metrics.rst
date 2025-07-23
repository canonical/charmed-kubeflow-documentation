.. _prometheus_metrics:

Prometheus metrics
==================

This guide presents an overview of the Charmed Kubeflow (CKF) charms that provide `Prometheus <https://prometheus.io/>`_ monitoring metrics.

All metrics can be accessed using the Prometheus or `Grafana <https://grafana.com/>`_ User Interface (UI). 
See :ref:`Integrate with COS <integrate_cos>` for more information.

----------------
Argo controller
----------------

See `Argo controller upstream documentation <https://argo-workflows.readthedocs.io/en/stable/metrics/#default-controller-metrics>`_ for more information on provided metrics.

You can check its metrics through the Prometheus or Grafana UI using the following query:

.. code-block:: bash

    {juju_charm="argo-controller"}

-------------
Dex Auth
-------------

The ``dex-auth`` charm provides:

* A custom metric counting HTTP requests. See `Dex Auth source code <https://github.com/dexidp/dex/blob/87553087591db31de96953fed038b2447924b3af/server/server.go#L332-L336>`_ for more details.
* `Go runtime and process metrics <https://pkg.go.dev/runtime/metrics#hdr-Supported_metrics>`_ for monitoring the controller.
* gRPC server metrics.

You can check its metrics through the Prometheus or Grafana UI using the following query:

.. code-block:: bash

    {juju_charm="dex-auth"}

--------
Envoy
--------

The ``envoy`` charm provides the following metrics:

* `Listener <https://www.envoyproxy.io/docs/envoy/v1.27.5/configuration/listeners/stats>`_.
* `HTTP connection manager <https://www.envoyproxy.io/docs/envoy/v1.27.5/configuration/http/http_conn_man/stats>`_.
* `Cluster manager <https://www.envoyproxy.io/docs/envoy/v1.27.5/configuration/upstream/cluster_manager/cluster_stats>`_.
* `Server <https://www.envoyproxy.io/docs/envoy/v1.27.5/configuration/observability/statistics>`_.
* `Access logs <https://www.envoyproxy.io/docs/envoy/v1.27.5/configuration/observability/access_log/stats>`_.

You can check its metrics through the Prometheus or Grafana UI using the following query:

.. code-block:: bash

    {juju_charm="envoy"}

-----------------
Istio pilot
-----------------

See `Istio pilot upstream documentation <https://istio.io/latest/docs/concepts/observability/>`_ for more information on provided metrics.

You can check its metrics through the Prometheus or Grafana UI using the following query:

.. code-block:: bash

    {juju_charm="istio-pilot"}

------------------
Istio gateway
------------------

See `Istio gateway upstream documentation <https://istio.io/latest/docs/concepts/observability/>`_ for more information on provided metrics.

You can check its metrics through the Prometheus or Grafana UI using the following query:

.. code-block:: bash

    {juju_charm="istio-gateway"}

-----------------------
Jupyter controller
-----------------------

The ``jupyter-controller`` provides the following metrics:

* Custom notebook-related metrics. See `Jupyter controller source code <https://github.com/kubeflow/kubeflow/blob/bd7f250df22e144b114177536309d28651b4ddbb/components/notebook-controller/pkg/metrics/metrics.go#L25-L58>`_ for more details.
* `Go runtime metrics <https://pkg.go.dev/runtime/metrics#hdr-Supported_metrics>`_ for monitoring the controller.
* `Controller runtime <https://book.kubebuilder.io/reference/metrics-reference>`_ metrics.

You can check its metrics through the Prometheus or Grafana UI using the following query:

.. code-block:: bash

    {juju_charm="jupyter-controller"}

--------------------
Katib controller
--------------------

The ``katib`` controller provides the following metrics:

* Custom experiment-related metrics. See `Katib controller source code <https://github.com/kubeflow/katib/blob/ea46a7f2b73b2d316b6b7619f99eb440ede1909b/pkg/controller.v1beta1/experiment/util/prometheus_metrics.go#L39-L62>`_ for more details.
* `Go runtime metrics <https://pkg.go.dev/runtime/metrics#hdr-Supported_metrics>`_ for monitoring the controller.
* `Controller runtime <https://book.kubebuilder.io/reference/metrics-reference>`_ metrics.

You can check its metrics through the Prometheus or Grafana UI using the following query:

.. code-block:: bash

    {juju_charm="katib-controller"}

-------------
Kfp api
-------------

The ``kfp-api`` provides the following metrics:

* Custom metrics related to its several components. See its source code for more details:
* `Resource manager <https://github.com/kubeflow/pipelines/blob/33db1284f57b5b277c95d4a44b35b1fdd830bd18/backend/src/apiserver/resource/resource_manager.go>`_.
* `Experiment server <https://github.com/kubeflow/pipelines/blob/33db1284f57b5b277c95d4a44b35b1fdd830bd18/backend/src/apiserver/server/experiment_server.go>`_.
* `Job server <https://github.com/kubeflow/pipelines/blob/33db1284f57b5b277c95d4a44b35b1fdd830bd18/backend/src/apiserver/server/job_server.go>`_.
* `Pipeline server <https://github.com/kubeflow/pipelines/blob/33db1284f57b5b277c95d4a44b35b1fdd830bd18/backend/src/apiserver/server/pipeline_server.go>`_.
* `Pipeline upload <https://github.com/kubeflow/pipelines/blob/33db1284f57b5b277c95d4a44b35b1fdd830bd18/backend/src/apiserver/server/pipeline_upload_server.go>`_.
* `Run server <https://github.com/kubeflow/pipelines/blob/33db1284f57b5b277c95d4a44b35b1fdd830bd18/backend/src/apiserver/server/run_server.go>`_.
* `Go runtime and process metrics <https://pkg.go.dev/runtime/metrics#hdr-Supported_metrics>`_ for monitoring the controller.

You can check its metrics through the Prometheus or Grafana UI using the following query:

.. code-block:: bash

    {juju_charm="kfp-api"}

-----------------------
Knative eventing
-----------------------

The ``knative-eventing`` metrics come from the ``knative-operator`` charm that deploys `otel-collector <https://opentelemetry.io/docs/collector/>`_. 
See `Knative eventing upstream documentation <https://knative.dev/docs/eventing/observability/metrics/eventing-metrics/>`_ for more details.

You can check its metrics through the Prometheus or Grafana UI using the following query:

.. code-block:: bash

    {juju_charm="knative-operator", namespace_name="knative-eventing"}

-----------------------
Knative serving
-----------------------

The ``knative-serving`` metrics come from the ``knative-operator`` charm that deploys `otel-collector <https://opentelemetry.io/docs/collector/>`_. 
See `Knative serving upstream documentation <https://knative.dev/docs/eventing/observability/metrics/eventing-metrics/>`_ for more details.

You can check its metrics through the Prometheus or Grafana UI using the following query:

.. code-block:: bash

    {juju_charm="knative-operator", namespace_name="knative-serving"}

------------------------
Knative operator
------------------------

See `Knative operator upstream documentation <https://knative.dev/docs/serving/observability/metrics/collecting-metrics/>`_ for more information on provided metrics.

You can check its metrics through the Prometheus or Grafana UI using the following query:

.. code-block:: bash

    {juju_charm="knative-operator"}

-----------------------------
Metacontroller operator
-----------------------------

The ``metacontroller-operator`` provides the following metrics:

* Custom metrics. See `Metacontroller source code <https://github.com/metacontroller/metacontroller/blob/f54c2335e938cabfe3c15932ac721a2f1408d9c6/pkg/metrics/http.go#L113-L165>`_ for more details.
* `Go runtime and process metrics <https://pkg.go.dev/runtime/metrics#hdr-Supported_metrics>`_ for monitoring the controller.
* `Controller runtime <https://book.kubebuilder.io/reference/metrics-reference>`_ metrics.

You can check its metrics through the Prometheus or Grafana UI using the following query:

.. code-block:: bash

    {juju_charm="metacontroller-operator"}

--------
Minio
--------

See `Minio upstream documentation <https://min.io/docs/minio/kubernetes/upstream/operations/monitoring/metrics-and-alerts.html>`_ for more information on provided metrics.

You can check its metrics through the Prometheus or Grafana UI using the following query: 

.. code-block:: bash

    {juju_charm="minio"}

-----------------------------
Seldon controller manager
-----------------------------

See `Seldon controller manager upstream documentation <https://docs.seldon.io/>`_ for more information on provided metrics.

You can check its metrics through the Prometheus or Grafana UI using the following query:

.. code-block:: bash

    {juju_charm="seldon-controller-manager"}

-------------------
Training operator
-------------------

The ``training-operator`` provides the following metrics:

* Custom job-related metrics. See `Training operator source code <https://github.com/kubeflow/trainer>`_ for more details.
* `Go runtime and process metrics <https://pkg.go.dev/runtime/metrics#hdr-Supported_metrics>`_ for monitoring the controller.
* `Controller runtime <https://book.kubebuilder.io/reference/metrics-reference>`_ metrics.

You can check its metrics through the Prometheus or Grafana UI using the following query:

.. code-block:: bash

    {juju_charm="training-operator"}

-----------------------
Pvcviewer operator
-----------------------

The ``pvcviewer-operator`` provides the following metrics:

* `Go runtime and process metrics <https://pkg.go.dev/runtime/metrics#hdr-Supported_metrics>`_ for monitoring the controller.
* `Controller runtime <https://book.kubebuilder.io/reference/metrics-reference>`_ metrics.

You can check its metrics through the Prometheus or Grafana UI using the following query:

.. code-block:: bash

    {juju_charm="pvcviewer-operator"}

----------------------
Kserve controller
----------------------

The ``kserve-controller`` provides the following metrics:

* `Go runtime and process metrics <https://pkg.go.dev/runtime/metrics#hdr-Supported_metrics>`_ for monitoring the controller.
* `Controller runtime <https://book.kubebuilder.io/reference/metrics-reference>`_ metrics.

You can check its metrics through the Prometheus or Grafana UI using the following query:

.. code-block:: bash

    {juju_charm="kserve-controller"}

---------------------
Kubeflow profiles
---------------------

Kubeflow profiles manage two `Pebble <https://juju.is/docs/sdk/pebble>`_ services:

* ``profile-controller``.
* ``kfam``.

~~~~~~~~~~~~~~~~~~~~
Profile controller
~~~~~~~~~~~~~~~~~~~~

The ``profile-controller`` provides the following metrics:

* Custom job-related metrics. See `Profile controller source code <https://github.com/kubeflow/kubeflow/tree/48b8643bee14b8c85c3de9f6d129752bb55b44d3/components/profile-controller>`_ for more details.
* `Go runtime and process metrics <https://pkg.go.dev/runtime/metrics#hdr-Supported_metrics>`_ for monitoring the controller.

You can check its metrics through the Prometheus or Grafana UI using the following query:

.. code-block:: bash

    {juju_charm="kubeflow-profiles"}

~~~~~~~
Kfam
~~~~~~~

The ``kfam`` provides the following metrics:

* Custom job-related metrics. See `Kfam source code <https://github.com/kubeflow/kubeflow/blob/48b8643bee14b8c85c3de9f6d129752bb55b44d3/components/access-management/kfam/monitoring.go#L24C1-L44C2>`_ for more details.
* `Go runtime and process metrics <https://pkg.go.dev/runtime/metrics#hdr-Supported_metrics>`_ for monitoring the controller.

You can check its metrics through the Prometheus or Grafana UI using the following query:

.. code-block:: bash

    {juju_charm="kubeflow-profiles"}

-----------------------------
Tensorboard controller
-----------------------------

The ``tensorboard-controller`` provides the following metrics:

* `Go runtime and process metrics <https://pkg.go.dev/runtime/metrics#hdr-Supported_metrics>`_ for monitoring the controller.
* `Controller runtime <https://book.kubebuilder.io/reference/metrics-reference>`_ metrics.

You can check its metrics through the Prometheus or Grafana UI using the following query:

.. code-block:: bash

    {juju_charm="tensorboard-controller"}
