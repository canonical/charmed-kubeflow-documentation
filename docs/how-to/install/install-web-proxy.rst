.. _install_web_proxy:

Install behind a web proxy
==========================

This guide describes how to install Charmed Kubeflow (CKF) behind a web proxy.

-------------------------
Prepare your environment
-------------------------

Before installing CKF, first you need to set up your client with the required proxy settings.

~~~~~~~~~~~~~~~~~~~
Configure snap
~~~~~~~~~~~~~~~~~~~

Save the value of your proxy server address for reuse:

.. code-block:: bash

   PROXY=http://<username>:<password>@<proxy IP>:<proxy port>/

.. note::

   Add the ``username:<password>@`` part only if the proxy server is configured with credentials, check with your network administrator.

Set the snap proxy settings:

.. code-block:: bash

   sudo snap set system proxy.http=$PROXY
   sudo snap set system proxy.https=$PROXY

This will enable you to install snap packages.

Now restart the snap service:

.. code-block:: bash

   sudo systemctl restart snapd.service

~~~~~~~~~~~~~~~~~~~
Configure Juju
~~~~~~~~~~~~~~~~~~~

Configure Juju with the proxy settings. Beside the `http_proxy` and `https_proxy`, it is also important to set the `no_proxy` environment variable to make sure that requests within Kubernetes and among the various K8s services are not sent through the proxy. 

.. code-block:: bash

    export http_proxy=$PROXY
    export https_proxy=$PROXY

.. code-block:: bash

    export no_proxy=$CLUSTER_CIDR,\
    $SERVICE_CIDR,\
    127.0.0.1,\
    localhost,\
    $NODE_IP/24,\
    $HOSTNAME,\
    .svc,\
    .local,\
    .kubeflow

Make sure to replace ``<hostname>``, ``CLUSTER_CIDR``, ``SERVICE_CIDR`` and ``NODE_IP`` with the settings for your environment.
As an example, on a local deployment of MicroK8s you can use the following snippet:

.. code-block:: bash

    export CLUSTER_CIDR=$(cat /var/snap/microk8s/current/args/kube-proxy | grep cluster-cidr | sed 's/^[^=]*=//')
    export SERVICE_CIDR=$(cat /var/snap/microk8s/current/args/kube-apiserver | grep service-cluster-ip-range | sed 's/^[^=]*=//')
    export NODE_IP=$(kubectl get nodes -o jsonpath='{.items[0].status.addresses[?(@.type=="InternalIP")].address}')
    export HOSTNAME=$(hostname)
    export NO_PROXY="$CLUSTER_CIDR,$SERVICE_CIDR,127.0.0.1,localhost,$NODE_IP/24,$HOSTNAME,.svc,.local,.kubeflow"

For more information on how to set variables on MicroK8s, please refer to [this guide](https://github.com/canonical/charmed-kubeflow-documentation/blob/95d5ce435492bf3d976c2b45928a682b0058693c/docs/how-to/install/install-web-proxy.rst#configure-microk8s).

Once the proxy environment variables are correctly set, install Juju:

.. code-block:: bash

    sudo snap install juju --classic --channel=3.6/stable

Create a Juju controller in your cluster and set the proxy model default values. 

.. code-block:: bash

    juju bootstrap microk8s micro --model-default juju-http-proxy=$http_proxy \
    --model-default juju-https-proxy=$https_proxy \
    --model-default juju-no-proxy=$no_proxy


When using a controller that has already been bootstrapped, the proxy settings can also be set afterwards using `juju model-config`, e.g.

.. code-block:: bash

    juju switch controller
    juju model-config juju-http-proxy=$http_proxy juju-https-proxy=$https_proxy juju-no-proxy=$no_proxy

---------------------
Deploy CKF
---------------------

To deploy CKF, follow the steps provided in the :ref:`general installation <general_installation>` guide. 

When deploying with terraform, make sure you are providing the proxy input variables, e.g. using a `tfvars.json` file

.. code-block:: json

   {
	...
	"http_proxy": <http_proxy>,
	"https_proxy": <https_proxy>,
	"no_proxy": <no_proxy>
	...
   }


If you are re-using an existing model, make sure the ``kubeflow`` model has your proxy settings, run:

.. code-block:: bash

   juju model-config

You should see the proxy settings in the ``juju-http-proxy``, ``juju-https-proxy`` and ``juju-no-proxy`` variables. If these are not set, proceed to set them to their correct values using the `juju model-config <key>=<value> command, as shonw above. 

--------------------------------------
Use Kubeflow components behind a proxy
--------------------------------------

The following sections provides information on how to set and manage the proxy values for the various kind of Kubeflow user workloads. However, note that how to set the proxy values may depend on the specific type of user workload and on their requirement to reach external resources. In particular, some python libraries may not handle the `no_proxy` environment variable correctly or they may not work with CIDR. In this specific cases, we recommend you to disable proxy to ensure that internal requests are not sent and blocked by the proxy.  

~~~~~~~~~~~~~~~~~~~
Notebooks
~~~~~~~~~~~~~~~~~~~

Apply the following PodDefault to your user namespace so each notebook you create will have proxy configurations set.
The ``NO_PROXY`` and ``no_proxy`` values would be the same as you configured in the Juju model.

.. code-block:: bash

   cat <<EOF | kubectl apply -n $USER_NAMESPACE -f -
   apiVersion: kubeflow.org/v1alpha1
   kind: PodDefault
   metadata:
        name: notebook-proxy
   spec:
        desc: Add proxy settings
        env:
            - name: HTTP_PROXY
            value: http://10.0.1.119:3128/ # replace with $PROXY
            - name: http_proxy
            value: http://10.0.1.119:3128/ # replace with $PROXY
            - name: HTTPS_PROXY
            value: http://10.0.1.119:3128/ # replace with $PROXY
            - name: https_proxy
            value: http://10.0.1.119:3128/ # replace with $PROXY
            - name: NO_PROXY
            value: <cluster cidr>,<service cluster ip range>,127.0.0.1,<nodes internal ip(s)>/24,<cluster hostname>,.svc,.local
            - name: no_proxy
        value: <cluster cidr>,<service cluster ip range>,127.0.0.1,<nodes internal ip(s)>/24,<cluster hostname>,.svc,.local,.kubeflow
        selector:
        matchLabels:
             notebook-proxy: "true"
   EOF

You should now be able to see ``Add proxy settings`` when creating a new notebook under ``Advanced Options > Configurations``. 
Always select that option.

.. image:: https://assets.ubuntu.com/v1/49809832-web-proxy-ckf.png

~~~~~~~~~~~~~~~~~~~
Katib
~~~~~~~~~~~~~~~~~~~

Before running a Katib experiment, add your proxy environment variables to your experiment definition for each container under ``spec.trialTemplate.trialSpec.spec.template.spec.containers``:

.. code-block:: bash

   env:
      - name: HTTP_PROXY
      value: http://10.0.1.119:3128/ # replace with $PROXY
      - name: http_proxy
      value: http://10.0.1.119:3128/ # replace with $PROXY
      - name: HTTPS_PROXY
      value: http://10.0.1.119:3128/ # replace with $PROXY
      - name: https_proxy
      value: http://10.0.1.119:3128/ # replace with $PROXY

See here a full Katib experiment example:

.. code-block:: bash

   apiVersion: kubeflow.org/v1beta1
   kind: Experiment
   metadata:
   name: grid-proxy
   spec:
   objective:
      type: maximize
      goal: 0.99
      objectiveMetricName: Validation-accuracy
      additionalMetricNames:
         - Train-accuracy
   algorithm:
      algorithmName: grid
   parallelTrialCount: 1
   maxTrialCount: 1
   maxFailedTrialCount: 1
   parameters:
      - name: lr
         parameterType: double
         feasibleSpace:
         min: "0.001"
         max: "0.01"
         step: "0.001"
      - name: num-layers
         parameterType: int
         feasibleSpace:
         min: "2"
         max: "5"
      - name: optimizer
         parameterType: categorical
         feasibleSpace:
         list:
            - sgd
            - adam
            - ftrl
   trialTemplate:
      primaryContainerName: training-container
      trialParameters:
         - name: learningRate
         description: Learning rate for the training model
         reference: lr
         - name: numberLayers
         description: Number of training model layers
         reference: num-layers
         - name: optimizer
         description: Training model optimizer (sdg, adam or ftrl)
         reference: optimizer
      trialSpec:
         apiVersion: batch/v1
         kind: Job
         spec:
         template:
            metadata:
               annotations:
               sidecar.istio.io/inject: "false"
            spec:
               containers:
               - name: training-container
                  image: docker.io/kubeflowkatib/mxnet-mnist:latest
                  command:
                     - "python3"
                     - "/opt/mxnet-mnist/mnist.py"
                     - "--batch-size=64"
                     - "--lr=${trialParameters.learningRate}"
                     - "--num-layers=${trialParameters.numberLayers}"
                     - "--optimizer=${trialParameters.optimizer}"
                  env:
                     - name: HTTP_PROXY
                     value: http://10.0.1.119:3128/
                     - name: http_proxy
                     value: http://10.0.1.119:3128/
                     - name: HTTPS_PROXY
                     value: http://10.0.1.119:3128/
                     - name: https_proxy
                     value: http://10.0.1.119:3128/
               restartPolicy: Never

~~~~~~~~~~~~~~~~~~~
Pipelines
~~~~~~~~~~~~~~~~~~~

If your pipeline needs to download data or pull an image, you can inject your proxy environment variables into a pipeline from inside a notebook with the KFP SDK as done in `this example notebook <https://raw.githubusercontent.com/Barteus/kubeflow-examples/main/e2e-wine-kfp-mlflow/proxy-e2e-kfp-mlflow-seldon-pipeline.ipynb>`_.

~~~~~~~~~~~~~~~~~~~
Istio
~~~~~~~~~~~~~~~~~~~

If needed, configure proxy settings for Istio as follows:

.. code-block:: bash

   kubectl apply -n kubeflow -f - <<EOF
   apiVersion: networking.istio.io/v1beta1
   kind: ServiceEntry
   metadata:
     name: proxy
   spec:
     hosts:
     - my-company-proxy.com # ignored
     addresses:
     - 10.0.1.119/32 # replace with proxy IP
     ports:
     - number: 3128 # replace with proxy port
       name: tcp
       protocol: TCP
     location: MESH_EXTERNAL
   EOF
