.. _grafana_dashboards:

Grafana dashboards
==================

This guide presents the `Grafana <https://grafana.com>`_ dashboards provided by Charmed Kubeflow (CKF). 
See `Grafana dashboards <https://grafana.com/docs/grafana/latest/dashboards/>`_ for more details.

.. note::
   All Grafana dashboards provided by CKF use the ``ckf`` tag.

---------------------
Generic dashboards
---------------------

~~~~~~~~~~~~~~~~~~~
CKF charms state
~~~~~~~~~~~~~~~~~~~

This dashboard shows the state, ``up`` represented in green or ``down`` represented in red, of CKF charms. 
This includes only charms that provide metrics. 
See :ref:`Prometheus metrics <prometheus_metrics>` to learn which are those.

.. image:: https://assets.ubuntu.com/v1/6a05687d-ckf-gen-dashboard.png

~~~~~~~~~~~~~~~~~~~
Istio control plane
~~~~~~~~~~~~~~~~~~~

This dashboard provides a general overview of the health and performance of the `Istio control plane <https://istio.io/>`_. 
It combines metrics from ``istio-pilot`` and ``istio-gateway``.

See `Visualizing Istio metrics with Grafana <https://istio.io/latest/docs/tasks/observability/metrics/using-istio-dashboard/>`_ for more details.

.. image:: https://assets.ubuntu.com/v1/eb664b29-istio-control-plane.png

-----------
Pipelines
-----------

The following dashboards provide visualisations related to `Kubeflow Pipelines <https://www.kubeflow.org/docs/components/pipelines/>`_ (KFP).

~~~~~~~~~~~~~~~~~~~~~
ArgoWorkflow metrics
~~~~~~~~~~~~~~~~~~~~~

The metrics from the ``Argo Workflow`` controller expose the status of `Argo Workflow custom resources <https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/>`_, including the following information:

1. The number of workflows that have failed or are in ``error`` state.
2. The time workflows spend in the queue before being run.
3. The total size of captured logs that are pushed into S3 from the workflow pods.

.. image:: https://assets.ubuntu.com/v1/a4d1388d-argo_metrics1.png
.. image:: https://assets.ubuntu.com/v1/2031ef28-argo_metrics2.png
.. image:: https://assets.ubuntu.com/v1/cb89f0e6-argo_metrics3.png

~~~~~~~~~~~~~~~~~
Envoy service
~~~~~~~~~~~~~~~~~

The metrics from the ``envoy`` proxy expose the history of requests proxied from the KFP user interface to the `MLMD <https://www.kubeflow.org/docs/components/pipelines/concepts/metadata/>`_ application, including the following information:

1. The total number of requests.
2. The success rate of requests with status code ``non 5xx`` as well the number of requests with ``4xx response``, either upstream or downstream.

.. image:: https://assets.ubuntu.com/v1/df25758f-envoy_metrics1.png
.. image:: https://assets.ubuntu.com/v1/bcd0037b-envoy_metrics2.png

~~~~~~~~~~~~~~~~~~
MinIO dashboard
~~~~~~~~~~~~~~~~~~

The metrics from ``MinIO`` expose the status of the S3 storage instance used by KFP, including the following information:

1. S3 available space and storage capacity.
2. S3 traffic.
3. S3 API request errors and data transferred.
4. Node CPU, memory, file descriptors and IO usage.

.. image:: https://assets.ubuntu.com/v1/9a55fbb3-minio_metrics1.png
.. image:: https://assets.ubuntu.com/v1/8f12173e-minio_metrics2.png
.. image:: https://assets.ubuntu.com/v1/63bbf536-minio_metrics3.png

-------------
Notebooks
-------------

The following dashboards provide visualisations related to `Kubeflow Notebooks <https://www.kubeflow.org/docs/components/notebooks/>`_.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Jupyter Notebook controller
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The metrics from the ``Jupyter`` controller expose the status of `Jupyter Notebook custom resources <https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/>`_.

.. image:: https://assets.ubuntu.com/v1/3bd35f35-jupyternotebook_metrics1.png

--------------
Experiments
--------------

The following dashboards provide visualisations related to `Katib <https://www.kubeflow.org/docs/components/katib/>`_ experiments.

~~~~~~~~~~~~~~
Katib status
~~~~~~~~~~~~~~

The metrics from the ``Katib`` controller expose the status of `Experiment and Trial custom resources <https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/>`_.

.. image:: https://assets.ubuntu.com/v1/e30e40e3-katib_metrics1.png

-------------------
Serving models
-------------------

The following dashboards provide visualisations related to serving ML models.

~~~~~~~~~~~~~~~
Seldon Core
~~~~~~~~~~~~~~~

The metrics from the ``Seldon Core`` controller expose the status of `Seldon Deployment custom resources <https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/>`_, also called models, including information related to Seldon deployments currently available on the controller.

.. image:: https://assets.ubuntu.com/v1/0186bceb-seldon_metrics1.png