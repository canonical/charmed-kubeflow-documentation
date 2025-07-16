.. _integrate_cos:

Integrate with Canonical Observability Stack
============================================

This guide describes how Charmed Kubeflow (CKF) and the `Canonical Observability Stack <https://charmhub.io/topics/canonical-observability-stack>`_ (COS) 
can be integrated using `Juju`_. 
This integration enables monitoring Kubeflow deployments.

.. note::

   The COS integration allows CKF to collect and analyse control plane data. User namespace data, such as logs from model training, cannot be monitored.

---------------------
Requirements
---------------------

- A CKF deployment and access to the Kubeflow dashboard. See the :ref:`CKF getting started guide <get_started>` for more details.
- A COS deployment. See the `COS getting started on MicroK8s guide <https://charmhub.io/topics/canonical-observability-stack/tutorials/install-microk8s>`_ for more details. If Kubeflow has been already deployed, skip the ``Configure MicroK8s`` section of the guide.
- Minimum system requirements are at least 8 cores CPU processor, 64GB of RAM and 150GB of disk space.
- `MicroK8s`_, Juju, `yq <https://snapcraft.io/yq>`_, `jq <https://snapcraft.io/jq>`_ and ``curl``. See :ref:`CKF supported versions <supported_kubeflow_versions>` for more details about compatible versions of `Kubeflow <https://www.kubeflow.org/docs/releases/>`_ and Juju.

---------------------
Integrate with COS
---------------------

As per `COS best practices <https://charmhub.io/topics/canonical-observability-stack/reference/best-practices#juju-compatibility>`_, 
this guide assumes that COS and CKF are deployed each using their own controllers. 
This means that after the deployment, there is a ``kubeflow`` and a ``cos`` model. 
These have associated the ``kf-controller`` and ``cos-controller`` controllers, respectively. 
These are the default names for the controllers. Users can set any other name during the controller bootstrapping.

Integrating CKF with COS involves adding relations to `Prometheus <https://prometheus.io/>`_, 
that provides metrics and alerts, and to `Grafana <https://grafana.com/>`_, which provides dashboards. 
To avoid cross-model relations and ensure COS is accessible, Kubeflow components are related to COS through the `Grafana Agent <https://charmhub.io/grafana-agent-k8s>`_ charm. 
Data flows from CKF charms through the Grafana agent to COS.

.. note::

   All the code examples provided throughout this guide can be run from any host terminal with Juju installed in the host machine.

~~~~~~~~~~~~~~~~~~~~~
Deploy Grafana Agent
~~~~~~~~~~~~~~~~~~~~~

Within the ``kubeflow`` model, deploy the Grafana agent as follows:

.. code-block:: bash

   juju deploy -m kf-controller:kubeflow grafana-agent-k8s --channel=stable

.. _get_cos_urls:

~~~~~~~~~~~~~~~~~~~
Get COS URLs
~~~~~~~~~~~~~~~~~~~

You can navigate the User Interface (UI) of charms such as Grafana and Prometheus using `Catalogue <https://charmhub.io/catalogue-k8s>`_.

You can get their related URLs by running the following code:

.. code-block:: bash

   juju show-unit -m cos-controller:cos catalogue/0 --format json | jq '.[]."relation-info".[]."application-data".url | select (. != null)'

You should see something like this:

.. code-block:: bash

   "http://10.64.140.43/cos-grafana"
   "http://10.64.140.43/cos-prometheus-0"
   "http://10.64.140.43/cos-alertmanager"

For more information on COS URLs, see `Browse dashboards <https://charmhub.io/topics/canonical-observability-stack/tutorials/install-microk8s>`_.

~~~~~~~~~~~~~~~~~~~
Check connectivity
~~~~~~~~~~~~~~~~~~~

To check the Grafana agent to COS connectivity, try to access any of the COS URLs. 
For example, "prometheus", from within the Grafana agent:

.. code-block:: bash

   juju exec --unit grafana-agent-k8s/0 -m kf-controller:kubeflow 'curl -s <URL>'

If successful, you will get an OK response, similar to this one:

.. code-block:: bash

   juju exec --unit grafana-agent-k8s/0 -m kf-controller:kubeflow 'curl -s http://10.64.140.43/cos-prometheus-0/api/v1/status/runtimeinfo'
   {"status":"success","data":{"startTime":"2024-07-24T16:28:30.48462767Z","CWD":"/","reloadConfigSuccess":true,"lastConfigTime":"2024-07-24T16:28:30Z","corruptionCount":0,"goroutineCount":56,"GOMAXPROCS":16,"GOMEMLIMIT":9223372036854775807,"GOGC":"","GODEBUG":"","storageRetention":"15d or 819MiB204KiB819B"}}

~~~~~~~~~~~~~~~~~~~~~
Make offers from COS
~~~~~~~~~~~~~~~~~~~~~

You can make offers for Prometheus, Grafana and Loki from COS as follows:

.. code-block:: bash

   juju offer -c cos-controller cos.prometheus:receive-remote-write prometheus-receive-remote-write
   juju offer -c cos-controller cos.grafana:grafana-dashboard grafana-dashboards
   juju offer -c cos-controller cos.loki:logging loki-logging

.. note::

   If you've deployed COS with ``offers`` overlay, making offers is not necessary, since they already exist.

~~~~~~~~~~~~~~~~~~~~~~~~~~~
Consume Offers in Kubeflow
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Within the ``kubeflow`` model, you can consume the COS offers for Prometheus, Grafana and Loki as follows:

.. code-block:: bash

   juju consume -m kf-controller:kubeflow cos-controller:cos.prometheus-receive-remote-write
   juju consume -m kf-controller:kubeflow cos-controller:cos.grafana-dashboards
   juju consume -m kf-controller:kubeflow cos-controller:cos.loki-logging

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Connect Grafana agent to endpoints
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The Grafana agent can provide metrics, alerts, dashboards and logs to the COS via these three relation endpoints:

* `send-remote-write <https://charmhub.io/grafana-agent-k8s/integrations#send-remote-write>`_
* `grafana-dashboards-provider <https://charmhub.io/grafana-agent-k8s/integrations#grafana-dashboards-provider>`_
* `logging-provider <https://charmhub.io/grafana-agent-k8s/integrations#logging-provider>`_

You can tell the Grafana agent to provide those by consuming those offers as follows:

.. code-block:: bash

   juju integrate -m kf-controller:kubeflow grafana-agent-k8s:send-remote-write prometheus-receive-remote-write
   juju integrate -m kf-controller:kubeflow grafana-agent-k8s:grafana-dashboards-provider grafana-dashboards
   juju integrate -m kf-controller:kubeflow grafana-agent-k8s:logging-consumer loki-logging

Verify the relations for all offers are in place:

.. code-block:: bash

   juju status -m cos-controller:cos grafana-agent-k8s --relations

You should see ``1/1`` in the ``Connected`` column under ``Offers``.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Integrate with Prometheus
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You can provide charms metrics to Prometheus in COS by linking the CKF charms to the ``metrics-endpoint`` as follows:

.. code-block:: bash

   juju switch kf-controller:kubeflow
   juju integrate argo-controller:metrics-endpoint grafana-agent-k8s:metrics-endpoint
   juju integrate dex-auth:metrics-endpoint grafana-agent-k8s:metrics-endpoint
   juju integrate envoy:metrics-endpoint grafana-agent-k8s:metrics-endpoint
   juju integrate istio-ingressgateway:metrics-endpoint grafana-agent-k8s:metrics-endpoint
   juju integrate istio-pilot:metrics-endpoint grafana-agent-k8s:metrics-endpoint
   juju integrate jupyter-controller:metrics-endpoint grafana-agent-k8s:metrics-endpoint
   juju integrate katib-controller:metrics-endpoint grafana-agent-k8s:metrics-endpoint
   juju integrate katib-db:metrics-endpoint grafana-agent-k8s:metrics-endpoint
   juju integrate kfp-api:metrics-endpoint grafana-agent-k8s:metrics-endpoint
   juju integrate kfp-db:metrics-endpoint grafana-agent-k8s:metrics-endpoint
   juju integrate knative-operator:metrics-endpoint grafana-agent-k8s:metrics-endpoint
   juju integrate knative-eventing:otel-collector knative-operator:otel-collector
   juju integrate knative-serving:otel-collector knative-operator:otel-collector
   juju integrate kserve-controller:metrics-endpoint grafana-agent-k8s:metrics-endpoint
   juju integrate kubeflow-dashboard:metrics-endpoint grafana-agent-k8s:metrics-endpoint
   juju integrate kubeflow-profiles:metrics-endpoint grafana-agent-k8s:metrics-endpoint
   juju integrate metacontroller-operator:metrics-endpoint grafana-agent-k8s:metrics-endpoint
   juju integrate minio:metrics-endpoint grafana-agent-k8s:metrics-endpoint
   juju integrate pvcviewer-operator:metrics-endpoint grafana-agent-k8s:metrics-endpoint
   juju integrate tensorboard-controller:metrics-endpoint grafana-agent-k8s:metrics-endpoint
   juju integrate training-operator:metrics-endpoint grafana-agent-k8s:metrics-endpoint

Verify the relations are successfully added with:

.. code-block:: bash

   juju status --relations

~~~~~~~~~~~~~~~~~~~~~~
Integrate with Grafana
~~~~~~~~~~~~~~~~~~~~~~

You can link Kubeflow charms to the Grafana agent via the ``grafana-dashboards-consumer`` endpoint in COS as follows:

.. code-block:: bash

   juju switch kf-controller:kubeflow
   juju integrate argo-controller:grafana-dashboard grafana-agent-k8s:grafana-dashboards-consumer
   juju integrate dex-auth:grafana-dashboard grafana-agent-k8s:grafana-dashboards-consumer
   juju integrate envoy:grafana-dashboard grafana-agent-k8s:grafana-dashboards-consumer
   juju integrate istio-pilot:grafana-dashboard grafana-agent-k8s:grafana-dashboards-consumer
   juju integrate jupyter-controller:grafana-dashboard grafana-agent-k8s:grafana-dashboards-consumer
   juju integrate katib-controller:grafana-dashboard grafana-agent-k8s:grafana-dashboards-consumer
   juju integrate katib-db:grafana-dashboard grafana-agent-k8s:grafana-dashboards-consumer
   juju integrate kfp-api:grafana-dashboard grafana-agent-k8s:grafana-dashboards-consumer
   juju integrate kfp-db:grafana-dashboard grafana-agent-k8s:grafana-dashboards-consumer
   juju integrate kubeflow-dashboard:grafana-dashboard grafana-agent-k8s:grafana-dashboards-consumer
   juju integrate metacontroller-operator:grafana-dashboard grafana-agent-k8s:grafana-dashboards-consumer
   juju integrate minio:grafana-dashboard grafana-agent-k8s:grafana-dashboards-consumer
   juju integrate pvcviewer-operator:grafana-dashboard grafana-agent-k8s:grafana-dashboards-consumer
   juju integrate training-operator:grafana-dashboard grafana-agent-k8s:grafana-dashboards-consumer

Verify the relations are successfully added:

.. code-block:: bash

   juju status --relations

~~~~~~~~~~~~~~~~~~~
Integrate with Loki
~~~~~~~~~~~~~~~~~~~

.. note::

   Log forwarding to Loki is available from CKF 1.9.

You can provide charms logs to Loki in COS by integrating the CKF charms with ``loki-logging`` endpoint and Grafana agent as follows:

.. code-block:: bash

   juju switch kf-controller:kubeflow
   juju integrate admission-webhook:logging grafana-agent-k8s:logging-provider
   juju integrate argo-controller:logging grafana-agent-k8s:logging-provider
   juju integrate dex-auth:logging grafana-agent-k8s:logging-provider
   juju integrate envoy:logging grafana-agent-k8s:logging-provider
   juju integrate jupyter-controller:logging grafana-agent-k8s:logging-provider
   juju integrate jupyter-ui:logging grafana-agent-k8s:logging-provider
   juju integrate katib-controller:logging grafana-agent-k8s:logging-provider
   juju integrate katib-db-manager:logging grafana-agent-k8s:logging-provider
   juju integrate katib-db:logging grafana-agent-k8s:logging-provider
   juju integrate katib-ui:logging grafana-agent-k8s:logging-provider
   juju integrate kfp-api:logging grafana-agent-k8s:logging-provider
   juju integrate kfp-db:logging grafana-agent-k8s:logging-provider
   juju integrate kfp-metadata-writer:logging grafana-agent-k8s:logging-provider
   juju integrate kfp-persistence:logging grafana-agent-k8s:logging-provider
   juju integrate kfp-profile-controller:logging grafana-agent-k8s:logging-provider
   juju integrate kfp-schedwf:logging grafana-agent-k8s:logging-provider
   juju integrate kfp-ui:logging grafana-agent-k8s:logging-provider
   juju integrate kfp-viewer:logging grafana-agent-k8s:logging-provider
   juju integrate kfp-viz:logging grafana-agent-k8s:logging-provider
   juju integrate knative-operator:logging grafana-agent-k8s:logging-provider
   juju integrate kserve-controller:logging grafana-agent-k8s:logging-provider
   juju integrate kubeflow-dashboard:logging grafana-agent-k8s:logging-provider
   juju integrate kubeflow-profiles:logging grafana-agent-k8s:logging-provider
   juju integrate kubeflow-volumes:logging grafana-agent-k8s:logging-provider
   juju integrate mlmd:logging grafana-agent-k8s:logging-provider
   juju integrate oidc-gatekeeper:logging grafana-agent-k8s:logging-provider
   juju integrate pvcviewer-operator:logging grafana-agent-k8s:logging-provider
   juju integrate tensorboard-controller:logging grafana-agent-k8s:logging-provider
   juju integrate tensorboards-web-app:logging grafana-agent-k8s:logging-provider

Verify the relations are successfully added with:

.. code-block:: bash

   juju status --relations

---------------------------
Access monitoring resources
---------------------------

Using COS URLs, you can access Prometheus and Grafana to monitor resources, including metrics, alerts and dashboards from CKF charms.

~~~~~~~~~~~~~~~~~~~
Prometheus metrics
~~~~~~~~~~~~~~~~~~~

Navigate to the Prometheus URL (see `Get COS URLs`_ for more details).

To view the metrics for a specific charm, query ``{juju_application="<app-name>"}``.

For example, you can check the ``argo-controller`` logs using this query:

.. code-block:: bash

   argo_workflows_count{juju_model="kubeflow", juju_charm="argo-controller"}

It returns all the charm-related metrics:

.. image:: https://assets.ubuntu.com/v1/87a6a16b-prometheus_metrics.png

To view all the metrics available from Prometheus, use the metrics explorer by clicking on the round icon next to ``Execute`` in that query form.

See :ref:`Prometheus metrics <prometheus_metrics>` for more details on available metrics.

~~~~~~~~~~~~~~~~~~~
Prometheus alerts
~~~~~~~~~~~~~~~~~~~

Navigate to the Prometheus URL and click on ``Alerting`` under the left-hand side navigation bar. This shows all available alerts.

.. image:: https://assets.ubuntu.com/v1/08b0c10d-prometheus_alerts.png

You can filter from ``Active``, ``Pending`` and ``Firing`` alerts using the available checkboxes.

To view alerts for a specific charm, type its name in the search bar on the top.

See :ref:`Prometheus alerts <prometheus_alerts>` for more details.

~~~~~~~~~~~~~~~~~~~
Grafana dashboards
~~~~~~~~~~~~~~~~~~~

Navigate to the Grafana dashboard URL (see `Get COS URLs`_ for more details).

Get the admin password:

.. code-block:: bash

   juju run -m cos-controller:cos grafana/leader get-admin-password

Using ``admin`` as the username, log in with the password returned.

See the available dashboards by clicking on ``Dashboards`` in the sidebar menu:

.. image:: https://assets.ubuntu.com/v1/b8f3ec4d-grafana_dashboards.png

See :ref:`Grafana dashboards <grafana_dashboards>` for more information.

~~~~~~~~~~~~~~~~~~~
Loki logs
~~~~~~~~~~~~~~~~~~~

Loki does not provide a UI. You can use the Grafana UI for checking Loki logs.

Navigate to the Grafana URL and click on ``Explore``, where you can navigate through all collected logs by selecting Loki as source.

.. image:: https://assets.ubuntu.com/v1/c4eb2044-loki_logs.png

See `Visualize log data <https://grafana.com/docs/loki/latest/visualize/grafana/>`_ for more details on navigating Grafana Loki.

For more information on forwarded logs, see :ref:`Loki logs <loki_logs>`.


