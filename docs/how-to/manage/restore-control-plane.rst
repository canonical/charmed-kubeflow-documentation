.. _restore_control_plane:

Restore control plane
=====================

This guide describes how to restore the Charmed Kubeflow (CKF) control plane from a compatible S3 storage.

.. warning::
   It is expected that these steps are followed all at once, backing up all databases, pipelines MinIO bucket, and ML Metadata database at the same time. Failing to do so may result in data loss.

.. note::
   Running Kubeflow pipelines and Katib experiments can affect the outcome of the backup, please make sure all pipelines and experiments are stopped and no other processes are calling them, such as Jupyter Notebooks.

.. note::
   User workloads in user namespaces are not backed up.

---------------------
Requirements
---------------------

- Access to a S3 storage used for the backup data - only AWS S3 and S3 RadosGW are supported.
- Admin access to the Kubernetes cluster where CKF is deployed.
- Juju admin access to the ``kubeflow`` model.
- `yq <https://snapcraft.io/yq>`_ binary.
- Ensure the local storage is big enough to back up the data.

---------------------
Configure ``rclone``
---------------------

``rclone`` is a tool that allows file management in cloud storage. 
This tool will be used for backing up several files throughout this guide and it can be installed as a `snap`_:

.. code-block:: bash

   sudo snap install rclone

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Connect to a shared S3 storage
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. `Configure <https://rclone.org/commands/rclone_config/>`_ ``rclone`` to connect to the shared S3 storage. The following can be used as reference:

.. code-block::

   [remote-s3]
   type = s3
   provider = AWS
   env_auth = true
   access_key_id = ...
   secret_access_key = ...
   region = eu-central-1
   acl = private
   server_side_encryption = AES256

.. note::
   You can check where this configuration file is located with ``rclone config file``.

2. Save the name of the S3 remote in an ``ENV`` variable:

.. code-block:: bash

   RCLONE_S3_REMOTE=remote-s3

~~~~~~~~~~~~~~~~~~~~~
Connect to CKF MinIO
~~~~~~~~~~~~~~~~~~~~~

1. The following steps require an accessible MinIO endpoint, which can be done port forwarding the ``minio`` service:

.. code-block:: bash

   kubectl port-forward -n kubeflow svc/minio 9000:9000

2. Get ``minio``'s ``secret-key`` value:

.. code-block:: bash

   juju show-unit kfp-ui/0 \
       | yq '.kfp-ui/0.relation-info.[] | select (.endpoint == "object-storage") | .application-data.data' \
       | yq '.secret-key'

3. Get ``minio``'s ``access-key``:

.. code-block:: bash

   juju config minio access-key

4. `Configure <https://rclone.org/commands/rclone_config/>`_ ``rclone`` to connect to CKF MinIO. The following can be used as reference:

.. code-block::

   [minio-ckf]
   type = s3
   provider = Minio
   access_key_id = minio
   secret_access_key = ...
   endpoint = http://localhost:9000
   acl = private

5. Save the name of the MinIO remote in an ``ENV`` variable:

.. code-block:: bash

   RCLONE_MINIO_REMOTE=minio-ckf

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

.. _restore_mlmd_sqlite:

-------------------------------------
Restore ML metadata using ``sqlite3``
-------------------------------------

The ``mlmd`` charm uses a SQLite database to store ML metadata generated from Kubeflow pipelines.

1. Install the required tools inside the application container:

.. note::
   This step expects the ``mlmd`` application container to have Internet access.

.. code-block:: bash

   # MLMD > 1.14, CKF 1.9
   MLMD_POD="mlmd-0"
   MLMD_CONTAINER="mlmd-grpc-server"

   # MLMD 1.14, CKF 1.8
   MLMD_POD="mlmd-0"
   MLMD_CONTAINER="mlmd"

   kubectl exec -n kubeflow $MLMD_POD -c $MLMD_CONTAINER -- \
       /bin/bash -c "apt update && apt install sqlite3 -y"

2. Scale down ``kfp-metadata-writer``:

.. code-block:: bash

   juju scale-application kfp-metadata-writer 0

3. Copy the backup file from the shared S3 storage to a local storage:

.. code-block:: bash

   S3_BUCKET=backup-bucket-2024
   RCLONE_S3_REMOTE=remote-s3
   MLMD_BACKUP=<backup-file-name-in-s3-storage>

   rclone --size-only copy \
       $RCLONE_S3_REMOTE:$S3_BUCKET/$MLMD_BACKUP .

4. Restore data from a backup file:

Copy the local database file into the application container:

.. code-block:: bash

   kubectl cp -n kubeflow -c $MLMD_CONTAINER \
       $MLMD_BACKUP \
       $MLMD_POD:/tmp/$MLMD_BACKUP

Move the current database file to a temporary directory:

.. code-block:: bash

   kubectl exec -n kubeflow $MLMD_POD -c $MLMD_CONTAINER -- \
       /bin/bash -c "mv /data/mlmd.db /tmp/mlmd.current"

Restore the database from the backup file:

.. code-block:: bash

   kubectl exec -n kubeflow $MLMD_POD -c $MLMD_CONTAINER -- \
       /bin/bash -c "zcat /tmp/$MLMD_BACKUP | sqlite3 /data/mlmd.db"

5. Optionally remove the local backup file:

.. code-block:: bash

   rm -rf $MLMD_BACKUP

6. Scale up ``kfp-metadata-writer``:

.. code-block:: bash

   juju scale-application kfp-metadata-writer 1

-----------------------------------
Restore ``mlpipeline`` MinIO bucket
-----------------------------------

Sync all files from the shared S3 storage to ``minio``:

.. code-block:: bash

   S3_BUCKET=backup-bucket-2024
   RCLONE_S3_REMOTE=remote-s3
   RCLONE_BWIDTH_LIMIT=20M

   rclone --size-only sync \
       --bwlimit $RCLONE_BWIDTH_LIMIT \
       $RCLONE_S3_REMOTE:$S3_BUCKET/mlpipeline \
       $RCLONE_MINIO_REMOTE:mlpipeline

-------------------------------------
Restore ML metadata using ``kubectl``
-------------------------------------

You can also restore ML metadata using ``kubectl``.

1. Scale down ``kfp-metadata-writer``:

.. code-block:: bash

   juju scale-application kfp-metadata-writer 0

2. Copy the backup file from the shared S3 storage to a local storage:

.. code-block:: bash

   S3_BUCKET=backup-bucket-2024
   RCLONE_S3_REMOTE=remote-s3
   MLMD_BACKUP=<backup-file-name-in-s3-storage>

   rclone --size-only copy \
       $RCLONE_S3_REMOTE:$S3_BUCKET/$MLMD_BACKUP .

3. Restore data from a backup file:

Copy the local database file into the application container:

.. code-block:: bash

   kubectl cp -n kubeflow -c $MLMD_CONTAINER \
       $MLMD_BACKUP \
       $MLMD_POD:/tmp/$MLMD_BACKUP

Move the current database file to a temporary directory:

.. code-block:: bash

   kubectl exec -n kubeflow $MLMD_POD -c $MLMD_CONTAINER -- \
       /bin/bash -c "mv /data/mlmd.db /tmp/mlmd.current"

Place the backup file into the ``data`` path:

.. code-block:: bash

   kubectl exec -n kubeflow $MLMD_POD -c $MLMD_CONTAINER -- \
       /bin/bash -c "mv /tmp/$MLMD_BACKUP /data/mlmd.db"

4. Optionally remove the local backup file:

.. code-block:: bash

   rm -rf $MLMD_BACKUP

5. Scale up ``kfp-metadata-writer``:

.. code-block:: bash

   juju scale-application kfp-metadata-writer 1
