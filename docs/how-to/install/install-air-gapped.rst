.. _install_airgapped:

Install in an air-gapped environment
====================================

This guide describes how to install Charmed Kubeflow (CKF) in an air-gapped environment, that is, an environment without internet access.

---------------------
Requirements
---------------------

Your environment should meet the following criteria:

* A Kubernetes (K8s) cluster is running.
* A container registry such as Artifactory is reachable from the K8s cluster over HTTPS.

~~~~~~~~~~~~~~~~~~~
MicroK8s DNS
~~~~~~~~~~~~~~~~~~~

If you are using MicroK8s, the DNS add-on should be configured to the host's local nameserver. 
This can be done as follows:

.. code-block:: bash

   microk8s enable dns:$(resolvectl status | grep "Current DNS Server" | awk '{print $NF}')

---------------------
Process outline
---------------------

1. Artefact Generation.
2. Set up an air-gapped environment with a K8s cluster and HTTPS-enabled registry.
3. Extract and load the images from ``images.tar.gz`` into their container registry.
4. Extract all charms from ``charms.tar.gz``.
5. Setup Juju in the air-gapped cluster.
6. Deploy CKF.

---------------------
Generate artefacts
---------------------

The following artifacts must be generated: ``images.tar.gz``, ``charms.tar.gz``. 
To generate those, you'll need to utilise CKF helper scripts that scan a CKF release and gather all the charm and images files.

Clone the ``bundle-kubeflow`` repository:

.. code-block:: bash

   git clone https://github.com/canonical/bundle-kubeflow.git

Change directory to the Airgap utility scripts one:

.. code-block:: bash

   cd bundle-kubeflow/scripts/airgapped

Install the pre-requisites of the utility scripts:

.. code-block:: bash

   pip3 install -r requirements.txt
   sudo apt install pigz
   sudo snap install docker
   sudo snap install yq
   sudo snap install jq

Get a list of all the images for the CKF bundle you are deploying. 
For example, to get the list of images for Charmed Kubeflow 1.8:

.. code-block:: bash

   ./scripts/airgapped/get-all-images.sh releases/1.8/stable/kubeflow/bundle.yaml > images.txt

Pull the images to your Docker cache using the ``save-images-to-cache.py`` script:

.. code-block:: bash

   python3 scripts/airgapped/save-images-to-cache.py images.txt

Rename the images in the Docker cache to have the URL of the registry in your air-gapped environment:

.. code-block:: bash

   python3 scripts/airgapped/retag-images-to-cache.py --new-registry=<your air-gap registry> images.txt

Save the images to ``images.tar.gz``:

.. code-block:: bash

   python3 scripts/airgapped/save-images-to-tar.py retagged-images.txt

Save the charms to ``charms.tar.gz``:

.. code-block:: bash

   BUNDLE_PATH=releases/1.8/stable/kubeflow/bundle.yaml

   python3 scripts/airgapped/save-charms-to-tar.py $BUNDLE_PATH

---------------------
Extract artefacts
---------------------

Both charms and OCI images must be extracted. 
Charms are extracted to the same machine as the Juju client. 
OCI images are pushed to the private container registry running in their air-gapped environment.

1. Move ``charms.tar.gz`` to the air-gapped machine, then extract it to ``~/charms`` directory. This directory will be used in the deployment step:

.. code-block:: bash

   mkdir charms
   tar -xzvf charms.tar.gz --directory charms

2. Move ``retagged-images.txt`` generated in the previous step to the air-gapped machine under the ``$HOME`` directory. This is also needed for the deployment step.

3. Move ``images.tar.gz`` to the air-gapped machine, then load the images into the private registry. Here's an example:

.. code-block:: bash

   # Extract the images from tar
   mkdir images
   tar -xzvf images.tar.gz --directory images
   rm images.tar.gz

   # Load the images into intermediate Docker client
   for img in images/*.tar; do docker load < $img && rm $img; done
   rmdir images

   # Push the images from local docker to Registry
   python3 scripts/airgapped/push-images-to-registry.py retagged-images.txt

Additionally you need to import the charms' ubuntu base images to your private registry. The images are:

.. code-block:: bash

   docker.io/jujusolutions/charm-base:ubuntu-20.04
   docker.io/jujusolutions/charm-base:ubuntu-22.04

---------------------
Setup Juju
---------------------

See `Juju airgapped <hhttps://documentation.ubuntu.com/juju/latest/howto/manage-your-deployment/#set-up-your-deployment-offline>`_ for details.

---------------------
Deploy CKF
---------------------

To deploy CKF, use `Air-gapped deployment script <https://github.com/canonical/bundle-kubeflow/blob/main/scripts/airgapped/deploy-1.8.sh>`_.

The script assumes:

1. A ``retagged-images.txt`` file exists in the home directory of your air-gapped machine. 
It contains a list of all the images needed for Charmed Kubeflow, where each image is defined with the air-gapped registry. 
Here's a sample of ``retagged-images.txt``:

.. code-block:: bash

   # retagged-images.txt
   172.17.0.2:5000/argoproj/argocli:v3.3.10
   172.17.0.2:5000/argoproj/workflow-controller:v3.3.10
   172.17.0.2:5000/charmedkubeflow/api-server:2.0.5-63c48d5
   172.17.0.2:5000/charmedkubeflow/argoexec:3.3.10-c88862f
   172.17.0.2:5000/charmedkubeflow/dex:2.36.0-f262d95

In the example above, the air-gapped registry is ``172.17.0.2:5000``.

2. A ``charms`` directory exists in the home directory of your air-gapped machine. 
t contains all the charm files to be deployed in Charmed Kubeflow. Here's a sample of ``~/charms`` directory:

.. code-block:: bash

   ls ~/charms
   admission-webhook_r301.charm   jupyter-ui_r858.charm           kfp-profile-controller_r1278.charm  knative-serving_r354.charm          minio_r278.charm               tensorboard-controller_r257.charm
   argo-controller_r424.charm     katib-controller_r446.charm     kfp-schedwf_r1302.charm             kserve-controller_r523.charm        mlmd_r127.charm                tensorboards-web-app_r245.charm
   dex-auth_r422.charm            katib-db-manager_r411.charm     kfp-ui_r1285.charm                  kubeflow-dashboard_r454.charm       mysql-k8s_r127.charm           training-operator_r347.charm
   envoy_r194.charm               katib-ui_r422.charm             kfp-viewer_r1317.charm              kubeflow-profiles_r355.charm        oidc-gatekeeper_r350.charm
   istio-gateway_r723.charm       kfp-api_r1283.charm             kfp-viz_r1235.charm                 kubeflow-roles_r187.charm           pvcviewer-operator_r30.charm
   istio-pilot_r827.charm         kfp-metadata-writer_r334.charm  knative-eventing_r353.charm         kubeflow-volumes_r260.charm         resource-dispatcher_r93.charm
   jupyter-controller_r849.charm  kfp-persistence_r1291.charm     knative-operator_r328.charm         metacontroller-operator_r252.charm  seldon-core_r664.charm

Once you meet the requirements above, you can add the ``kubeflow`` model and run the deploy script:

.. code-block:: bash

   juju add-model kubeflow

   bash deploy-1.8.sh

~~~~~~~~~~~~~~~~~~~~~
Gateway service type
~~~~~~~~~~~~~~~~~~~~~

In ``deploy-1.8.sh`` script, the ``gateway_service_type`` for the `Istio Gateway configuration <https://charmhub.io/istio-gateway/configure?channel=latest/edge>`_ is set to ``LoadBalancer``. 
However, if you don't have a load balancer within your cluster, you can configure the service to ``NodePort`` by adding ``--config gateway_service_type="NodePort"`` to the ``istio-ingressgateway`` deploy command. 
The changes in the ``deploy-1.8.sh`` script are as follows:

.. code-block:: bash

   -juju deploy --trust --debug ./$(charm istio-gateway) istio-ingressgateway --config kind=ingress --config proxy-image=$(img istio/proxyv2)
   +juju deploy --trust --debug ./$(charm istio-gateway) istio-ingressgateway --config kind=ingress --config proxy-image=$(img istio/proxyv2) --config gateway_service_type="NodePort"

---------------------
Example
---------------------

Every setup may be different depending on the K8s choice (Charmed K8s, EKS, GKE, AKS, MicroK8s, etc.), cloud provider (GCP, AWS, Azure etc.) and container registry (Docker, Artifactory etc.).

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Air-gapped environment setup
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In this example, the air-gapped setup is as follows:

* MicroK8s runs inside a single node VM.
* The VM has cut-off internet connection (default Gateway has been removed).
* The Docker daemon is running on the VM, alongside MicroK8s, and the Docker CLI is available to those logged into the VM.
* A Docker registry is deployed as a container inside that VM (not inside MicroK8s cluster). See `Deploying a Registry Server <https://docs.docker.com/registry/deploying/>`_ for more details.
* The Docker registry has HTTPS enabled, using a TLS cert that we created, with domain ``air-gapped.registry.com``.
* The VM has been configured to trust our TLS cert for HTTPS traffic and recognise the domain name for our registry.
* The MicroK8s cluster can reach the Docker registry container via its domain name ``air-gapped.registry.com``, to fetch images.

~~~~~~~~~~~~~~~~~~~~~~~
Extract and load images
~~~~~~~~~~~~~~~~~~~~~~~

It is up to you how to extract and load the images provided to them in ``images.tar.gz``. 
This example just focuses on how the process might look for one image. 
Within the overall tarball, there will be a sub-tarball per image. 
For example, the tarball ``jupyter-web-app.tar`` will contain the ``jupyter-web-app`` image.

The extraction process might look like this:

1. The main archive is extracted to retrieve all the sub-tarballs: ``tar -xzvf images.tar.gz``. Inside this extracted archive will be ``jupyter-web-app.tar``.
2. ``docker load < jupyter-web-app.tar`` - this will pull the image from the tarball into Docker.
3. The image pulled will have the default name assigned to it in production: ``docker.io/kubeflownotebookswg/jupyter-web-app:v1.8.0``. Note that this image name implies that it lives in the ``docker.io`` public registry.
4. A new name is given to the image to specify its new home in our air-gapped registry: ``docker tag docker.io/kubeflownotebookswg/jupyter-web-app:v1.8.0 air-gapped.registry.com/kubeflownotebookswg/jupyter-web-app:v1.8.0``. 

.. note::
   At this point there should be two names for the same image, in the Docker cache, as can be seen with ``docker image ls``.

5. The image is pushed to the air-gapped registry with ``docker push air-gapped.registry.com/kubeflownotebookswg/jupyter-web-app:v1.8.0``.

A similar process would then be followed for all images. 
The new names of the images, as they appear in the air-gapped registry, should be noted, as they will be needed in the bundle configuration step.
