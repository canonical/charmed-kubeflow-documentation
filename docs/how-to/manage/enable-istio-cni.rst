.. _enable_istio_cni:

Enable Istio CNI plugin
=======================

This guide describes how to enable `Istio CNI plugin <https://istio.io/latest/docs/setup/additional-setup/cni/>`_ on Charmed Kubeflow (CKF).

By default, `Istio <https://charmhub.io/istio>`_ injects an init container in all Pods inside the service mesh, 
which configures each ``Pod``'s network traffic redirection to and from the Istio sidecar proxy. 
This operation requires elevated permissions: Kubernetes (K8s) RBAC permissions to deploy containers with the ``NET_ADMIN`` and ``NET_RAW`` capabilities, 
which can conflict with some organisations' security policies.

The Istio CNI plugin is a replacement of that init container that resolves the security concerns by avoiding the need for elevated permissions while providing the same functionality.

---------------------
Requirements
---------------------

* A running Istio control plane deployed by ``istio-operators > 1.17/*``.

.. note::

   The Istio CNI plugin is only available in > ``1.17/*``. See `Upgrading istio-pilot <https://github.com/canonical/istio-operators/tree/main/charms/istio-pilot#upgrading-istio-pilot>`_ for more details.

---------------------
Enable the plugin
---------------------

You can enable the Istio CNI plugin by setting the following configuration values for the ``istio-pilot`` charm:

* ``cni-bin-dir``: the path where the CNI binaries, which implement the CNI specification, are located in the host system where the Kubernetes control plane is deployed.
* ``cni-conf-dir``: the path where the CNI's conflist files (in JSON format) are located in the host system where the Kubernetes control plane is deployed.

Set both of these to enable the plugin:

.. code-block:: bash

   juju config istio-pilot cni-bin-dir=<path to cni bin dir in host>

.. code-block:: bash

   juju config istio-pilot cni-conf-dir=<path to cni conf dir in host>

.. warning::
    Before configuring these options, make sure the paths are correct and exist. Otherwise the Istio control plane installation could fail.

These values vary on each K8s installation and depend on the CNI's configuration. 
The defaults for some installations are ``/opt/cni/bin`` and ``/etc/cni/net.d`` respectively.

For example, in ``microk8s``, these values are ``/var/snap/microk8s/current/opt/cni/bin`` and ``/var/snap/microk8s/current/args/cni-network``. 
Refer to `MicroK8s CNI configuration <https://microk8s.io/docs/change-cidr>`_ for more details.

.. note::

   Enabling the Istio CNI plugin only affects ``Pods`` that are created in a namespace with automatic sidecar injection after the plugin has been installed in the control plane.

See `Network Plugins <https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/>`_ and `CNI <https://github.com/containernetworking/cni/tree/main#cni---the-container-network-interface>`_ for further information.
