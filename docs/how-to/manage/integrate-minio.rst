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

   juju refresh -kubeflow

Now you should be able to access MinIO.

You can check if it is added to the model as follows:

.. code-block:: bash

   k8s kubectl get PodDefault -n admin

You should see ``allow-minio``.

Another option is to access the MinIO dashboard. 
You can do so by running ``juju status`` and accessing the provided MinIO IP address.
To login, use the credentials you previously set up.

------------------------
Configure Operation Mode
------------------------

~~~~~~~~~~~
Server Mode
~~~~~~~~~~~

By default, MinIO is run in server mode, where it directly provides the object storage backend.

To verify so, run:

.. code-block:: bash

   juju config minio mode

If the output differs from ``server`` and you want to restore server mode, run:

.. code-block:: bash

   juju config minio mode=server

This is the recommended setup and no further configuration changes are required.

~~~~~~~~~~~~
Gateway Mode
~~~~~~~~~~~~

Nevertheless, running MinIO in gateway mode, despite being deprecated, is still a popular request to run it as a stateless proxy to add an S3-compatible API around an actual storage backend that would not otherwise support it.

To configure MinIO to run in gateway mode, run:

.. code-block:: bash

   juju config minio gateway-storage-service=<your-storage-service-type>
   juju config minio mode=gateway

where ``<your-storage-service-type>`` can be either ``s3`` or ``azure``.

Additionally, you may need to run:

.. code-block:: bash

   juju config minio storage-service-endpoint=<your-storage-service-endpoint>

where ``<your-storage-service-endpoint>`` represents the endpoint of your storage service. This is only necessary for some endpoints and is specific to the storage service provider.

.. note::

   With S3 storage by AWS, this URI should be in the form ``https://s3.<your-region>.amazonaws.com``, e.g.: ``https://s3.eu-west-1.amazonaws.com``.

.. note::

   With S3 storage by AWS, avoid prefixing the service endpoint with the bucket name, e.g.: not ``https://<your-bucket-name>.s3.eu-west-1.amazonaws.com`` but ``https://s3.eu-west-1.amazonaws.com``. Find :ref:`here <configure_custom_bucket_for_pipelines>` details for configuring custom buckets.
