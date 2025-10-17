.. _back_up:

Back Up
=====================

This guide describes how to back up Charmed Kubeflow (CKF) control plane data and user workloads to object storage using the `Charmed Velero <https://charmhub.io/velero-operator>`_.

.. warning::
   These steps are expected to be followed all at once, backing up all databases, pipelines, the MinIO bucket, and the ML Metadata database simultaneously. Failing to do so may result in data loss.

.. note::
   Running Kubeflow pipelines and Katib experiments can affect the backup outcome. Please make sure all pipelines and experiments are stopped and no other processes, such as Jupyter Notebooks, are calling them.

.. note::
   Full backups with volume snapshotting are only supported on AWS and Azure Kubernetes clusters. You might opt for the File System Backup for Persistent Volume Claims, but depending on the Storage Class, it might not work. For example, MicroK8s deployments will not support full backups.

---------------------
Requirements
---------------------

- Admin access to the Kubernetes (K8s) cluster where CKF is deployed.
- Juju admin access to the ``kubeflow`` model.
- Ensure the object storage is big enough to back up the data.

---------------------------------------------------
Deploy and configure Charmed Velero
---------------------------------------------------

Follow one of the `How-To <https://charmhub.io/velero-operator/docs/how-to>`_ for your cloud provider to deploy and configure Charmed Velero. After Charmed Velero is ready, offer the relation for backup from the Velero model:

.. code-block:: bash

    juju offer velero-operator:velero-backups velero-backups

-----------------------------------
Make a backup
-----------------------------------

Switch to the Kubeflow model and consume the offered backup relation:

.. code-block:: bash

    juju switch kubeflow
    juju consume velero.velero-backups

Integrate each relevant Kubeflow application with Charmed Velero:

.. code-block:: bash

    juju integrate minio velero-backups
    juju integrate mlmd velero-backups
    juju integrate kubeflow-profiles:profiles-backup-config velero-backups
    juju integrate kubeflow-profiles:user-workloads-backup-config velero-backups

Switch back to the Velero model and run the following commands to make a backup for each application:

.. code-block:: bash

    juju switch velero

    juju run velero-operator/0 create-backup \
    target=minio:velero-backup-config \
    model=kubeflow
    juju run velero-operator/0 create-backup \
    target=mlmd:velero-backup-config \
    model=kubeflow
    juju run velero-operator/0 create-backup \
    target=kubeflow-profiles:profiles-backup-config \
    model=kubeflow
    juju run velero-operator/0 create-backup \
    target=kubeflow-profiles:user-workloads-backup-config \
    model=kubeflow

-----------------------------------
List backups
-----------------------------------

You can check the created backups by running the following action in the Velero model:

.. code-block:: bash

    juju run velero-operator/0 list-backups

Please refer to the `Charmed Velero documentation <https://charmhub.io/velero-operator/docs>`_ for more details.

-----------------------------------
Back up CKF databases to S3 storage
-----------------------------------

CKF uses ``katib-db`` and ``kfp-db`` as databases for Katib and Kubeflow pipelines respectively.

1. Deploy and configure the `s3-integrator <https://charmhub.io/s3-integrator>`_ to connect to the shared S3 storage.

See `S3 AWS <https://charmhub.io/mysql-k8s/docs/h-configure-s3-aws>`_ and `S3 Radowsg <https://charmhub.io/mysql-k8s/docs/h-configure-s3-radosgw>`_ configuration guides for this step.

2. Scale up ``kfp-db`` and ``katib-db``.

This step avoids the ``Primary`` database from becoming unavailable during backup.

.. code-block:: bash

   juju scale-application kfp-db 2
   juju scale-application katib-db 2

3. `Create a backup <https://charmhub.io/mysql-k8s/docs/h-create-backup>`_ for each database.

Replace ``mysql-k8s`` with the name of the database you intend to create a backup for in the commands from that guide.
