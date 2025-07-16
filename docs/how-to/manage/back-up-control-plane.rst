.. _back_up:

Back up control plane
=====================

This guide describes how to back up the Charmed Kubeflow (CKF) control plane data to a compatible S3 storage.

.. warning::
   It is expected that these steps are followed all at once, backing up all databases, pipelines MinIO bucket, and ML Metadata database at the same time. Failing to do so may result in data loss.

.. note::
   Running Kubeflow pipelines and Katib experiments can affect the outcome of the backup, please make sure all pipelines and experiments are stopped and no other processes are calling them, such as Jupyter Notebooks.

.. note::
   User workloads in user namespaces are not backed up.

---------------------
Requirements
---------------------

- Access to an S3 compatible storage, such as RadosGW, AWS S3, or MinIO, for the backup data.
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

-------------------------------------
Back up ML metadata using ``sqlite3``
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

2. Scale down ``kfp-metadata-writer``.

.. code-block:: bash

   juju scale-application kfp-metadata-writer 0

3. Perform a database backup:

.. code-block:: bash

   MLMD_BACKUP=mlmd-$(date -d "today" +"%Y-%m-%d-%H-%M").dump.gz

   kubectl exec -n kubeflow $MLMD_POD -c $MLMD_CONTAINER -- \
       /bin/bash -c \
       "sqlite3 /data/mlmd.db .dump | gzip -c >/tmp/$MLMD_BACKUP"

4. Copy the backup file to local storage:

.. code-block:: bash

   kubectl cp -n kubeflow -c $MLMD_CONTAINER \
       $MLMD_POD:/tmp/$MLMD_BACKUP \
       ./$MLMD_BACKUP

5. Copy the ``mlmd`` backup data to the S3 storage:

.. code-block:: bash

   S3_BUCKET=backup-bucket-2024
   RCLONE_S3_REMOTE=remote-s3
   RCLONE_BWIDTH_LIMIT=20M

   rclone --size-only copy \
       --bwlimit $RCLONE_BWIDTH_LIMIT \
       ./$MLMD_BACKUP \
       $RCLONE_S3_REMOTE:$S3_BUCKET

Optionally, you can remove the ``mlmd`` data from your local machine:

.. code-block:: bash

   rm -rf $MLMD_BACKUP

6. Scale up ``kfp-metadata-writer``:

.. code-block:: bash

   juju scale-application kfp-metadata-writer 1

------------------------------------
Back up ``mlpipeline`` MinIO bucket
------------------------------------

Sync all files from ``minio`` to the shared S3 storage:

.. code-block:: bash

   S3_BUCKET=backup-bucket-2024
   RCLONE_S3_REMOTE=remote-s3
   RCLONE_BWIDTH_LIMIT=20M

   rclone --size-only sync \
       --bwlimit $RCLONE_BWIDTH_LIMIT \
       $RCLONE_MINIO_REMOTE:mlpipeline \
       $RCLONE_S3_REMOTE:$S3_BUCKET/mlpipeline

------------------------------------
Back up ML metadata with ``kubectl``
------------------------------------

You can also perform the backup using ``kubectl``.

1. Scale down ``kfp-metadata-writer``:

.. code-block:: bash

   juju scale-application kfp-metadata-writer 0

2. Copy the backup file to local storage:

.. code-block:: bash

   # MLMD > 1.14, CKF 1.9
   MLMD_POD="mlmd-0"
   MLMD_CONTAINER="mlmd-grpc-server"

   # MLMD 1.14, CKF 1.8
   MLMD_POD="mlmd-0"
   MLMD_CONTAINER="mlmd"

   kubectl cp -n kubeflow -c $MLMD_CONTAINER \
       $MLMD_POD:/data/mlmd.db \
       ./$MLMD_BACKUP

3. Copy the ``mlmd`` backup data to the S3 storage:

.. code-block:: bash

   S3_BUCKET=backup-bucket-2024
   RCLONE_S3_REMOTE=remote-s3
   RCLONE_BWIDTH_LIMIT=20M

   rclone --size-only copy \
       --bwlimit $RCLONE_BWIDTH_LIMIT \
       ./$MLMD_BACKUP \
       $RCLONE_S3_REMOTE:$S3_BUCKET

Optionally, you can remove the ``mlmd`` backup data from your local machine:

.. code-block:: bash

   rm -rf $MLMD_BACKUP

4. Scale up ``kfp-metadata-writer``:

.. code-block:: bash

   juju scale-application kfp-metadata-writer 1
