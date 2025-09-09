.. _deploy_autoscaling_model:

Deploy Autoscaling model serving
================================

The Autoscaling model serving solution offers the ability to deploy `KServe <https://kserve.github.io/website/0.13/>`_, `Knative <https://knative.dev/>`_, 
and `Istio <https://istio.io/>`_ charms on their own to serve Machine Learning (ML) models that can be accessed through ingress.

---------------------
Requirements
---------------------

* `Juju`_ 2.9.49 or above.
* A Kubernetes cluster with a configured ``LoadBalancer``, DNS, and a storage class solution.

---------------------
Deploy the solution
---------------------

You can deploy the solution in the following ways:

1. Deploy with Terraform.
2. Deploy with charm bundle.

Regardless of the chosen deployment method, the following charm configuration is required:

.. code-block:: bash

   juju config knative-serving istio.gateway.namespace="<Istio ingress gateway namespace>"

where the Istio ingress gateway namespace corresponds to the model name where the ``autoscaling-model-serving`` bundle is deployed.

~~~~~~~~~~~~~~~~~~~~~
Deploy with Terraform
~~~~~~~~~~~~~~~~~~~~~

The Autoscaling model serving is defined with a `Terraform module <https://github.com/canonical/autoscaling-model-serving/tree/track/0.1/terraform>`_ 
that facilitates its deployment using the `Terraform Juju provider <https://github.com/juju/terraform-provider-juju/>`_. 

In its most basic form, the solution can be deployed as follows:

.. code-block:: bash

   terraform apply -v

Refer to `this code <https://github.com/canonical/autoscaling-model-serving/tree/track/0.1/terraform#api>`_ for more information about inputs and outputs of the module.

~~~~~~~~~~~~~~~~~~~~~~~~~~
Deploy with charm bundle
~~~~~~~~~~~~~~~~~~~~~~~~~~

Charm bundles are now `obsolete <https://discourse.charmhub.io/t/discontinuing-new-charmhub-bundle-registrations/15344>`_, 
but as part of v0.1, the ``bundle.yaml`` file is still available. 
To deploy:

1. Clone the ``autoscaling-model-serving`` `repository <https://github.com/canonical/autoscaling-model-serving/>`_.
2. Deploy using the ``bundle.yaml`` file:

.. code-block:: bash

   juju deploy ./bundle/bundle.yaml --trust

---------------------
Perform inference
---------------------

1. Apply an ``InferenceService``.

.. note::

   Kserve offers a simple example in steps `1 <https://kserve.github.io/website/docs/getting-started/predictive-first-isvc#1-create-a-namespace>`_, `2 <https://kserve.github.io/website/docs/getting-started/predictive-first-isvc#2-create-an-inferenceservice>`_, and `3 <https://kserve.github.io/website/docs/getting-started/predictive-first-isvc#3-check-inferenceservice-status>`_.

2. Perform inference by making a request using the URL from the recently created ``InferenceService``.

For example, by running:

.. code-block:: bash

   kubectl get inferenceservices <name of the inferenceservice> -n <namespace where it is deployed>

You get the following output:

.. code-block:: bash

   NAME       	URL                                             	READY   PREV   LATEST   PREVROLLEDOUTREVISION   LATESTREADYREVISION                	AGE
   <name>   http://<name>.<namespace>.<LoadBalancer IP.DNS>     	True       	100  

The ``http://<name>.<namespace>.<LoadBalancer IP.DNS>`` can be used in any sort of request, for example:

.. code-block:: bash

   curl -v -H "Content-Type: application/json" http://<name>.<namespace>.<LoadBalancer IP.DNS>/v1/models/<name>:predict -d @./some-input.json

---------------------
Integrate with COS
---------------------

You can integrate the solution with Canonical Observability Stack (COS) while deploying with the Terraform module by running:

.. code-block:: bash

   terraform apply -var cos_configuration=true

If the solution was deployed using the charm bundle, or using the Terraform module without the COS options passed, see :ref:`Integrate with COS <integrate_cos>` for more details.
