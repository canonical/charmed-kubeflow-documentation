.. _migrate_ambient:

Migrate to Istio Ambient Mesh
==============================

This guide describes how to migrate Charmed Kubeflow (CKF) from Istio sidecar mode to Istio Ambient Mesh.

`Istio Ambient Mesh`_ is a data plane mode that provides secure communication between services without injecting sidecar proxies into application pods. 

.. warning::
   This migration process involves removing and redeploying several critical components. 
   Ensure you have a backup of your deployment before proceeding. See :ref:`back_up` for more details.

.. warning::
   During the migration, your Kubeflow deployment will experience downtime as components are removed and redeployed. 
   Plan the migration during a maintenance window.

---------------------
Requirements
---------------------

* A running CKF 1.11 deployment with sidecar-based Istio.
* Admin access to the Kubernetes (K8s) cluster where CKF is deployed.
* Juju admin access to the ``kubeflow`` model.
* ``kubectl`` CLI tool installed and configured.

---------------------
Migration process
---------------------

Configure Cilium (Canonical Kubernetes only)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If you are running on Canonical Kubernetes, configure Cilium to work with Charmed Istio in ambient mode. 
For more information, see the `Cilium documentation`_.

.. code-block:: bash

   kubectl -n kube-system patch configmap cilium-config --type merge --patch '{"data":{"bpf-lb-sock-hostns-only":"true"}}'
   kubectl -n kube-system rollout restart daemonset cilium

Remove legacy Istio components
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Remove the existing sidecar-based Istio installation:

.. code-block:: bash

   juju remove-application --no-prompt istio-ingressgateway
   juju remove-application --no-prompt istio-pilot

Remove Knative components
^^^^^^^^^^^^^^^^^^^^^^^^^^

Remove Knative components, as serverless deployment is not supported with Ambient Mesh:

.. code-block:: bash

   juju remove-application --no-prompt knative-serving
   juju remove-application --no-prompt knative-eventing
   juju remove-application --no-prompt knative-operator

.. warning::
   Ambient Mesh does not support serverless deployments. 
   This is why Knative must be removed and KServe will be configured to use raw deployment mode instead.

Deploy new Istio charms
^^^^^^^^^^^^^^^^^^^^^^^^

Deploy the new Istio charms that support Ambient Mesh.

Deploy ``istio-k8s`` with the appropriate ``platform`` value for your Kubernetes substrate:

* For **MicroK8s** (default): Use ``platform=microk8s`` or omit the config (defaults to ``microk8s``)

  .. code-block:: bash

     juju deploy istio-k8s --trust --channel 2/edge --config platform=microk8s

* For **Canonical Kubernetes**: Use ``platform=""`` (empty string)

  .. code-block:: bash

     juju deploy istio-k8s --trust --channel 2/edge --config platform=""

* For other platforms: Refer to the `Istio platform prerequisites`_ for the appropriate value

  .. code-block:: bash

     juju deploy istio-k8s --trust --channel 2/edge --config platform=<appropriate-value>

Deploy the remaining Istio components and integrate:

.. code-block:: bash

   juju deploy istio-beacon-k8s --trust --channel 2/edge
   juju deploy istio-ingress-k8s --trust --channel 2/edge
   juju integrate istio-ingress-k8s istio-k8s

Migrate admission webhook
^^^^^^^^^^^^^^^^^^^^^^^^^^

Upgrade the admission webhook to the latest version and integrate it with the Ambient Mesh:

.. code-block:: bash

   juju refresh admission-webhook
   juju integrate admission-webhook:service-mesh istio-beacon-k8s:service-mesh

Migrate Katib components
^^^^^^^^^^^^^^^^^^^^^^^^^

Upgrade the Katib controller and UI, and integrate them with the service mesh:

.. code-block:: bash

   juju refresh katib-controller
   juju integrate katib-controller:service-mesh istio-beacon-k8s:service-mesh
   juju refresh katib-ui
   juju integrate katib-ui:service-mesh istio-beacon-k8s:service-mesh
   juju integrate katib-ui:istio-ingress-route istio-ingress-k8s:istio-ingress-route

Migrate training operator
^^^^^^^^^^^^^^^^^^^^^^^^^^

Upgrade the training operator, and integrate with the service mesh:

.. code-block:: bash

   juju refresh training-operator
   juju integrate training-operator:service-mesh istio-beacon-k8s:service-mesh

Migrate KServe controller
^^^^^^^^^^^^^^^^^^^^^^^^^^

Upgrade KServe, configure it for raw deployment mode, and integrate with both the service mesh and the ingress gateway:

.. code-block:: bash

   juju refresh kserve-controller
   juju config kserve-controller deployment-mode=rawdeployment
   juju integrate kserve-controller:service-mesh istio-beacon-k8s:service-mesh
   juju integrate kserve-controller:gateway-metadata istio-ingress-k8s:gateway-metadata

.. note::
   With Ambient Mesh, KServe only supports ``rawdeployment`` mode. 
   Serverless deployment mode is not supported, which is why Knative was removed earlier in the migration process.

Migrate authentication components
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Upgrade Dex and OIDC Gatekeeper to work with the new Istio ingress:

.. code-block:: bash

   juju refresh dex-auth
   juju refresh oidc-gatekeeper
   juju integrate dex-auth:service-mesh istio-beacon-k8s:service-mesh
   juju integrate dex-auth:istio-ingress-route-unauthenticated istio-ingress-k8s:istio-ingress-route-unauthenticated
   juju integrate oidc-gatekeeper:service-mesh istio-beacon-k8s:service-mesh
   juju integrate oidc-gatekeeper:forward-auth istio-ingress-k8s:forward-auth
   juju integrate oidc-gatekeeper:istio-ingress-route-unauthenticated istio-ingress-k8s:istio-ingress-route-unauthenticated

Migrate Envoy
^^^^^^^^^^^^^

Upgrade the Envoy proxy and integrate with the service mesh:

.. code-block:: bash

   juju refresh envoy
   juju integrate envoy:service-mesh istio-beacon-k8s
   juju integrate envoy:istio-ingress-route istio-ingress-k8s:istio-ingress-route

Migrate Jupyter components
^^^^^^^^^^^^^^^^^^^^^^^^^^

Upgrade the Jupyter UI and controller, integrating both with the service mesh:

.. code-block:: bash

   juju refresh jupyter-controller
   juju integrate jupyter-controller:service-mesh istio-beacon-k8s:service-mesh
   juju integrate jupyter-controller:gateway-metadata istio-ingress-k8s:gateway-metadata
   juju refresh jupyter-ui
   juju integrate jupyter-ui:service-mesh istio-beacon-k8s:service-mesh
   juju integrate jupyter-ui:istio-ingress-route istio-ingress-k8s:istio-ingress-route

Migrate Kubeflow Pipelines components
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Upgrade all KFP components, add new relations, and integrate them with the service mesh:

.. code-block:: bash

   juju refresh kfp-api
   juju integrate kfp-api:service-mesh istio-beacon-k8s:service-mesh
   juju refresh kfp-persistence
   juju integrate kfp-persistence:service-mesh istio-beacon-k8s:service-mesh
   juju refresh kfp-profile-controller
   juju integrate kfp-profile-controller:service-mesh istio-beacon-k8s:service-mesh
   juju refresh kfp-schedwf
   juju integrate kfp-schedwf:service-mesh istio-beacon-k8s:service-mesh
   juju refresh kfp-ui
   juju integrate kfp-ui:service-mesh istio-beacon-k8s:service-mesh
   juju integrate kfp-ui:istio-ingress-route istio-ingress-k8s:istio-ingress-route
   juju refresh kfp-viz
   juju integrate kfp-viz:service-mesh istio-beacon-k8s:service-mesh
   juju integrate kfp-api:kfp-api-grpc kfp-persistence:kfp-api-grpc
   juju integrate kfp-api:kfp-api-grpc kfp-schedwf:kfp-api-grpc

Migrate MinIO
^^^^^^^^^^^^^

Upgrade MinIO and integrate it with the service mesh:

.. code-block:: bash

   juju refresh minio
   juju integrate minio:service-mesh istio-beacon-k8s:service-mesh

Migrate Kubeflow Dashboard
^^^^^^^^^^^^^^^^^^^^^^^^^^^

Upgrade the central dashboard and integrate with the ingress:

.. code-block:: bash

   juju refresh kubeflow-dashboard
   juju integrate kubeflow-dashboard:service-mesh istio-beacon-k8s:service-mesh
   juju integrate kubeflow-dashboard:istio-ingress-route istio-ingress-k8s:istio-ingress-route

Migrate Kubeflow Profiles
^^^^^^^^^^^^^^^^^^^^^^^^^^

Upgrade the profiles controller with Ambient Mesh configuration:

.. code-block:: bash

   juju refresh kubeflow-profiles
   juju integrate kubeflow-profiles:service-mesh istio-beacon-k8s:service-mesh
   juju config kubeflow-profiles service-mesh-mode=istio-ambient
   juju config kubeflow-profiles istio-gateway-service-account=istio-ingress-k8s-istio

.. note::
   The ``service-mesh-mode`` configuration is critical for Ambient Mesh support. 
   The ``istio-gateway-service-account`` must match the service account used by the Istio ingress gateway.

Migrate Kubeflow Volumes
^^^^^^^^^^^^^^^^^^^^^^^^^

Upgrade the volumes web app and integrate with the service mesh:

.. code-block:: bash

   juju refresh kubeflow-volumes
   juju integrate kubeflow-volumes:service-mesh istio-beacon-k8s:service-mesh
   juju integrate kubeflow-volumes:istio-ingress-route istio-ingress-k8s:istio-ingress-route

Migrate Tensorboard components
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Upgrade both the Tensorboard controller and web app:

.. code-block:: bash

   juju refresh tensorboard-controller
   juju integrate tensorboard-controller:service-mesh istio-beacon-k8s:service-mesh
   juju integrate tensorboard-controller:gateway-metadata istio-ingress-k8s:gateway-metadata

.. code-block:: bash

   juju refresh tensorboards-web-app
   juju integrate tensorboards-web-app:service-mesh istio-beacon-k8s:service-mesh
   juju integrate tensorboards-web-app:istio-ingress-route istio-ingress-k8s:istio-ingress-route

Remove sidecar containers from user namespaces
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

After migrating to Ambient Mesh, delete the ML Pipeline pods from existing user namespaces to remove the old sidecar containers. 
The pods will be automatically recreated by their deployments without sidecars:

.. code-block:: bash

   kubectl delete pod -l app=ml-pipeline-ui-artifact -n <user-namespace>
   kubectl delete pod -l app=ml-pipeline-visualizationserver -n <user-namespace>

Replace ``<user-namespace>`` with the actual user namespace name (for example, ``admin``).

---------------------
Verify the migration
---------------------

After completing the migration, verify that all components are operational:

1. Check that all applications are in the ``active`` state:

.. code-block:: bash

   juju status

2. Verify that the Kubeflow Dashboard is accessible, meaning that the ingress gateway allows for traffic.

3. Test core functionality, for example:

   * User authentication through Dex
   * Creating and accessing Jupyter notebooks
   * Running Kubeflow Pipelines
   * Deploying models with KServe

