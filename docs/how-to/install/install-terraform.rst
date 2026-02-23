.. _install_terraform:

Install using Terraform
=======================

This guide describes how to install Charmed Kubeflow (CKF) using `Terraform <https://www.terraform.io/>`_.  

You can do so by using any `CNCF certified <https://www.cncf.io/training/certification/software-conformance/#logos>`_ Kubernetes (K8s) cluster 
and deploying Kubeflow using the Terraform, `Kubernetes <https://kubernetes.io/>`_ and `Juju`_ Command Line Interfaces (CLIs).

---------------------
Requirements
---------------------

* A K8s cluster version 1.29-1.32 with a default `storage class <https://kubernetes.io/docs/concepts/storage/storage-classes/>`_ configured.  
* `Terraform CLI <https://developer.hashicorp.com/terraform/cli>`_. You can install it using the `snap`_.

---------------------
Bootstrap Juju
---------------------

CKF is deployed to Kubernetes with Juju. 
Before deployment, Juju must be bootstrapped to the K8s cluster. 
See `Get started with Juju <https://documentation.ubuntu.com/juju/latest/tutorial/>` for more details.

.. note::

   Check :ref:`Supported versions <supported_kubeflow_versions>` for version compatibility between CKF, Juju and K8s.

---------------------
Deploy CKF
---------------------

Deploy CKF as follows:

1. Clone the repository and change directory to the Kubeflow module:

.. note::

   This command checks out the default branch, which should be ``track/1.9``. If that's not the case, make sure to ``git checkout`` to that branch.

.. code-block:: bash

   git clone https://github.com/canonical/charmed-kubeflow-solutions
   cd charmed-kubeflow-solutions/modules/kubeflow

2. Initialise Terraform. The following command downloads all the required `Terraform modules <https://developer.hashicorp.com/terraform/language/modules>`_ and installs the Terraform `Juju provider <https://registry.terraform.io/providers/juju/juju/latest/docs>`_:

.. code-block:: bash

   terraform init

3. Define credentials:

.. code-block:: bash

   DEX_USERNAME="your-username"
   DEX_PASSWORD="your-password"

4. Deploy CKF using Terraform as follows:

.. code-block:: bash

   terraform apply \
      -var cos_configuration=true \
      -var dex_static_username=$DEX_USERNAME \
      -var dex_static_password=$DEX_PASSWORD

The command above:

* Creates a `Juju model <https://juju.is/docs/juju/model>`_ named ``kubeflow``.  
* Deploys CKF ``1.11/stable``.  
* Configures CKF to integrate with `Canonical Observability Stack <https://charmhub.io/topics/canonical-observability-stack>`_. See :ref:`Monitoring <index_monitoring>` for more details.  
* Configures `dex-auth <https://charmhub.io/dex-auth>`_ charm with a static user username and password.  

See `CKF Terraform solution <https://github.com/canonical/charmed-kubeflow-solutions/blob/track/1.11/modules/kubeflow/README.md>`_ for more details.

5. Once the deployment is completed, you should see the Terraform solution module's outputs:

.. code-block:: bash

   Outputs:

   grafana_agent_k8s  = {
       app_name = "grafana-agent-k8s-kubeflow"
       provides = {
           grafana_dashboards_provider = "grafana-dashboards-provider"
       }
       requires = {
           logging_consumer  = "logging-consumer"
           send_remote_write = "send-remote-write"
       }
   }
   kserve_controller = {
       app_name = "kserve-controller"
       provides = {
           metrics_endpoint = "metrics-endpoint"
       }
       requires = {
           ingress_gateway  = "ingress-gateway"
           local_gateway    = "local-gateway"
           logging          = "logging"
           object_storage   = "object-storage"
           secrets          = "secrets"
           service_accounts = "service-accounts"
       }
   }
   model = "kubeflow"
   tls_certificate_requirer = {
       app_name = "istio-pilot"
       requires = "certificates"
   }

See `Outputs <https://github.com/canonical/charmed-kubeflow-solutions/blob/track/1.9/modules/kubeflow/README.md#outputs>`_ for more details.

6. Verify all charms are in ``active`` status by monitoring the Juju model:

.. code-block:: bash

   juju status --watch 1s

.. note::

   This may take up to some minutes, depending on the cluster's node specifications.

---------------------
Access CKF dashboard
---------------------

You can access the CKF dashboard through an IP address. 
See :ref:`Access CKF dashboard <access_ckf_dashboard>` for more details.
