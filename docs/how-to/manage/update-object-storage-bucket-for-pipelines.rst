.. _update_object_storage_bucket_for_pipelines:

Update Object Storage Bucket for Pipelines
=======================

This guide describes how to update the object storage bucket employed by Pipelines.

---------------------
Requirements
---------------------

- An object storage bucket, for Pipelines to store its artifacts, to replace the default/current bucket with.

---------------------
Update the Object Storage Bucket Employed by Pipelines
---------------------

The following charms need to have their respective bucket configurations updated:

- Argo Controller: the name of the bucket

    .. code-block:: bash

        juju config argo-controller bucket=your-bucket-name

- KFP API: the name of the bucket

    .. code-block:: bash

        juju config kfp-api object-store-bucket-name=your-bucket-name

- KFP Profile Controller:

    the `default pipeline root <https://www.kubeflow.org/docs/components/pipelines/concepts/pipeline-root>`_, which includes the protocol for, the name of and the path to the bucket

    .. code-block:: bash

        juju config kfp-profile-controller default_pipeline_root=minio://your-bucket-name/your/bucket/path
