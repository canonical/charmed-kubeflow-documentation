.. _update_bucket_for_pipelines:

Update the Object Storage Bucket for Pipelines
==========================================

This guide describes how to update the object storage bucket employed by Pipelines.

------------
Requirements
------------

- An object storage bucket, for Pipelines to store its artifacts, to replace the default/current bucket with.

------------------------------------------------------
Update the Object Storage Bucket Employed by Pipelines
------------------------------------------------------

The following charms need to have their respective bucket configurations updated:

- Argo Controller:

  the name of the bucket via ``bucket``, e.g.:

  .. code-block:: bash

    juju config argo-controller bucket=your-bucket-name

- KFP API:

  the name of the bucket via ``object-store-bucket-name``, e.g.:

  .. code-block:: bash

    juju config kfp-api object-store-bucket-name=your-bucket-name

- KFP Profile Controller:

  the `default pipeline root <https://www.kubeflow.org/docs/components/pipelines/concepts/pipeline-root>`_, which includes the schema for, the name of and the path to the bucket, via ``default_pipeline_root``, e.g.:

  .. code-block:: bash

    juju config kfp-profile-controller default_pipeline_root=your-protocol://your-bucket-name/your/bucket/path

  .. note::

    For this configuration change to take effect, existing ConfigMaps — if any — named ``kfp-launcher`` in the namespace of each Profile of interest have to be manually deleted after the configuration change is applied, so that they are automatically recreated with the updated default pipeline root. Refer to `this issue <https://github.com/canonical/metacontroller-operator/issues/193>`_ for details.
