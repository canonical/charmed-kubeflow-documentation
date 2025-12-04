.. _configure_unprivileged_workloads:

Configure user workloads to run unprivileged
============================================

This guide describes how to configure Charmed Kubeflow (CKF) to run all user-generated workloads without elevated permissions. This is accomplished by configuring the [Pod Security Standards](https://kubernetes.io/docs/concepts/security/pod-security-standards/) for different levels of privileges for workloads running on namespaces associated with Kubeflow Profiles.

---------------------
Requirements
---------------------

* A Charmed Kubeflow deployment with version 1.10 or later.

-----------------------
Enable Istio CNI Plugin
-----------------------

First, follow :ref:`enable Istio CNI<enable_istio_cni>` to enable the Istio CNI plugin in your CKF deployment. This will ensure that all Istio init containers follow the stricter security standards that are enforced later in this guide.


-------------------------------------------
Configure Kubeflow Profiles security policy
-------------------------------------------

CKF security policies are based on the `Pod Security Standards <https://kubernetes.io/docs/concepts/security/pod-security-standards/#baseline>`_. Configure the security policy in `kubeflow-profiles` to `baseline` by running:

.. code-block:: bash

    juju config kubeflow-profiles security-policy=baseline

The deployment should now be configured to enforce the `baseline` pod security standards policy in all user-generated workloads. To learn more about the allowed permissions in the `baseline` policy, refer to the `Kubernetes documentation <https://kubernetes.io/docs/concepts/security/pod-security-standards/#baseline>`_.


-------------------------------------------
Revert changes
-------------------------------------------

If you don't want to enforce a more restricted policy, you can set the security policy in `kubeflow-profiles` back to `privileged`:

.. code-block:: bash

    juju config kubeflow-profiles security-policy=privileged
