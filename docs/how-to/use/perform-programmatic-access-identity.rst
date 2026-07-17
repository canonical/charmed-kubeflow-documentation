.. _perform_programmatic_access_identity:

Perform programmatic access with Canonical Identity Platform
============================================================

This guide describes how to perform machine-to-machine (M2M) programmatic access, from outside the cluster, to Charmed Kubeflow (CKF) components when CKF is integrated with the
`Canonical Identity Platform <https://charmhub.io/topics/canonical-identity-platform>`_.

In this setup, API access goes through a dedicated machine-to-machine ingress gateway, and requests are authenticated with a `JSON Web Token (JWT) <https://jwt.io/introduction>`_
issued by `Hydra <https://charmhub.io/hydra>`_ using the OAuth 2.0 ``client_credentials`` grant.

Two conditions must be met for a request to succeed:

* **Authentication**: the request carries a valid JWT whose issuer is trusted by the machine-to-machine gateway.
* **Authorization**: the token's identity (the OAuth client) is granted access to the target Kubeflow profile.

The steps to obtain a token and authorize a client are the same for any Kubeflow component that exposes an API through the machine-to-machine gateway.
This guide uses a `KServe <https://kserve.github.io/website/>`_ ``InferenceService`` (ISVC) as a worked example.

To do so, follow these steps:

1. `Create an OAuth client <#create-an-oauth-client>`_.
2. `Export the client credentials <#export-the-client-credentials>`_.
3. `Authorize the client on a profile <#authorize-the-client-on-a-profile>`_.
4. `Request an access token <#request-an-access-token>`_.
5. `Access the component API <#access-the-component-api>`_.

-----------------------------------
Requirements
-----------------------------------

* A CKF deployment integrated with the Canonical Identity Platform. See :ref:`Install with Canonical Identity Platform using Terraform <install_terraform_identity>`.
* A Kubeflow profile owned by an Identity Platform user. See :ref:`Manage profiles <manage_profiles>`.
* ``juju``, ``kubectl``, ``curl`` and `jq <https://jqlang.github.io/jq/>`_ available on your machine.

.. note::

   The following steps assume the two ingress gateways use the hostnames from the deployment guide: ``ui.kubeflow.com`` for the UI gateway and ``api.kubeflow.com`` for the machine-to-machine gateway.
   Adjust the hostnames to match your deployment.

-----------------------------------
Create an OAuth client
-----------------------------------

Create an OAuth client in Hydra using the ``client_credentials`` grant type:

.. code-block:: bash

   juju run -m iam hydra/0 create-oauth-client \
       name="my-m2m-client" \
       grant-types='["client_credentials"]' \
       response-types='["token"]' \
       scope='["openid"]'

.. note::

   You can use any ``name`` for the client. You can either reuse a single client for all programmatic access, or create a separate client per consumer.
   Because a client is authorized individually as a profile contributor (see `Authorize the client on a profile <#authorize-the-client-on-a-profile>`_), separate
   clients let you grant or revoke access per consumer, whereas a shared client grants the same access to everyone using it.

.. note::

   The ``audience`` field is optional. Set it if you want to restrict the scope of the token. If you set it here, you must also request the token with a matching ``audience`` in the
   `Request an access token <#request-an-access-token>`_ step. See the `create-oauth-client action <https://charmhub.io/hydra/actions#create-oauth-client>`_ for details.

-----------------------------------
Export the client credentials
-----------------------------------

From the action output, export the client credentials as environment variables:

.. code-block:: bash

   CLIENT_ID=<client_id from the action output>
   CLIENT_SECRET=<client_secret from the action output>

-----------------------------------
Authorize the client on a profile
-----------------------------------

Authentication alone is not enough. The token's identity (the ``CLIENT_ID``) must also be authorized to access the target profile, otherwise the request is authenticated but rejected with ``RBAC: access denied``.

The ``github-profiles-automator`` charm reconciles profiles from a ``pmr.yaml`` file in the configured repository. Add the client as a contributor on the profile that owns the target resource.

Edit ``pmr.yaml`` in the repository synced by ``github-profiles-automator`` so the profile lists the ``CLIENT_ID`` as a contributor:

.. code-block:: yaml

   profiles:
   - name: ml-engineering
     owner:
       kind: User
       name: user1
     contributors:
     - name: "<CLIENT_ID>"   # the OAuth client's CLIENT_ID
       role: edit

Commit and push the change. Once ``github-profiles-automator`` syncs it, an ``AuthorizationPolicy`` is created granting the client ``edit`` access to the profile.

.. note::

   Contributors can have the ``admin``, ``edit`` or ``view`` role. The gateway ``AuthorizationPolicy`` that lets a client reach the component is created for any of these roles,
   so a ``view`` (read-only) contributor can still send inference requests. The role only changes the client's Kubernetes RBAC (``kubeflow-view`` versus ``kubeflow-edit``), that is,
   what it can create or modify through the Kubernetes API. Use ``view`` for consumers that should only run inferences without modifying the served models.

.. note::

   The profile ``owner.name`` maps to the Identity Platform (Kratos) username, while a machine-to-machine ``contributor`` is identified by the OAuth client's ``CLIENT_ID``.
   See :ref:`Manage profiles <manage_profiles>` for the full ``pmr.yaml`` format.

-----------------------------------
Request an access token
-----------------------------------

First discover the token issuer and endpoint, then request the token:

.. code-block:: bash

   # the JWT issuer URL trusted by the machine-to-machine gateway (sourced from oauth2-proxy)
   ISSUER_URL=$(juju run -m kubeflow oauth2-proxy/0 get-extra-jwt-issuers --format json \
       | jq -r '.["oauth2-proxy/0"].results["extra-jwt-issuers"]' \
       | tr "'" '"' | jq -r '.[0]["oidc-issuer-url"]')

   # resolve the token endpoint from the issuer's OpenID configuration
   TOKEN_URL=$(curl -sk "$ISSUER_URL/.well-known/openid-configuration" | jq -r '.token_endpoint')

   # request the token
   TOKEN=$(curl -sk -X POST "$TOKEN_URL" \
       -u "$CLIENT_ID:$CLIENT_SECRET" \
       -d "grant_type=client_credentials" \
       -d "scope=openid" | jq -r '.access_token')

.. note::

   The token lifetime cannot currently be configured through the charm and defaults to 1 hour.

.. note::

   The issuer hostname (for example, ``auth.kubeflow.com``) must be resolvable from the machine requesting the token, and in-cluster (through CoreDNS) so that Istio can fetch the issuer's
   JWKS to validate the token at the gateway. This DNS configuration is a deployment-time step covered in :ref:`Install with Canonical Identity Platform using Terraform <install_terraform_identity>`.

-----------------------------------
Access the component API
-----------------------------------

Once you have a token, pass it as a bearer token in the ``Authorization`` header of your requests to the target component's API through the machine-to-machine gateway.
The following example uses a KServe ``InferenceService``.

.. note::

   Every component is reached through the machine-to-machine gateway's (sub)domain, which is the single point of ingress. KServe is a special case: it automatically provisions a
   per-service subdomain of the gateway domain (for example, ``sklearn-v2-iris-ml-engineering.api.kubeflow.com``). Other components are reached at the gateway hostname
   (``api.kubeflow.com``) under their own route path (for example, MLflow at ``https://api.kubeflow.com/mlflow/``).

First, discover the machine-to-machine gateway serving the KServe domain. The gateway name matches the ``istio-ingress-k8s`` app serving the ``api.kubeflow.com`` domain:

.. code-block:: bash

   # discover the Gateway whose listeners serve the machine-to-machine domain
   GATEWAY=$(kubectl -n kubeflow get gateway -o \
       jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.spec.listeners[*].hostname}{"\n"}{end}' \
       | grep "api.kubeflow.com" | head -1 | cut -f1)

.. TODO: Remove the following workaround once canonical/service-mesh#102 is fixed and the istio-ingress-k8s charm supports wildcard listeners natively.

.. warning::

   **Workaround for** `canonical/service-mesh#102 <https://github.com/canonical/service-mesh/issues/102>`_ **(KServe only).**
   The ``istio-ingress-k8s`` charm pins each gateway listener to the exact ``external_hostname`` (``api.kubeflow.com``). KServe publishes per-service subdomain routes
   (for example, ``sklearn-v2-iris-ml-engineering.api.kubeflow.com``), which do not attach to an exact-hostname listener and are rejected with ``NoMatchingListenerHostname``.
   Until the charm supports wildcard listeners, patch the live ``Gateway`` so both listeners use ``*.api.kubeflow.com``:

   .. code-block:: bash

      kubectl -n kubeflow patch gateway "${GATEWAY}" --type=json -p='[
        {"op": "replace", "path": "/spec/listeners/0/hostname", "value": "*.api.kubeflow.com"},
        {"op": "replace", "path": "/spec/listeners/1/hostname", "value": "*.api.kubeflow.com"}
      ]'

   The charm reconciles the ``Gateway``, so this patch may be reverted on the next ``update-status`` or ``config-changed`` hook. Re-apply it if KServe routes stop attaching.

1. Create an ``InferenceService`` in the profile namespace (``ml-engineering`` in this example):

.. code-block:: bash

   cat <<EOF | kubectl apply -n ml-engineering -f -
   apiVersion: "serving.kserve.io/v1beta1"
   kind: "InferenceService"
   metadata:
     name: "sklearn-v2-iris"
   spec:
     predictor:
       model:
         modelFormat:
           name: sklearn
         protocolVersion: v2
         runtime: kserve-sklearnserver
         storageUri: "gs://kfserving-examples/models/sklearn/1.0/model"
         resources:
           limits:
             cpu: 1
             memory: 500Mi
           requests:
             cpu: 100m
             memory: 250Mi
   EOF

2. Wait for the ``InferenceService`` to become ready:

.. code-block:: bash

   kubectl get isvc -n ml-engineering

The ``READY`` column should show ``True``:

.. code-block:: none

   NAME              URL                                                       READY
   sklearn-v2-iris   http://sklearn-v2-iris-ml-engineering.api.kubeflow.com    True

3. Resolve the ``InferenceService`` hostname:

.. code-block:: bash

   SERVICE_HOSTNAME=$(kubectl get inferenceservice sklearn-v2-iris -n ml-engineering \
       -o jsonpath='{.status.url}' | cut -d "/" -f 3)

4. Make the inference request, passing the token in the ``Authorization`` header:

.. code-block:: bash

   curl -sk \
     -H "Authorization: Bearer ${TOKEN}" \
     -H "Content-Type: application/json" \
     "https://${SERVICE_HOSTNAME}/v1/models/sklearn-v2-iris:predict" \
     -d '{"instances": [[6.8, 2.8, 4.8, 1.4], [6.0, 3.4, 4.5, 1.6]]}'

A successful request returns a prediction:

.. code-block:: none

   {"predictions":[1,1]}

This confirms that Hydra issued a ``client_credentials`` token, the machine-to-machine gateway validated the JWT issuer, the profile's ``AuthorizationPolicy`` authorized the client identity,
and the component served the request once authenticated and authorized.
