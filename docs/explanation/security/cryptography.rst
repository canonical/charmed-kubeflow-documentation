:tocdepth: 2

.. _cryptography:

Cryptography
============

This document describes the cryptography used by Charmed Kubeflow (CKF).

Resource checksums
------------------

CKF uses pinned revisions of `Rocks <https://documentation.ubuntu.com/rockcraft/en/latest/explanation/rocks>`_. 
`Charms <https://documentation.ubuntu.com/juju/3.6/reference/charm/#charm>`_ are refreshed to a newer revision periodically, but each revision is pointing to a specific `Rock hash <https://github.com/canonical/notebook-operators/blob/track/1.10/charms/jupyter-controller/metadata.yaml#L16>`_. 
This provides reproducible and secure environments.

Every artifact bundled into CKF is verified against their MD5, SHA256 or SHA512 checksum after download.

Sources verification
--------------------

CKF sources are stored in GitHub repositories.
See `Rocks repositories <https://github.com/search?q=org%3Acanonical+topic%3Arocks+topic%3Akubeflow&type=repositories>`_ and `Charms ones <https://github.com/search?q=org%3Acanonical+topic%3Acharm+topic%3Akubeflow&type=repositories>`_.

All CKF artifacts built by Canonical are programmatically published and released using release pipelines implemented with GitHub Actions.
Distributions, provided as `bundles <https://github.com/canonical/bundle-kubeflow>`_, are published and versioned in tracks via `Charmhub <https://charmhub.io/kubeflow>`_, while Rocks and Charms are published using the workflows defined in their respective repositories.

All repositories in GitHub are set up with branch protection rules, requiring:

* New commits to be merged to main branches via pull request with at least one approval from repository maintainers.
* New commits to be signed, for instance, using GPG keys.
* Developers to sign the `Canonical Contributor License Agreement (CLA) <https://ubuntu.com/legal/contributors>`_.

Cryptographic processes
-----------------------

CKF is a `Juju bundle <https://juju.is/docs/juju/bundle>`_ consisting of several charms integrated using Juju `relations <https://juju.is/docs/juju/relation>`_. 
See `Kubeflow bundle <https://charmhub.io/kubeflow>`_ for more details.

Below is an overview of all cryptographic processes handled by the charms integrated in the CKF bundle.

.. note::
    The charms within the CKF bundle that are not mentioned in this guide do not involve any cryptography in their logic.

``admission-webhook``
~~~~~~~~~~~~~~~~~~~~~

The `admission-webhook charm <https://charmhub.io/admission-webhook>`_ is responsible for applying :ref:`Kubeflow PodDefaults <leverage_poddefaults>` to newly created `Pods <https://kubernetes.io/docs/concepts/workloads/pods/>`_, and thus creates a `MutatingWebhookConfiguration <https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/#webhook-configuration>`_ object for registering a webhook.

The charm code generates a self-signed `X.509 <https://en.wikipedia.org/wiki/X.509>`_ certificate, 
so that the Kubernetes API server can confirm the workload container identity.

``dex-auth``
~~~~~~~~~~~~

The `dex-auth charm <https://charmhub.io/dex-auth>`_ is an operator for `Dex <https://dexidp.io/>`_, which provides :ref:`authentication <authentication>` to Charmed Kubeflow.

If the `Dex built-in connector <https://dexidp.io/docs/connectors/local/>`_ is used, then the source of truth resides in the connected system, such as Azure Entra ID, AWS Cognito. 
Dex also supports credentials login via `configuration options <https://charmhub.io/dex-auth/configurations#static-password>`_ in the charms. 
In this case, the password is always stored as a hash, generated using ``bcrypt.hashpwd`` with a Blowfish cipher.

When using the Dex built-in connector, the user password specified in the ``static_password`` field is hashed with the ``bcrypt`` library before being stored in the Dex configuration.

Additionally, the stored charm state is `salted <https://en.wikipedia.org/wiki/Salt_(cryptography)>`_ with ``bcrypt`` to prevent reverse engineering.

``istio-pilot``
~~~~~~~~~~~~~~~

CKF uses `Istio service mesh <https://istio.io/latest/docs/>`_ to enable end-to-end authentication and access control. 
The workload container of `istio-pilot <https://charmhub.io/istio-pilot>`_ is responsible for distributing an X.509 certificate using ``sha25withRSAEncryption`` to every sidecar container in the following path: ``/var/run/secrets/istio/root-cerm.pem``.

When a workload container, i.e., the client, sends a request to another workload container, i.e., the server:

* Istio reroutes the outbound traffic to the client's sidecar.
* The client's sidecar starts an `mTLS <https://www.cloudflare.com/learning/access-management/what-is-mutual-tls/>`_ handshake with the server's sidecar.
* The two sidecars establish an mTLS connection, and Istio forwards the traffic from the client to the server.
* The server sidecar authorises the request, and forwards the traffic to the backend service through local TCP connections.

.. note::
    The minimum required version of TLS is ``TLSv1_2``.

See `Istio Mutual TLS authentication <https://istio.io/latest/docs/concepts/security/#mutual-tls-authentication>`_ for more details.

Additionally, the charm uses the `cert-handler <https://charmhub.io/observability-libs/libraries/cert_handler>`_ library to generate an X.509 certificate for the Istio ``Gateway`` object.

``katib-controller``
~~~~~~~~~~~~~~~~~~~~

The `katib-controller charm <https://charmhub.io/katib-controller>`_ creates a ``MutatingWebhookConfiguration`` object that calls a webhook whenever a new `Kubeflow experiment <https://www.kubeflow.org/docs/components/pipelines/concepts/experiment/>`_ is created or updated.

Similar to other charms that create ``MutatingWebhookConfiguration`` objects, ``katib-controller`` generates a self-signed X.509 certificate so that the Kubernetes API server can confirm the workload container identity.

``kfp-persistence``
~~~~~~~~~~~~~~~~~~~

The `kfp-persistence charm <https://charmhub.io/kfp-persistence>`_ creates a `service account token <https://kubernetes.io/docs/reference/access-authn-authz/authentication/#service-account-tokens>`_ that is used to verify requests to the Kubeflow Pipelines service.

``kserve-controller``
~~~~~~~~~~~~~~~~~~~~~

The `kserve-controller charm <https://discourse.charmhub.io/t/kserve-controller-docs-index/11939>`_ creates a ``MutatingWebhookConfiguration`` object that calls a webhook whenever a new `KServe InferenceService <https://kserve.github.io/website/latest/get_started/first_isvc/>`_ is created or updated.

Similar to other charms that create ``MutatingWebhookConfiguration`` objects, this charm generates a self-signed X.509 certificate so that the Kubernetes API server can confirm the workload container identity.

``minio``
~~~~~~~~~~

The `minio charm <https://charmhub.io/minio>`_ is an operator for `MinIO <https://min.io/>`_, which provides S3 object storage. It uses a field in the ``object-storage`` interface named ``secret-key``. 
Its value is created from a randomly generated 30-character string.

Additionally, the charm adds a randomly generated salt to its configuration before it is hashed with SHA-256, to prevent reverse engineering the ``secret-key`` field.

``oidc-gatekeeper``
~~~~~~~~~~~~~~~~~~~

The `oidc-gatekeeper charm <https://charmhub.io/oidc-gatekeeper>`_ uses the ``client-name`` and ``client-secret`` configuration options for the `OpenID Connect <https://openid.net/developers/how-connect-works/>`_ client. 
Similarly to ``minio``, the value of ``secret-key`` is created from a randomly generated 30-character string.

``pvc-viewer``
~~~~~~~~~~~~~~

The `pvc-viewer charm <https://github.com/canonical/pvcviewer-operator>`_ creates a ``MutatingWebhookConfiguration`` object that calls a webhook whenever a new `PVCViewer <https://github.com/kubeflow/kubeflow/blob/v1.9-branch/components/proposals/20230130-pvcviewer-controller.md>`_ is created or updated.

Similar to other charms that create ``MutatingWebhookConfiguration`` objects, this charm generates a self-signed X.509 certificate so that the Kubernetes API server can confirm the workload container identity.
