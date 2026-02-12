.. _integrate_spark:

Integrate with Charmed Apache Spark
============================================

.. note::

   This feature is experimental and therefore should not to be used in a production environment.

This guide describes how Charmed Kubeflow (CKF) and the 
`Charmed Apache Spark <https://canonical-charmed-spark.readthedocs-hosted.com/main/>`_ can be integrated using 
`Juju`_. This integration enables running Spark jobs in Kubeflow Notebooks and Kubeflow Pipelines.

.. _kf_spark_requirements:

---------------------
Requirements
---------------------

Minimum requirements for this guide are:

- A system with at least 8 cores CPU processor, 64GB of RAM and 150GB of disk space.
- `MicroK8s`_ and Juju (agent version ``<=3.6.9``) installed and configured in the system. 
  See :ref:`CKF supported versions <supported_kubeflow_versions>` for more details about compatible versions 
  of `Kubeflow <https://www.kubeflow.org/docs/releases/>`_ and Juju.
- ``charmcraft`` and ``tox``
- ``terraform``, if you chose to 
  :ref:`deploy CKF + Charmed Apache Spark solution using Terraform <deploy_kubeflow_spark_solution_using_terraform>`.

.. _integrate_with_existing_kubeflow_deployment:

-------------------------------------------------
Integrate Spark with an existing CKF deployment
-------------------------------------------------

This section of the guide assumes that you already have a CKF deployment in a Juju model named ``kubeflow`` in your 
Juju cloud. If not, please see :ref:`CKF getting started guide <get_started>` for more details on how to do so, or 
refer to :ref:`Deploy Kubeflow-Spark solution using Terraform <deploy_kubeflow_spark_solution_using_terraform>` 
section below.

.. note::

   When using an existing CKF deployment, ensure that the ``metacontroller-operator`` charm is up to date with the 
   ``4.11/edge`` channel, since the changes that support Charmed Apache Spark integration are not yet merged 
   to the stable channel.

To integrate Charmed Kubeflow with Charmed Apache Spark, you need to deploy Spark Integration Hub, Data-Kubeflow
Integrator and Resource Dispatcher charms, configure them and integrate them with Juju relations. The following 
sections describe these actions in detail.

.. _prepare_spark_integration_hub:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Spark Integration Hub setup
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Charmed Kubeflow is integrated with the Charmed Apache Spark ecosystem with the help of the Spark Integration Hub
charm. You can either use an existing deployment of this charm, or deploy a new instance of this charm as described
in the subsections below.


.. _use_existing_spark_integration_hub:

"""""""""""""""""""""""""""""""""""""""""""""""""""
Use an existing Spark Integration Hub deployment
"""""""""""""""""""""""""""""""""""""""""""""""""""

.. note::

   If you don't yet have a deployment of Spark Integration Hub charm, please follow the step
   :ref:`Deploy a new instance of Spark Integration Hub <deploy_and_configure_spark_integration_hub>` in place of this step.

This step assumes that there is an existing deployment of Spark Integration Hub charm from channel ``3/edge`` with 
app name ``integration-hub`` in a separate Juju model named ``spark``.

Check if the ``spark-service-account`` endpoint is already offered by the Spark Integration Hub charm in the ``spark`` 
model.

.. code-block:: bash

   juju switch kubeflow
   juju find-offers

If the offer doesn't exist, create one with the commands below:

.. code-block:: bash

   juju switch spark
   juju offer integration-hub:spark-service-account

You can then verify the offer has indeed been created with the command ``juju offers``.

Once you verify that the offer is available for use, switch back to the ``kubeflow`` model to consume the offer as 
follows:

.. code-block:: bash

   juju switch kubeflow
   juju consume spark.integration-hub

If successful, you should now see ``integration-hub`` listed as a ``saas`` at the top when you run ``juju status`` 
command in the ``kubeflow`` model.


.. _deploy_and_configure_spark_integration_hub:

"""""""""""""""""""""""""""""""""""""""""""""""
Deploy a new instance of Spark Integration Hub
"""""""""""""""""""""""""""""""""""""""""""""""

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

.. _deploy_data_kubeflow_integrator:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Data-Kubeflow Integrator setup
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Within the ``kubeflow`` model, deploy the Data-Kubeflow Integrator as follows:

.. code-block:: bash

   juju deploy data-kubeflow-integrator --channel=1/edge


Set the ``profile`` and ``spark-service-account`` config in the Data-Kubeflow Integrator charm to specify the name 
of Kubeflow profile where Spark is supposed to be enabled and the name of the Spark service account that is supposed
to be created respectively. 

.. code-block:: bash

   juju config data-kubeflow-integrator \
       profile=* \
       spark-service-account=spark


Note that the value of ``profile`` is set as ``*`` to enable Spark in all Kubeflow profiles. Refer to 
`this documentation <https://canonical-charmed-spark.readthedocs-hosted.com/main/how-to/manage-service-accounts/>`_ 
to read more about the Spark service accounts.

Integrate the Data-Kubeflow Integrator with Spark Integration Hub as follows:

.. code-block:: bash

   juju integrate data-kubeflow-integrator integration-hub


.. _deploy_resource_dispatcher:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Resource Dispatcher setup
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Within the ``kubeflow`` model, deploy the Resource Dispatcher charm as follows:

.. code-block:: bash

   juju deploy resource-dispatcher --channel=latest/edge --trust

Note that the ``--trust`` flag is essential when deploying the ``resource-dispatcher`` charm, for it 
to be able to create resources in Kubernetes.

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
Deploy CKF + Charmed Apache Spark solution using Terraform
------------------------------------------------------------------

Alternatively, you can deploy the entire Charmed Kubeflow-Spark solution from scratch on an existing Juju controller using Terraform.
This section of the guide assumes you have a working Juju K8s controller and the ``terraform`` and ``charmcraft`` CLI installed.


Clone the ``charmed-kubeflow-solutions`` repository and change directory to the Kubeflow-Spark module as follows:

.. code-block:: bash

   git clone https://github.com/canonical/charmed-kubeflow-solutions
   cd charmed-kubeflow-solutions/modules/kubeflow-spark


Initialise Terraform. The following command downloads all the required 
`Terraform modules <https://developer.hashicorp.com/terraform/language/modules>`_ and installs the Terraform 
`Juju provider <https://registry.terraform.io/providers/juju/juju/latest/docs>`_:

.. code-block:: bash

   terraform init

Define credentials that will later be used to log into Kubeflow dashboard:

.. code-block:: bash

   DEX_USERNAME="your-username"
   DEX_PASSWORD="your-password"

Finally, deploy Charmed Kubeflow-Spark solution using Terraform as follows:

.. code-block:: bash

   terraform apply \
      -var dex_static_username=$DEX_USERNAME \
      -var dex_static_password=$DEX_PASSWORD \
      -var risk=edge

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

-----------------------
Verify the deployment
-----------------------

As the first step, verify all charms are in ``active`` status by monitoring the Juju model:

.. code-block:: bash

   juju switch kubeflow
   watch -n 1 "juju status"

.. note::

   This may take up to some minutes, depending on the cluster's node specifications.

For additional validation, Charmed Kubeflow-Spark User Acceptance Tests (UAT) can be run on top of this deployment
to verify that the deployment was indeed successful and that Spark is enabled in the Kubeflow notebooks and pipelines.

To run the UAT on top of the deployment, first fetch the UAT tests from the ``charmed-kubeflow-uats`` repository:

.. code-block:: bash

   git clone https://github.com/canonical/charmed-kubeflow-uats.git
   cd charmed-kubeflow-uats

Now run the following command to run the UAT:

.. note::

   Please make sure to use the correct Charmed Apache Spark Jupyterlab image for Apache Spark version of your choice. 
   In the command below, the image for Apache Spark 3.5 is used. OCI image corresponding to other versions of Spark 
   can be found `here <https://github.com/canonical/charmed-spark-rock/pkgs/container/charmed-spark-jupyterlab>`_.

.. code-block:: bash

   tox -e spark-remote -- \
       --test-image ghcr.io/canonical/charmed-spark-jupyterlab:3.5-22.04_edge@sha256:72a6e89985e35e0920fb40c063b3287425760ebf823b129a87143d5ec0e99af7  \
       --bundle ''

This will run the tests to verify that Spark is enabled in both Kubeflow Notebooks and Kubeflow Pipeline steps. The test
takes around five minutes to complete and you should see some lines similar to the following lines at the end of the output,
if the test was successful.

.. code-block:: bash

   spark-remote: OK (223.32=setup[0.07]+cmd[1.15,222.10] seconds)
   congratulations :) (223.35 seconds)


---------------------------------------
Access CKF dashboard to run Spark jobs
---------------------------------------

Once you have Charmed Kubeflow deployment along with the Spark support by following the instructions above, you can
now access the CKF dashboard through an IP address. See :ref:`Access CKF dashboard <access_ckf_dashboard>` for more 
details on how to access the CKF dashboard.

Using the Kubeflow dashboard, you can now create Kubeflow Notebooks and Kubeflow Pipelines and write code to run Spark
jobs within them. See :ref:`Run Spark jobs <run_spark_jobs>` for the guide on how to run sample Spark jobs using 
Kubeflow Notebook and Kubeflow Pipeline.