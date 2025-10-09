.. _configure_custom_bucket_for_pipelines:

Configure a Custom Bucket for Pipelines
==============================================

This guide describes how to update the object storage bucket employed by Pipelines.

------------
Requirements
------------

- A CKF deployment. See :ref:`Get started <get_started>` for more details.

- `Juju`_ version >= ``3.4``.

- Juju admin access to the ``kubeflow`` model.

- An object storage bucket, for Pipelines to store its artifacts, to replace the default/current bucket with.

------------------------------------------------
Configure Pipelines' Charms with a Custom Bucket
------------------------------------------------

After defining environment variables for your bucket name and path, for your convenience:

.. code-block:: bash

  BUCKET_NAME="your-bucket-name"
  BUCKET_PATH="your/bucket/path"

The following charms need to have their respective bucket configurations updated:

~~~~~~~~~~~~~~~
Argo Controller
~~~~~~~~~~~~~~~

Update the name of the bucket via its ``bucket`` `configuration option <https://charmhub.io/argo-controller/configurations>`__, e.g.:

.. code-block:: bash

  juju config argo-controller bucket=${BUCKET_NAME}

~~~~~~~~~
KFP API
~~~~~~~~~

Update the name of the bucket via its ``object-store-bucket-name`` `configuration option <https://charmhub.io/kfp-api/configurations>`__, e.g.:

.. code-block:: bash

  juju config kfp-api object-store-bucket-name=${BUCKET_NAME}

~~~~~~~~~~~~~~~~~~~~~~~~
KFP Profile Controller
~~~~~~~~~~~~~~~~~~~~~~~~

Update the `default pipeline root <https://www.kubeflow.org/docs/components/pipelines/concepts/pipeline-root>`__ for pipeline Runs across all Profiles, which includes the schema for, the name of and the path to the bucket, via its ``default_pipeline_root`` `configuration option <https://charmhub.io/kfp-profile-controller/configurations>`__, e.g.:

.. code-block:: bash

  juju config kfp-profile-controller default_pipeline_root=minio://${BUCKET_NAME}/${BUCKET_PATH}

.. note::

  In Charmed Kubeflow, Pipelines can only be configured with the MinIO schema.

This automatically creates (or updates) a ``ConfigMaps`` named ``kfp-launcher`` in each Profile

.. note::

  For this configuration change to take effect, existing ``ConfigMaps`` — if any — named ``kfp-launcher`` in the namespace of each Profile of interest have to be manually deleted after the configuration change is applied, so that they are automatically recreated with the updated default pipeline root. Refer to `this issue <https://github.com/canonical/metacontroller-operator/issues/193>`__ for details.
