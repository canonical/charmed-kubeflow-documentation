.. _restore:

Restore
=====================

This guide describes how to restore Charmed Kubeflow (CKF) control plane data and user workloads from the object storage using the `Charmed Velero <https://charmhub.io/velero-operator>`_.

.. warning::
   These steps are expected to be followed simultaneously, restoring all databases, pipelines, the MinIO bucket, and the ML Metadata database. Failing to do so may result in data loss.

.. note::
   Running Kubeflow pipelines and Katib experiments can affect the restoration outcome. Please make sure all pipelines and experiments are stopped and no other processes, such as Jupyter Notebooks, are calling them.

.. note::
   Full backups with volume snapshotting are only supported on AWS and Azure Kubernetes Clusters. You might opt for the File System Backup for Persistent Volume Claims, but depending on the Storage Class, it might not work.

---------------------
Requirements
---------------------

- Admin access to the Kubernetes cluster where CKF is deployed.
- Juju admin access to the ``kubeflow`` model.
- Charmed Velero was deployed and configured with the object storage where the backups were stored.

---------------------
Check backups
---------------------

In the Charmed Velero model, run the ``list-backups`` action to get the backups:

.. code-block:: bash

    juju run velero-operator/0 list-backups

The action returns a YAML list of backups. Please note the application name, endpoint, and backup UID. You will use the UID to make a restore.
 
----------------------------
Prepare for restore
----------------------------

Prepare CKF for the restore. The ID of a storage for each application can be retrieved by running ``juju storage``:

.. code-block:: bash

    juju scale-application minio 0
    juju scale-application mlmd 0
    juju remove-storage mlmd-data/<id>
    juju remove-storage minio-data/<id>

.. warning::
    Removing the storage from the ``minio`` and ``mlmd`` charms may result in data loss. Make sure nothing important is stored in the databases. This can be safely done on the clean `CKF install <https://documentation.ubuntu.com/charmed-kubeflow/tutorial/get-started/>`_.

----------------------
Make a restore
----------------------

Switch to the Charmed Velero model and initiate restores using the backup UIDs from the previous steps:

.. code-block::bash

    juju run velero-operator/0 restore backup-uid=<minio>
    juju run velero-operator/0 restore backup-uid=<mlmd>
    juju run velero-operator/0 restore backup-uid=<profiles>
    juju run velero-operator/0 restore backup-uid=<user-workloads>

.. note::
    Make sure the following order of execution is preserved.

After each restore, you need to re-attach the recreated storage for ``minio`` and ``mlmd`` charms by getting the names of the restored PVs using ``kubectl get pv``:

.. code-block:: bash

    kubectl get pv
    juju import-filesystem kubernetes <pv_name> minio-data --force
    juju import-filesystem kubernetes <pv_name> mlmd-data --force
    juju add-unit minio --attach-storage minio-data/<id>
    juju add-unit mlmd --attach-storage mlmd-data/<id>

The restore is now complete. Open the dashboard to see the backed-up data.

Please refer to the `Charmed Velero documentation <https://charmhub.io/velero-operator>` for more details.

-----------------------------------
Restore CKF databases to S3 storage
-----------------------------------

CKF uses ``katib-db`` and ``kfp-db`` as databases for Katib and Kubeflow pipelines respectively.

1. Deploy and configure the `s3-integrator <https://charmhub.io/s3-integrator>`_ to connect to the shared S3 storage.

Follow the `S3 AWS <https://charmhub.io/mysql-k8s/docs/h-configure-s3-aws>`_ and `S3 Radowsg <https://charmhub.io/mysql-k8s/docs/h-configure-s3-radosgw>`_ configuration guides for this step.

2. Scale up ``kfp-db`` and ``katib-db``.

This step avoids the ``Primary`` database from becoming unavailable during backup:

.. code-block:: bash

   juju scale-application kfp-db 2
   juju scale-application katib-db 2

3. `Restore <https://charmhub.io/mysql-k8s/docs/h-restore-backup>`_ ``kfp-db`` and ``katib-db``.

Replace ``mysql-k8s`` with the name of the database you intend to restore, e.g., ``katib-db`` instead of ``mysql-k8s``.
