.. _install_gke:

Install on GKE
==============

This guide describes how to install Charmed Kubeflow (CKF) on `Google Kubernetes Engine (GKE) <https://cloud.google.com/kubernetes-engine>`_. 

You will spin up a GKE cluster on Google Gloud, and then deploy CKF using the Kubernetes (K8s) Command Line Interface (CLI), `kubectl <https://kubernetes.io/docs/reference/kubectl/>`_ , and `Juju`_. 

---------------------
Requirements
---------------------

- Ubuntu 22.04 LTS or later.
- Google Cloud account.
- ``kubectl``. 
- `gcloud CLI <https://cloud.google.com/sdk/docs/install>`_.
- Juju.

---------------------
Set up Google Cloud
---------------------

1. Install Google Cloud authorisation plugin using gcloud CLI:

.. code-block:: bash

   gcloud components install gke-gcloud-auth-plugin

2. Log in and enable required services on your Google Cloud project:

.. code-block:: bash

   gcloud auth login
   export PROJECT_ID=test-project
   gcloud config set project test-project
   gcloud --project=${PROJECT_ID} services enable \
       container.googleapis.com \
       containerregistry.googleapis.com \
       binaryauthorization.googleapis.com

---------------------
Deploy GKE cluster
---------------------

1. You can create a GKE cluster by specifying the machine type and disk size as follows:

.. note::

   `CKF <https://charmed-kubeflow.io/docs/get-started-with-charmed-kubeflow>`_ suggests at least 4 cores, 32G RAM and 50G of disk for the cluster machines.
   Therefore, you can use the *n1-standard-16* machine type for your cluster. 
   The *n1-standard-8* might be enough for testing purposes.

.. code-block:: bash

   gcloud container clusters create --binauthz-evaluation-mode=PROJECT_SINGLETON_POLICY_ENFORCE --zone us-central1-a --machine-type n1-standard-16 --disk-size=100G test-cluster

2. After your cluster is created, save the credentials to be used with ``kubectl``:

.. code-block:: bash

   gcloud container clusters get-credentials --zone us-central1-a test-cluster
   kubectl config rename-context gke_test-project_us-central1-a_test-cluster gke-cluster

3. Bootstrap a Juju controller to the GKE cluster:

.. code-block:: bash

   /snap/juju/current/bin/juju bootstrap gke-cluster

.. warning:: 

   The command ``/snap/juju/current/bin/juju`` is currently used as a workaround for a `bug <https://bugs.launchpad.net/juju/+bug/2007575>`_.

---------------------
Deploy CKF
---------------------

To deploy CKF and access its dashboard, follow the steps provided in the :ref:`general installation <general_installation>` guide from creating the ``kubeflow`` model section.

---------------------
Clean up resources
---------------------

You can clean up allocated resources in Juju and Google Cloud as follows:

.. code-block:: bash

   juju destroy-model kubeflow --destroy-storage
   gcloud container clusters delete test-cluster --zone us-central1-a
