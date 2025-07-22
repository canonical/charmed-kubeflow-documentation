:tocdepth: 3

.. _loki_logs:

Loki logs
=========

This guide presents the available Charmed Kubeflow (CKF) `Loki <https://grafana.com/oss/loki/>`_ logs. 

The CKF charms that use the sidecar pattern can provide log information from its workloads. 
Those that do not use it need to leverage `Promtail <https://grafana.com/docs/loki/latest/send-data/promtail/>`_ for doing so.

Loki does not provide a User Interface (UI). You can use the Grafana UI for checking Loki logs. 
See `Visualize log data <https://grafana.com/docs/loki/latest/visualize/grafana/>`_ for more details on navigating Grafana Loki. 

-----------------------
Sidecar pattern charms
-----------------------

The following subsections present CKF charms that use the sidecar pattern and can provide log information from their workloads. 

~~~~~~~~~~~~~~~~~~~
Admission-webhook
~~~~~~~~~~~~~~~~~~~

``Admission-webhook`` is a Go application that uses `k8s.io/klog <https://pkg.go.dev/k8s.io/klog>`_ for logging. 

You can check its logs through the Grafana UI using the query ``{pebble_service="admission-webhook", charm="admission-webhook"}``.

See `Admission-webhook logs source <https://github.com/kubeflow/kubeflow/tree/master/components/admission-webhook>`_ for more details.

~~~~~~~~~~~~~~~~~~~
Argo-controller
~~~~~~~~~~~~~~~~~~~

``Argo-controller`` is a GO application that uses `logrus <https://github.com/sirupsen/logrus>`_ for logging.

You can check its logs through the Grafana UI using the query ``{pebble_service="argo-controller", charm="argo-controller"}``.

See `Argo-controller logs source <https://github.com/argoproj/argo-workflows/blob/main/cmd/workflow-controller>`_ for more details.

~~~~~~~~~~~~~~~~~~~
Dex-auth
~~~~~~~~~~~~~~~~~~~

``Dex-auth`` is a GO application that uses `slog <https://pkg.go.dev/log/slog>`_ for logging.

You can check its logs through the Grafana UI using the query ``{pebble_service="dex", charm="dex-auth"}``

See `Dex-auth logs source <https://github.com/dexidp/dex>`_ for more details.

~~~~~~~~~~~~~~~~~~~
Envoy
~~~~~~~~~~~~~~~~~~~

``Envoy`` is a C++ application that uses standard ``spdlog`` library for logging.

You can check its logs through the Grafana UI using the query ``{pebble_service="envoy", charm="envoy"}``.

See `Envoy logs source <https://github.com/kubeflow/pipelines/tree/master/third_party/metadata_envoy>`_ for more details. 
Since this is a third-party application, you will only see the configuration of `envoyproxy <https://www.envoyproxy.io/>`_.

~~~~~~~~~~~~~~~~~~~
Jupyter-controller
~~~~~~~~~~~~~~~~~~~

``Jupyter-controller`` is a GO application that uses `controller-runtime/pkg/log <https://pkg.go.dev/sigs.k8s.io/controller-runtime/pkg/log>`_ for logging.

You can check its logs through the Grafana UI using the query ``{pebble_service="jupyter-controller", charm="jupyter-controller"}``.

See `Jupyter-controller logs source <https://github.com/kubeflow/kubeflow/tree/master/components/notebook-controller>`_ for more details.

~~~~~~~~~~~~~~~~~~~
Jupyter-ui
~~~~~~~~~~~~~~~~~~~

``Jupyter-ui`` is a Python application that uses the `logging <https://docs.python.org/3/library/logging.html>`_ library for logging.

You can check its logs through the Grafana UI using the query ``{pebble_service="jupyter-ui", charm="jupyter-ui"}``.

See `Jupyter-ui logs source <https://github.com/kubeflow/kubeflow/tree/master/components/crud-web-apps/common/backend>`_ for more details. 

~~~~~~~~~~~~~~~~~~~
Katib-controller
~~~~~~~~~~~~~~~~~~~

``Katib-controller`` is a GO application that uses `controller-runtime/pkg/log <https://pkg.go.dev/sigs.k8s.io/controller-runtime/pkg/log>`_ for logging.

You can check its logs through the Grafana UI using the query ``{pebble_service="katib-controller", charm="katib-controller"}``.

See `Katib-controller logs source <https://github.com/kubeflow/katib/tree/master/cmd/katib-controller/v1beta1>`_ for more details.

~~~~~~~~~~~~~~~~~~~
Katib-db-manager
~~~~~~~~~~~~~~~~~~~

``Katib-db-manager`` is a Go application that uses `k8s.io/klog <https://pkg.go.dev/k8s.io/klog>`_ for logging.

You can check its logs  through the Grafana UI using the query ``{pebble_service="katib-db-manager", charm="katib-db-manager"}``.

See `Katib-db-manager logs source <https://github.com/kubeflow/katib/tree/master/cmd/db-manager/v1beta1>`_ for more details.

~~~~~~~~~~~~~~~~~~~
Katib-ui
~~~~~~~~~~~~~~~~~~~

``Katib-ui`` is a Go application that uses `log <https://pkg.go.dev/log>`_ library for logging.

You can check its logs through the Grafana UI using the query ``{pebble_service="katib-ui", charm="katib-ui"}``.

See `Katib-ui logs source <https://github.com/kubeflow/katib/tree/master/cmd/ui>`_ for more details.

~~~~~~~~~~~~~~~~~~~
Kfp-api
~~~~~~~~~~~~~~~~~~~

``Kfp-api`` is a GO application that uses `logrus <https://github.com/sirupsen/logrus>`_ for logging.

You can check its logs through the Grafana UI using the query ``{pebble_service="apiserver", charm="kfp-api"}``.

See `Kfp-api logs source <https://github.com/kubeflow/pipelines/tree/master/backend/src/apiserver>`_ for more details.

~~~~~~~~~~~~~~~~~~~
Kfp-metadata-writer
~~~~~~~~~~~~~~~~~~~

``Kfp-metadata-writer`` is a Python application that uses ``print`` for logging.

You can check its logs through the Grafana UI using the query ``{pebble_service="kfp-metadata-writer", charm="kfp-metadata-writer"}``.

See `Kfp-metadata-writer logs source <https://github.com/kubeflow/pipelines/blob/master/backend/metadata_writer>`_ for more details.

~~~~~~~~~~~~~~~~~~~
Kfp-persistence
~~~~~~~~~~~~~~~~~~~

``Kfp-persistence`` is a GO application that uses `logrus <https://github.com/sirupsen/logrus>`_ for logging.

You can check its logs through the Grafana UI using the query ``{pebble_service="persistenceagent", charm="kfp-persistence"}``.

See `Kfp-persistence logs source <https://github.com/kubeflow/pipelines/blob/master/backend/src/agent/persistence>`_ for more details.

~~~~~~~~~~~~~~~~~~~~~~
Kfp-profile-controller
~~~~~~~~~~~~~~~~~~~~~~

``Kfp-profile-controller`` is a GO application that uses `controller-runtime <https://pkg.go.dev/sigs.k8s.io/controller-runtime>`_ for logging.

You can check its logs through the Grafana UI using the query ``{pebble_service="kfp-profile-controller", charm="kfp-profile-controller"}``.

See `Kfp-profile-controller logs source <https://github.com/kubeflow/kubeflow/tree/master/components/profile-controller>`_ for more details.

~~~~~~~~~~~~~~~~~~~
Kfp-schedwf
~~~~~~~~~~~~~~~~~~~

``Kfp-schedwf`` is a GO application that uses `logrus <https://github.com/sirupsen/logrus>`_ for logging.

You can check its logs through the Grafana UI using the query ``{pebble_service="controller", charm="kfp-schedwf"}``.

See `Kfp-schedwf logs source <https://github.com/kubeflow/pipelines/tree/master/backend/src/crd/controller/scheduledworkflow>`_ for more details.

~~~~~~~~~~~~~~~~~~~
Kfp-ui
~~~~~~~~~~~~~~~~~~~

``Kfp-ui`` is a TypeScript application that uses ``console.log`` for logging.

You can check its logs through the Grafana UI using the query ``{pebble_service="ml-pipeline-ui", charm="kfp-ui"}``.

See `Kfp-ui logs source <https://github.com/kubeflow/pipelines/tree/master/frontend/server>`_ for more details.

~~~~~~~~~~~~~~~~~~~
Kfp-viewer
~~~~~~~~~~~~~~~~~~~

``Kfp-viewer`` is a Go application that uses `glog <https://pkg.go.dev/github.com/golang/glog>`_ for logging.

You can check its logs through the Grafana UI using the query ``{pebble_service="controller", charm="kfp-viewer"}``.

See `Kfp-viewer logs source <https://github.com/kubeflow/pipelines/tree/master/backend/src/crd/controller/viewer>`_ for more details.

~~~~~~~~~~~~~~~~~~~
Kfp-viz
~~~~~~~~~~~~~~~~~~~

``Kfp-viz`` is a Python application that uses the `Tornado <https://www.tornadoweb.org/en/stable/index.html>`_ framework for logging.

You can check its logs through the Grafana UI using the query ``{pebble_service="vis-server", charm="kfp-viz"}``.

See `Kfp-viz logs source <https://github.com/kubeflow/pipelines/tree/master/backend/src/apiserver/visualization>`_ for more details.

~~~~~~~~~~~~~~~~~~~
Knative-operator
~~~~~~~~~~~~~~~~~~~

``Knative-operator`` comes with two workloads containers and both are a GO application that uses `go-kit/log <https://pkg.go.dev/github.com/go-kit/log>`_ for logging.

You can check its logs through the Grafana UI using the query ``{pebble_service="knative-operator", charm="knative-operator"}`` and ``{pebble_service="knative-operator-webhook", charm="knative-operator"}``.

See `Knative-operator logs source <https://github.com/knative/operator>`_ for more details.

~~~~~~~~~~~~~~~~~~~
Kserve-controller
~~~~~~~~~~~~~~~~~~~

``Kserve-controller`` is a Go application that uses `k8s.io/klog <https://pkg.go.dev/k8s.io/klog>`_ for logging. 
This app also uses `kube-rbac-proxy <https://github.com/brancz/kube-rbac-proxy>`_.

You can check its logs through the Grafana UI using the query ``{pebble_service="kserve-controller", charm="kserve-controller"}`` and ``{pebble_service="kube-rbac-proxy", charm="kserve-controller"}``

See `Kserve-controller logs source <https://github.com/knative/operator>`_ for more details.

~~~~~~~~~~~~~~~~~~~
Kubeflow-dashboard
~~~~~~~~~~~~~~~~~~~

``Kubeflow-dashboard`` is a TypeScript application that uses ``console.log`` for logging.

You can check its logs through the Grafana UI using the query ``{pebble_service="kubeflow-dashboard", charm="kubeflow-dashboard"}``.

See `Kubeflow-dashboard logs source <https://github.com/kubeflow/kubeflow/blob/master/components/centraldashboard>`_ for more details.

~~~~~~~~~~~~~~~~~~~
Kubeflow-profiles
~~~~~~~~~~~~~~~~~~~

``Kubeflow-profiles`` is a GO application that uses `logrus <https://github.com/sirupsen/logrus>`_ for logging.

You can check its logs through the Grafana UI using the query ``{pebble_service="kubeflow-kfam", charm="kubeflow-profiles"}``.

See `Kubeflow-profiles logs source <https://github.com/kubeflow/kubeflow/tree/master/components/profile-controller>`_ for more details.

~~~~~~~~~~~~~~~~~~~
Kubeflow-volumes
~~~~~~~~~~~~~~~~~~~

``Kubeflow-volumes`` is a Python application that uses the `logging <https://docs.python.org/3/library/logging.html>`_ library for logging.

You can check its logs through the Grafana UI using the query ``{pebble_service="kubeflow-volumes", charm="kubeflow-volumes"}``.

See `Kubeflow-volumes logs source <https://github.com/kubeflow/kubeflow/tree/master/components/crud-web-apps/volumes>`_ for more details.

~~~~~~~~~~~~~~~~~~~
Mlmd
~~~~~~~~~~~~~~~~~~~

``Mlmd`` is a C++ application that uses the `google/glog <https://github.com/google/glog>`_ library for logging.

You can check its logs through the Grafana UI using the query ``{pebble_service="mlmd", charm="mlmd"}``.

See `Mlmd logs source <https://github.com/google/ml-metadata/tree/master/ml_metadata>`_ for more details.

~~~~~~~~~~~~~~~~~~~
Oidc-gatekeeper
~~~~~~~~~~~~~~~~~~~

``Oidc-gatekeeper`` is a GO application that uses `logrus <https://github.com/sirupsen/logrus>`_ for logging.

You can check its logs through the Grafana UI using the query ``{pebble_service="oidc-authservice", charm="oidc-gatekeeper"}``.

See `Oidc-gatekeeper logs source <https://github.com/arrikto/oidc-authservice/blob/master>`_ for more details.

~~~~~~~~~~~~~~~~~~~~~~~~
Pvcviewer-operator
~~~~~~~~~~~~~~~~~~~~~~~~

``Pvcviewer-operator`` is a GO application that uses `controller-runtime/pkg/log <https://pkg.go.dev/sigs.k8s.io/controller-runtime/pkg/log>`_ for logging.

You can check its logs through the Grafana UI using the query ``{pebble_service="pvcviewer-operator", charm="pvcviewer-operator"}``.

See `Pvcviewer-operator logs source <https://github.com/kubeflow/kubeflow/tree/master/components/pvcviewer-controller>`_ for more details.

~~~~~~~~~~~~~~~~
Seldon-core
~~~~~~~~~~~~~~~~

``Seldon-core`` is a GO application that uses `controller-runtime/pkg/log <https://pkg.go.dev/sigs.k8s.io/controller-runtime/pkg/log>`_ for logging.

You can check its logs through the Grafana UI using the query  ``{pebble_service="seldon-core", charm="seldon-core"}``.

See `Seldon-core logs source <https://github.com/SeldonIO/seldon-core/tree/master/operator>`_ for more details.

~~~~~~~~~~~~~~~~~~~~~~~~~~~
Tensorboard-controller
~~~~~~~~~~~~~~~~~~~~~~~~~~~

``Tensorboard-controller`` is a GO application that uses `controller-runtime/pkg/log <https://pkg.go.dev/sigs.k8s.io/controller-runtime/pkg/log>`_ for logging.

You can check its logs through the Grafana UI using the query  ``{pebble_service="pvcviewer-operator", charm="pvcviewer-operator"}``.

See `Tensorboard-controller logs source <https://github.com/kubeflow/kubeflow/tree/master/components/tensorboard-controller>`_ for more details.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Tensorboards-web-app
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

``Tensorboards-web-app`` is a Python application that uses the `logging <https://docs.python.org/3/library/logging.html>`_ library for logging.

You can check its logs through the Grafana UI using the query ``{pebble_service="tensorboards-web-app", charm="tensorboards-web-app"}``.

See `Tensorboards-web-app logs source <https://github.com/kubeflow/kubeflow/tree/master/components/crud-web-apps/tensorboards>`_ for more details. 

--------------------------
Non sidecar pattern charms
--------------------------

The following CKF charms do not use the sidecar pattern and cannot use the log forwarding principle:

* Istio-gateway
* Istio-pilot
* Knative-eventing
* Knative-serving
* Metacontroller
* Training-operator

For monitoring these charms, you can use `Promtail <https://grafana.com/docs/loki/latest/send-data/promtail/>`_.

.. _promtail_configuration:

~~~~~~~~~~~~~~~~~~~~~~~~
Promtail configuration
~~~~~~~~~~~~~~~~~~~~~~~~

You can configure Promtail for a more efficient log collection, avoiding scraping all logs within the clusters and adding the correct Juju topology.

To do so, you need to configure the following: 

1. Clients section
1. Scrape jobs configuration

^^^^^^^^^^^^^^^
Clients section
^^^^^^^^^^^^^^^

The clients section requires adding:

* The ``<URL>`` to Loki.
* The Juju model name ``<JUJU_MODEL>``.
* The Juju model uuid ``<JUJU_MODEL_UUID>``. 

.. code-block:: bash

  clients:
    - url: <URL>
      external_labels:
        juju_model: <JUJU_MODEL>
        juju_model_uuid: <JUJU_MODEL_UUID>

The URL is required for accessing the Loki server. The external labels are part of the Juju topology required for querying logs. 

The Loki URL can be obtained as follows: 

.. code-block:: bash

  juju show-unit grafana-agent-k8s/0 -m cos-controller:cos --endpoint logging-consumer | yq '.[]."relation-info".[]."related-units".[].data.endpoint | fromjson | .url'

The Juju model name is always ``kubeflow`` in this use case. You can obtain the Juju model uuid as follows:

.. code-block:: bash

  juju models --format json | jq '.models.[] | select(."short-name"=="kubeflow") | ."model-uuid"'

^^^^^^^^^^^^^^^^^^^^^^^^^^
Scrape jobs configuration
^^^^^^^^^^^^^^^^^^^^^^^^^^

You can configure Promtail to optimize your scrape jobs. 
To do so, you need to follow these steps: 

1. Define a namespace for the `kubernetes_sd_config <https://prometheus.io/docs/prometheus/latest/configuration/configuration/#kubernetes_sd_config>`_.
2. Define a `label selectors <https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#label-selectors>`_ to scrape only required Pods. This is recommended to save resources.
3. [Optional] Enable all original labels from Pods via `relabel_configs <https://grafana.com/docs/loki/latest/send-data/promtail/configuration/#relabel_configs>`_ and action labelmap.
4. [Optional] Add the rest of Juju topology to each log via `pipeline stages and static_labels <https://grafana.com/docs/loki/latest/send-data/promtail/stages/static_labels/>`_.

Here's an example of scrape jobs for `istio-pilot <https://charmhub.io/istio-pilot>`_ and `istio-gateway <https://charmhub.io/istio-gateway>`_ controllers:

.. code-block:: bash

  - job_name: istio
    kubernetes_sd_configs:
      - role: pod
        namespaces:
          names:
            - kubeflow
        selectors:
          - role: pod
            label: "app in (istio-ingressgateway, istiod)"
    relabel_configs:
      - action: replace
        source_labels:
          - __meta_kubernetes_pod_container_name
        target_label: workload
      - action: labelmap
        regex: __meta_kubernetes_pod_label_(.+)
      - action: replace
        source_labels:
          - __meta_kubernetes_namespace
        target_label: namespace
      - action: replace
        source_labels:
          - __meta_kubernetes_pod_name
        target_label: pod
      - source_labels:
          - __meta_kubernetes_pod_node_name
        target_label: __host__
      - replacement: /var/log/pods/*$1/*.log
        separator: /
        source_labels:
          - __meta_kubernetes_pod_uid
          - __meta_kubernetes_pod_container_name
        target_label: __path__
    pipeline_stages:
      - labeldrop:
        - filename
      - match:
          selector: '{app="istio-ingressgateway"}'
          stages:
            - static_labels:
                juju_application: istio-ingressgateway
                juju_unit: istio-ingressgateway/0
                charm: istio-gateway
      - match:
          selector: '{app="istiod"}'
          stages:
            - static_labels:
                juju_application: istio-pilot
                juju_unit: istio-pilot/0
                charm: istio-pilot

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Full example of Promtail deployment
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This section provides an example that monitors all non sidecar pattern charms. 
You can check their logs through the Grafana UI using the query:

.. code-block:: bash

  {juju_model="kubeflow", charm=~"istio-pilot|istio-gateway|knative-serving|knative-eventing|metacontroller-operator|training-operator"}

Use the example code by specifying your :ref:`Promtail configuration <promtail_configuration>`.

This Promtail deployment ``.yaml`` file can be applied to the ``kubeflow`` model as follows:
 
.. code-block:: bash

  kubectl apply -f ./CKF_promtail.yaml -n kubeflow

Here's the code snippet:

.. code-block:: bash

    --- # Deployment.yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
    name: promtail
    labels:
        app: promtail
    spec:
    selector:
        matchLabels:
        app: promtail
    template:
        metadata:
        labels:
            app: promtail
        spec:
        serviceAccount: promtail-serviceaccount
        containers:
            - name: promtail
            image: grafana/promtail
            args:
                - -config.file=/etc/promtail/promtail.yaml
            env:
                - name: 'HOSTNAME' # needed when using kubernetes_sd_configs
                valueFrom:
                    fieldRef:
                    fieldPath: 'spec.nodeName'
            volumeMounts:
                - name: logs
                mountPath: /var/log/pods
                - name: promtail-config
                mountPath: /etc/promtail
        volumes:
            - name: logs
            hostPath:
                path: /var/log/pods
            - name: promtail-config
            configMap:
                name: promtail-config

    --- # configmap.yaml
    apiVersion: v1
    kind: ConfigMap
    metadata:
    name: promtail-config
    data:
    promtail.yaml: |
        server:
        http_listen_port: 9080
        grpc_listen_port: 0

        clients:
        - url: http://10.64.140.43/cos-loki-0/loki/api/v1/push
            external_labels:
            juju_model: kubeflow
            juju_model_uuid: f9e6966e-d7bb-4f19-8c4e-276c95880d39

        positions:
        filename: /tmp/positions.yaml

        target_config:
        sync_period: 10s

        scrape_configs:
        - job_name: istio
            kubernetes_sd_configs:
            - role: pod
                namespaces:
                names:
                    - kubeflow
                selectors:
                - role: pod
                    label: "app in (istio-ingressgateway, istiod)"
            relabel_configs:
            - action: replace
                source_labels:
                - __meta_kubernetes_pod_container_name
                target_label: workload
            - action: labelmap
                regex: __meta_kubernetes_pod_label_(.+)
            - action: replace
                source_labels:
                - __meta_kubernetes_namespace
                target_label: namespace
            - action: replace
                source_labels:
                - __meta_kubernetes_pod_name
                target_label: pod
            - source_labels:
                - __meta_kubernetes_pod_node_name
                target_label: __host__
            - replacement: /var/log/pods/*$1/*.log
                separator: /
                source_labels:
                - __meta_kubernetes_pod_uid
                - __meta_kubernetes_pod_container_name
                target_label: __path__
            pipeline_stages:
            - labeldrop:
                - filename
            - match:
                selector: '{app="istio-ingressgateway"}'
                stages:
                    - static_labels:
                        juju_application: istio-ingressgateway
                        juju_unit: istio-ingressgateway/0
                        charm: istio-gateway
            - match:
                selector: '{app="istiod"}'
                stages:
                    - static_labels:
                        juju_application: istio-pilot
                        juju_unit: istio-pilot/0
                        charm: istio-pilot

        - job_name: knative
            kubernetes_sd_configs:
            - role: pod
                namespaces:
                names:
                    - knative-eventing
                    - knative-serving
            relabel_configs:
            - action: replace
                source_labels:
                - __meta_kubernetes_pod_container_name
                target_label: workload
            - action: labelmap
                regex: __meta_kubernetes_pod_label_(.+)
            - action: replace
                source_labels:
                - __meta_kubernetes_namespace
                target_label: namespace
            - action: replace
                source_labels:
                - __meta_kubernetes_pod_name
                target_label: pod
            - source_labels:
                - __meta_kubernetes_pod_node_name
                target_label: __host__
            - replacement: /var/log/pods/*$1/*.log
                separator: /
                source_labels:
                - __meta_kubernetes_pod_uid
                - __meta_kubernetes_pod_container_name
                target_label: __path__
            pipeline_stages:
            - labeldrop:
                - filename
            - match:
                selector: '{namespace="knative-eventing"}'
                stages:
                    - static_labels:
                        juju_application: knative-eventing
                        juju_unit: knative-eventing/0
                        charm: knative-eventing
            - match:
                selector: '{namespace="knative-serving"}'
                stages:
                    - static_labels:
                        juju_application: knative-serving
                        juju_unit: knative-serving/0
                        charm: knative-serving

        - job_name: metacontroller
            kubernetes_sd_configs:
            - role: pod
                namespaces:
                names:
                    - kubeflow
                selectors:
                - role: pod
                    label: "app.kubernetes.io/name=metacontroller-operator"
            relabel_configs:
            - action: replace
                source_labels:
                - __meta_kubernetes_pod_container_name
                target_label: workload
            - action: labelmap
                regex: __meta_kubernetes_pod_label_(.+)
            - action: replace
                source_labels:
                - __meta_kubernetes_namespace
                target_label: namespace
            - action: replace
                source_labels:
                - __meta_kubernetes_pod_name
                target_label: pod
            - source_labels:
                - __meta_kubernetes_pod_node_name
                target_label: __host__
            - replacement: /var/log/pods/*$1/*.log
                separator: /
                source_labels:
                - __meta_kubernetes_pod_uid
                - __meta_kubernetes_pod_container_name
                target_label: __path__
            pipeline_stages:
            - labeldrop:
                - filename
            - static_labels:
                juju_application: metacontroller-operator
                juju_unit: metacontroller-operator/0
                charm: metacontroller-operator

        - job_name: training-operator
            kubernetes_sd_configs:
            - role: pod
                namespaces:
                names:
                    - kubeflow
                selectors:
                - role: pod
                    label: "control-plane=kubeflow-training-operator"
            relabel_configs:
            - action: replace
                source_labels:
                - __meta_kubernetes_pod_container_name
                target_label: workload
            - action: labelmap
                regex: __meta_kubernetes_pod_label_(.+)
            - action: replace
                source_labels:
                - __meta_kubernetes_namespace
                target_label: namespace
            - action: replace
                source_labels:
                - __meta_kubernetes_pod_name
                target_label: pod
            - source_labels:
                - __meta_kubernetes_pod_node_name
                target_label: __host__
            - replacement: /var/log/pods/*$1/*.log
                separator: /
                source_labels:
                - __meta_kubernetes_pod_uid
                - __meta_kubernetes_pod_container_name
                target_label: __path__
            pipeline_stages:
            - labeldrop:
                - filename
            - static_labels:
                juju_application: training-operator
                juju_unit: training-operator/0
                charm: training-operator

    --- # Clusterrole.yaml
    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRole
    metadata:
    name: promtail-clusterrole
    rules:
    - apiGroups: [""]
        resources:
        - nodes
        - services
        - pods
        verbs:
        - get
        - watch
        - list

    --- # ServiceAccount.yaml
    apiVersion: v1
    kind: ServiceAccount
    metadata:
    name: promtail-serviceaccount

    --- # Rolebinding.yaml
    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRoleBinding
    metadata:
    name: promtail-clusterrolebinding
    subjects:
    - kind: ServiceAccount
        name: promtail-serviceaccount
        namespace: kubeflow
    roleRef:
        kind: ClusterRole
        name: promtail-clusterrole
        apiGroup: rbac.authorization.k8s.io



