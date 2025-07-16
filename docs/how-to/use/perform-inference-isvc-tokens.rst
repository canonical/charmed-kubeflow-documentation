.. _perform_inference_isvc_tokens:

Perform inference on ISVCs using access tokens
==============================================

This guide describes how to configure Charmed Kubeflow to perform inference on user owned KServe Inference Services (ISVCs) using programmatic access tokens, for example from a Jupyter Notebook.

To do so, follow these steps:

1. Create a `ServiceAccount token <https://kubernetes.io/docs/reference/access-authn-authz/service-accounts-admin/#bound-service-account-tokens>`_ 
from inside a Notebook server with the following parameters:

.. code-block:: bash

   TOKEN=$(kubectl create token \
       default-editor \
       --duration=<duration in s> \
       --audience=istio-ingressgateway.kubeflow.svc.cluster.local)

.. note::

   The ``--audience`` parameter has to be set to ``istio-ingressgateway.kubeflow.svc.cluster.local`` for requests with this token to go through, 
   otherwise they can be rejected.

2. Pass the ``TOKEN`` to the request, for example:

.. code-block:: bash

   curl $ISVC_URL/v1/models/model-endpoint \
       -H "Authorization: Bearer $TOKEN" \
       -H 'Content-Type: <content-type>' \
       -d <data>

.. note::

   The ``ServiceAccount`` token is bound to the ``default-editor ServiceAccount`` that gets created for every Charmed Kubeflow user. That being said:
   * The token will not be valid if the ``default-editor ServiceAccount`` is deleted.
   * The access cannot be revoked unless the ``duration`` set at creation time has expired.

