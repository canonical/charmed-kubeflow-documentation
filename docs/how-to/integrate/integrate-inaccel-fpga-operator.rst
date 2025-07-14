.. _integrate_inaccel_fpga_operator:

Integrate with InAccel FPGA Operator
====================================

This guide describes how to leverage FPGA-hardware with Kubernetes (K8s) for accelerating Machine Learning (ML) experiments.

Specifically, you will learn how to enable the `InAccel FPGA Operator <https://artifacthub.io/packages/helm/inaccel/fpga-operator>`_ in `MicroK8s`_. 
to boost the hyperparameter tuning of some ML models using Charmed Kubeflow (CKF) `Katib component <https://kubeflow.org/docs/components/katib>`_.

The FPGA Operator is particularly useful for scenarios where the K8s cluster needs to scale quickly. 
For example provisioning additional FPGA nodes on the cloud and managing the lifecycle of the underlying software components.

---------------------
Requirements
---------------------

- `Amazon EC2 F1 instance <https://aws.amazon.com/ec2/instance-types/f1/>`_.
- A MicroK8s version >= 1.23 cluster running.

----------------------------
Enable InAccel FPGA Operator
----------------------------

InAccel FPGA Operator is already built into MicroK8s as an add-on. 
You can enable it after installing MicroK8s as follows:

.. code-block:: bash

   microk8s enable inaccel --wait

----------------------
Get started with Katib
----------------------

~~~~~~~~~~~~~~~~~~~~~
Enable local storage
~~~~~~~~~~~~~~~~~~~~~

Your MicroK8s cluster must have dynamic volume provisioning for Katib. 
You can enable the local storage service as follows:

.. code-block:: bash

   microk8s enable storage

~~~~~~~~~~~~~~~~~~~
Install Katib
~~~~~~~~~~~~~~~~~~~

1. Install `Juju`_, the operation lifecycle manager you will use it to deploy and manage Katib components.
   You can install it from a `snap`_ package:

   .. code-block:: bash

      sudo snap install juju --classic

2. Create a `Juju controller <https://documentation.ubuntu.com/juju/3.6/reference/controller/>`_:

   .. code-block:: bash

      juju bootstrap microk8s

3. Create the Katib model. A model in Juju matches with a K8s namespace:

   .. code-block:: bash

      juju add-model katib

4. Deploy the Katib bundle.

   .. code-block:: bash

      juju deploy katib

5. Create a namespace for running experiments, with `Katib metrics collector <https://www.kubeflow.org/docs/components/katib/user-guides/metrics-collector/>`_ enabled:

   .. code-block:: bash

      microk8s kubectl create namespace kubeflow
      microk8s kubectl label namespace kubeflow katib-metricscollector-injection=enabled katib.kubeflow.org/metrics-collector-injection=enabled

~~~~~~~~~~~~~~~~~~~
Verify installation
~~~~~~~~~~~~~~~~~~~

Run the following command to verify that Katib components are running:

.. code-block:: bash

   watch microk8s kubectl get --namespace katib pods

~~~~~~~~~~~~~~~~~~~
Access Katib UI
~~~~~~~~~~~~~~~~~~~

You can use the Katib User Interface (UI) to submit experiments and monitor results. 
The Katib home page looks like this:

.. image:: https://assets.ubuntu.com/v1/f209a2b3-acces_katib_UI.png

You can set port-forwarding for the Katib UI service:

.. code-block:: bash

   microk8s kubectl port-forward --namespace katib svc/katib-ui 8080:8080 --address 0.0.0.0

Now access the Katib UI at ``http://localhost:8080/katib``.

---------------------
Run an experiment
---------------------

The steps to configure and `run a hyperparameter tuning experiment <https://www.kubeflow.org/docs/components/katib/experiment>`_ in Katib are:

1. Package your training code in a Docker container image and make the image available in a registry.
2. Define the experiment in a YAML configuration file. The YAML file defines the range of potential values (the search space) for the parameters that you want to optimize, the objective metric to use when determining optimal values, the search algorithm to use during optimization, and other configurations.
3. Run the experiment from the Katib UI, either by supplying the entire YAML file containing the configuration or by entering the configuration values into the form.

As a reference, see this `FPGA XGBoost YAML file <https://github.com/kubeflow/katib/blob/master/examples/v1beta1/fpga/xgboost-example.yaml>`_.

For this example, you will use the SVHN image dataset obtained from Google Street View.

~~~~~~~~~~~~~~~~~~~~~
Create the experiment
~~~~~~~~~~~~~~~~~~~~~

Click on ``NEW EXPERIMENT`` on the Katib home page with the following options:

1. Metadata. The experiment name, for example, ``xgb-svhn-fpga``.

.. image:: https://assets.ubuntu.com/v1/a999e941-exp1.png

2. Trial thresholds. Use ``parallel trials`` to limit the number of hyperparameter sets that Katib should train in parallel.

.. image:: https://assets.ubuntu.com/v1/9d6a3e5e-exp2.png

3. Objective. The metric that you want to optimize. Use ``Additional metrics`` to monitor how the hyperparameters work with the model.

.. image:: https://assets.ubuntu.com/v1/83be19ac-exp3.png

4. Hyperparameters. The range of potential values for the parameters that you want to optimize. In this section, you define the name and the distribution of every hyperparameter. For example, you may provide a minimum and maximum value or a list of allowed values for each hyperparameter. Katib generates hyperparameter combinations in the range.

.. image:: https://assets.ubuntu.com/v1/bb49286b-exp4.png

5. Trial template. You have to package your ML training code into a Docker image. Your training container can receive hyperparameters as command-line arguments or as environment variables.

.. image:: https://assets.ubuntu.com/v1/81cee022-exp5.png

Here's a ``YAML`` file example for a FPGA-accelerated trial job:

.. code-block:: yaml

   apiVersion: batch/v1
   kind: Job
   spec:
     template:
       metadata:
         labels:
           inaccel/fpga: enabled
         annotations:
           inaccel/cli: |
             bitstream install --mode others https://store.inaccel.com/artifactory/bitstreams/xilinx/aws-vu9p-f1/shell-v04261818_201920.2/aws/com/inaccel/xgboost/0.1/2exact
       spec:
         containers:
           - name: training-container
             image: "docker.io/inaccel/jupyter:lab"
             command:
               - python3
               - XGBoost/parameter-tuning.py
             args:
               - "--name=SVHN"
               - "--test-size=0.35"
               - "--tree-method=fpga_exact"
               - "--max-depth=10"
               - "--alpha=${trialParameters.alpha}"
               - "--eta=${trialParameters.eta}"
               - "--subsample=${trialParameters.subsample}"
             resources:
               limits:
                 xilinx/aws-vu9p-f1: 1
         restartPolicy: Never

Here's a ``YAML`` file example for a CPU-only trial job:

.. code-block:: yaml

   apiVersion: batch/v1
   kind: Job
   spec:
     template:
       spec:
         containers:
           - name: training-container
             image: "docker.io/inaccel/jupyter:lab"
             command:
               - python3
               - XGBoost/parameter-tuning.py
             args:
               - "--name=SVHN"
               - "--test-size=0.35"
               - "--max-depth=10"
               - "--alpha=${trialParameters.alpha}"
               - "--eta=${trialParameters.eta}"
               - "--subsample=${trialParameters.subsample}"
         restartPolicy: Never

~~~~~~~~~~~~~~~~~~~
Check the results
~~~~~~~~~~~~~~~~~~~

1. Go to the Katib UI to view the list of experiments. Select the ``xgb-svhn-fpga`` experiment.

2. See the results graph showing the level of validation accuracy and train time for various combinations of the hyperparameter values, such as alpha, eta, and subsample:

   * FPGA-accelerated:

   .. image:: https://assets.ubuntu.com/v1/fc5d1923-results1.jpeg

   * CPU-only:

   .. image:: https://assets.ubuntu.com/v1/4b2bd231-results2.jpeg

Comparing the FPGA-accelerated experiment with the equivalent CPU-only one, you will notice that the accuracy of the best model is similar in both implementations.
However, the performance of the 8-core Intel Xeon CPU of the AWS F1 instance is significantly (~6 times) worse than its single (1) Xilinx VU9P FPGA for this use case.

---------------------
Clean up resources
---------------------

You can stop your FPGA instance as follows:

.. code-block:: bash

   aws ec2 stop-instances \
       --instance-ids <InstanceId> \
   | jq -r '.StoppingInstances[0].CurrentState.Name'
