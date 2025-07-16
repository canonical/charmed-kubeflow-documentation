.. _integrate_minio:

Integrate with MinIO
====================

This guide describes object storage access in Charmed Kubeflow (CKF) through `MinIO <https://charmhub.io/minio>`_.

---------------------
Set keys
---------------------

First, check if the access key is configured:

.. code-block:: bash

   juju config minio access-key

Then, check if the secret key is configured:

.. code-block:: bash

   juju config minio secret-key

In case they are not set, create a new username and password:

.. code-block:: bash

   juju config minio access-key=<username>
   juju config minio secret-key=<password>

.. note::

   Both username and password need to be at least eight characters long.

.. _configure_access_minio:

---------------------
Configure access
---------------------

MinIO needs to be added to the pod and configured to be accessible. 
To do so, you have to create a YAML file. For example,

.. code-block:: bash

   touch allow-minio.yaml

After that, open the file and update it as follows:

.. code-block:: bash

   #allow-minio.yaml
   apiVersion: kubeflow.org/v1alpha1
   kind: PodDefault
   metadata:
     name: access-minio
   spec:
     desc: Allow access to Minio
     selector:
       matchLabels:
         access-minio: "true"
     env:
       - name: AWS_ACCESS_KEY_ID
         valueFrom:
           secretKeyRef:
             name: mlpipeline-minio-artifact
             key: accesskey
             optional: false
       - name: AWS_SECRET_ACCESS_KEY
         valueFrom:
           secretKeyRef:
             name: mlpipeline-minio-artifact
             key: secretkey
             optional: false
       - name: MINIO_ENDPOINT_URL
         value: http://minio.kubeflow.svc.cluster.local:9000

Once updated, run in a terminal the following command:

.. code-block:: bash

   kubectl apply -f allow-minio.yaml -n <user-namespace>

---------------------
Refresh Juju
---------------------

Refresh Juju to update previous changes:

.. code-block:: bash

   juju refresh –kubeflow

Now you should be able to access MinIO.

You can check if it is added to the model as follows:

.. code-block:: bash

   sudo microk8s kubectl get PodDefault -n admin

You should see ``allow-minio``.

Another option is to access the MinIO dashboard. 
You can do so by running ``juju status`` and accessing the provided MinIO IP address. 
To login, use the credentials you previously set up.
