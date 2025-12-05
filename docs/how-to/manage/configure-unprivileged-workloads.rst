.. _configure_unprivileged_workloads:

Configure user workloads to run unprivileged
============================================

This guide describes how to configure Charmed Kubeflow (CKF) to run all user-generated workloads without elevated permissions. This is accomplished by configuring the `Pod Security Standards <https://kubernetes.io/docs/concepts/security/pod-security-standards/)>`_ for different levels of privileges for workloads running on namespaces associated with Kubeflow Profiles.

---------------------
Requirements
---------------------

* A Charmed Kubeflow deployment with version `1.10` or later.
* Revision `765` or later of the `kubeflow-profiles` charm. You can view the currently used revision with `juju status kubeflow-profiles --format=json | jq '.applications["kubeflow-profiles"]["charm-rev"]'`, which uses the `jq` package.
* The Istio CNI plugin has been enabled. See :ref:`enable Istio CNI plugin<enable_istio_cni>` for more details.


-----------------------------------------------------
Configure Pod Security Standards in Kubeflow Profiles
-----------------------------------------------------

CKF security policies are based on the `Kubernetes Pod Security Standards <https://kubernetes.io/docs/concepts/security/pod-security-standards/>`_, which use control plane mechanisms to enforce security settings. You can view the currently used security policy in your deployment by running:

.. code-block:: bash

    juju config kubeflow-profiles security-policy

.. note::

   The `restricted` policy is currently not supported in Charmed Kubeflow.

~~~~~~~~~~~~~~~~~~~~~~~~~~~
Configure `baseline` policy
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Configure the security policy in `kubeflow-profiles` to `baseline` by running:

.. code-block:: bash

    juju config kubeflow-profiles security-policy=baseline

The deployment should now be configured to enforce the `baseline` pod security standards policy in all user-generated workloads. To learn more about the allowed permissions in the `baseline` policy, refer to the `Kubernetes documentation <https://kubernetes.io/docs/concepts/security/pod-security-standards/#baseline>`_.


~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Configure `privileged` policy
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you don't want to enforce a more restrictive policy, you can set the security policy in `kubeflow-profiles` to `privileged`:

.. code-block:: bash

    juju config kubeflow-profiles security-policy=privileged
