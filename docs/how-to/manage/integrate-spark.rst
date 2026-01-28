.. _integrate_spark:

Integrate with Charmed Apache Spark
============================================

.. note::

   This feature is currently experimental. This guide exists here for experimental purposes.

This guide describes how Charmed Kubeflow (CKF) and the Charmed Apache Spark can be integrated using `Juju`_. 
This integration enables running Spark jobs in Kubeflow notebooks and pipelines.

.. _kf_spark_requirements:

---------------------
Requirements
---------------------

- Minimum system requirements are at least 8 cores CPU processor, 64GB of RAM and 150GB of disk space.
- `MicroK8s`_ and Juju. See :ref:`CKF supported versions <supported_kubeflow_versions>` 
  for more details about compatible versions of `Kubeflow <https://www.kubeflow.org/docs/releases/>`_ and Juju.
- CLI tools like ``charmcraft`` and ``tox``.
- Juju agent version ``<=3.6.9``

.. _integrate_with_existing_kubeflow_deployment:

-------------------------------------------------
Integrate Spark with an existing CKF deployment
-------------------------------------------------

This section of the guide assumes that you already have a CKF deployment in a Juju model named ``kubeflow`` in your 
Juju cloud. If not, please see :ref:`CKF getting started guide <get_started>` for more details on how to do so, or 
refer to :ref:`Deploy Kubeflow-Spark solution using Terraform <deploy_kubeflow_spark_solution_using_terraform>` 
section below.

.. note::

   When using an existing CKF deployment, sure that the ``metacontroller-operator`` charm is up to date with the 
   ``latest/edge`` channel, since the changes that support Charmed Apache Spark integration are not yet merged 
   to the stable channel.

Integrating CKF with Charmed Apache Spark involves the following:

1. Deploying the ``spark-integration-hub-k8s`` charm
2. Deploying ``data-kubeflow-integrator`` charm and integrating it with the ``spark-integration-hub-k8s`` charm
3. Deploying ``resource-dispatcher`` charm and integrating it with the ``data-kubeflow-integrator`` charm

.. _deploy_and_configure_spark_integration_hub:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Deploy Spark Integration Hub
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. note::

   If you have an existing deployment of Spark Integration Hub, follow the step 
   :ref:`Use an existing Spark Integration Hub deployment <use_existing_spark_integration_hub>` in place of this
   step.

Deploy the Spark Integration Hub charm in the ``kubeflow`` model by following the commands below.

.. code-block:: bash

   juju switch kubeflow
   juju deploy spark-integration-hub-k8s --channel=3/edge integration-hub --trust

Note that the ``--trust`` flag is essential when deploying the ``spark-integration-hub-k8s`` charm, for it 
to be able to create and watch resources in the Kubernetes cluster.

.. _use_existing_spark_integration_hub:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Use an existing Spark Integration Hub deployment
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. note::

   If you don't yet have a deployment of Spark Integration Hub charm, please follow the step
   :ref:`Deploy Spark Integration Hub <deploy_and_configure_spark_integration_hub>` in place of this step.

This step assumes that there is an existing deployment of Spark Integration Hub charm from channel ``3/edge`` with 
app name ``integration-hub`` in a separate Juju model named ``spark``.

Check if the ``spark-service-account`` endpoint is already offered by the Spark Integration Hub charm in the ``spark`` 
model.

.. code-block:: bash

   juju switch spark
   juju offers

If the offer doesn't exist, create one with the commands below:

.. code-block:: bash

   juju offer integration-hub:spark-service-account

You can then verify the offer has indeed been created with the command ``juju offers``.

Once the offer has been created, switch back to the ``kubeflow`` model to consume the offer as follows:

.. code-block:: bash

   juju switch kubeflow
   juju consume spark.integration-hub

If successful, you should now see ``integration-hub`` listed as a ``saas`` at the top when you run ``juju status`` 
command in the ``kubeflow`` model.


.. _deploy_data_kubeflow_integrator:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Deploy and configure Data-Kubeflow Integrator
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Within the ``kubeflow`` model, deploy the Data-Kubeflow Integrator as follows:

.. code-block:: bash

   juju deploy -m kubeflow data-kubeflow-integrator --channel=1/edge


Set the ``profile`` and ``spark-service-account`` config in the Data-Kubeflow Integrator charm to specify the name 
of Kubeflow profile where Spark is supposed to be enabled and the name of the Spark service account that is supposed
to be created respectively. 

.. code-block:: bash

   juju config -m kubeflow data-kubeflow-integrator \
       profile=* \
       spark-service-account=spark


Note that the value of ``profile`` is set as ``*`` to enable Spark in all Kubeflow profiles. Refer to 
`this documentation <https://canonical-charmed-spark.readthedocs-hosted.com/main/how-to/manage-service-accounts/>`_ 
to read more about the Spark service accounts.


.. _integrate_data_kubeflow_integrator_with_integration_hub:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Integrate Data-Kubeflow Integrator with Spark Integration Hub
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Integrate the Data-Kubeflow Integrator with Spark Integration Hub as follows:

.. code-block:: bash

   juju integrate -m kubeflow data-kubeflow-integrator integration-hub


.. _deploy_resource_dispatcher:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Deploy Resource Dispatcher
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Within the ``kubeflow`` model, deploy the Resource Dispatcher charm as follows:

.. code-block:: bash

   juju deploy resource-dispatcher --channel=latest/edge --trust

Note that the ``--trust`` flag is essential when deploying the ``resource-dispatcher`` charm, for it 
to be able to create resources in Kubernetes.

.. _integrate_resource_dispatcher_with_data_kubeflow_integrator:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Integrate Resource Dispatcher with Data-Kubeflow Integrator
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Integrate the Resource Dispatcher charm with Data-Kubeflow Integrator charm over the endpoints ``secrets``,
``service-accounts``, ``pod-defaults``, ``roles`` and ``role-bindings`` as follows:

.. code-block:: bash

   juju integrate resource-dispatcher:secrets data-kubeflow-integrator:secrets
   juju integrate resource-dispatcher:service-accounts data-kubeflow-integrator:service-accounts
   juju integrate resource-dispatcher:pod-defaults data-kubeflow-integrator:pod-defaults
   juju integrate resource-dispatcher:roles data-kubeflow-integrator:roles
   juju integrate resource-dispatcher:role-bindings data-kubeflow-integrator:role-bindings



.. _deploy_kubeflow_spark_solution_using_terraform:

------------------------------------------------------------------
Deploy a new CKF + Charmed Apache Spark solution using Terraform
------------------------------------------------------------------

Alternatively, you can deploy the entire Charmed Kubeflow-Spark solution from scratch on an existing Juju controller using Terraform.
This section of the guide assumes you have a working Juju K8s controller and the ``terraform`` and ``charmcraft`` CLI installed.

The steps to deploy the Charmed Kubeflow-Spark solution using Terraform is pretty similar to 
:ref:`Deploy Charmed Kubeflow using Terraform <install_terraform>`, the only difference being the Terraform module that is to be 
applied.

.. _fetch_kubeflow_spark_terraform_module:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Fetch the Charmed Kubeflow-Spark Terraform module
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Clone the ``charmed-kubeflow-solutions`` repository and change directory to the Kubeflow-Spark module as follows:

.. code-block:: bash

   git clone https://github.com/canonical/charmed-kubeflow-solutions
   cd charmed-kubeflow-solutions/modules/kubeflow-spark


.. _apply_terraform_module:

~~~~~~~~~~~~~~~~~~~~~~~~~~~
Apply the Terraform module
~~~~~~~~~~~~~~~~~~~~~~~~~~~

First of all, initialise Terraform. The following command downloads all the required 
`Terraform modules <https://developer.hashicorp.com/terraform/language/modules>`_ and installs the Terraform 
`Juju provider <https://registry.terraform.io/providers/juju/juju/latest/docs>`_.

.. code-block:: bash

   terraform init

Define credentials that will later be used to log into Kubeflow dashboard.

.. code-block:: bash

   DEX_USERNAME="your-username"
   DEX_PASSWORD="your-password"

Finally, deploy Charmed Kubeflow-Spark solution using Terraform as follows:

.. code-block:: bash

   terraform apply \
      -var dex_static_username=$DEX_USERNAME \
      -var dex_static_password=$DEX_PASSWORD \
      -var metacontroller_operator_revision=551 \
      -var resource_dispatcher_revision=434

The command above:

* Creates a `Juju model <https://juju.is/docs/juju/model>`_ named ``kubeflow``.  
* Deploys CKF ``1.10/stable``.  
* Deploys charms like Spark Integration Hub, Data-Kubeflow Integrator and Resource Dispatcher that are necessary 
  for Spark integration.
* Configures `dex-auth <https://charmhub.io/dex-auth>`_ charm with a static user username and password.  
* Deploys a specific revision of `metacontroller-operator <https://charmhub.io/metacontroller-operator>`_ 
  and `resource-dispatcher <https://charmhub.io/resource-dispatcher>`_ charms because the changes necessary for 
  Spark integration aren't released to the stable channel yet.

Wait until the deployment is complete, and the ``terraform apply`` command returns.


.. _verify_kubeflow_spark_deployment:

~~~~~~~~~~~~~~~~~~~~~~~
Verify the deployment
~~~~~~~~~~~~~~~~~~~~~~~

As the first step, verify all charms are in ``active`` status by monitoring the Juju model:

.. code-block:: bash

   juju switch kubeflow
   juju status --watch 1s

.. note::

   This may take up to some minutes, depending on the cluster's node specifications.

For additional validation, Charmed Kubeflow-Spark User Acceptance Tests (UAT) can be run on top of this deployment
to verify that the deployment was indeed successful and that Spark is enabled in the Kubeflow notebooks and pipelines.

To run the UAT on top of the deployment, first fetch the UAT tests from the ``charmed-kubeflow-uats`` repository:

.. code-block:: bash

   git clone https://github.com/canonical/charmed-kubeflow-uats.git
   cd charmed-kubeflow-uats

Now run the following command to run the UAT:

.. code-block:: bash

   tox -e spark-remote -- \
       --test-image ghcr.io/canonical/charmed-spark-jupyterlab:3.5-22.04_edge

This will run the tests to verify that Spark is enabled in both Kubeflow notebooks and Kubeflow pipeline steps.

---------------------
Access CKF dashboard
---------------------

Once you have Charmed Kubeflow deployment along with the Spark support by following the instructions above, you can
now access the CKF dashboard through an IP address. 

See :ref:`Access CKF dashboard <access_ckf_dashboard>` for more details on how to access the CKF dashboard.
See :ref:`Run Spark jobs <run_spark_jobs>` for running sample Spark jobs using Kubeflow notebook and pipelines.