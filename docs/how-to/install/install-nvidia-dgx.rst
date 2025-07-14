.. _install_nvidia_dgx:

Install on NVIDIA DGX
=====================

This guide describes how to install Charmed Kubeflow (CKF) on `NVIDIA DGX <https://www.nvidia.com/en-us/data-center/dgx-systems/>`_ hardware. 
DGX systems are purpose-built hardware for enterprise AI use cases, featuring NVIDIA Tensor Core GPUs.

---------------------
Requirements
---------------------

- NVIDIA DGX-enabled hardware setup, including no NVIDIA drivers preinstalled, BIOS settings and bootloader.
- `kubectl <https://kubernetes.io/docs/reference/kubectl/>`_.

---------------------
Install MicroK8s
---------------------

Install `MicroK8s <https://microk8s.io/>`_ and enable required add-ons as follows:

.. code-block:: bash

   sudo snap install microk8s --classic --channel 1.22

   sudo microk8s enable dns:10.229.32.21 storage ingress registry rbac helm3 metallb:10.64.140.43-10.64.140.49,192.168.0.105-192.168.0.111

   sudo usermod -a -G microk8s ubuntu
   sudo chown -f -R ubuntu ~/.kube
   newgrp microk8s

Edit ``/var/snap/microk8s/current/args/containerd-template.toml`` by adding:

.. code-block:: bash

   [plugins."io.containerd.grpc.v1.cri".registry.configs]

   [plugins."io.containerd.grpc.v1.cri".registry.configs."registry-1.docker.io".auth]
   username = "afrikha"
   password = "<>"

Finally, restart MicroK8s:

.. code-block:: bash

   microk8s.stop
   microk8s.start

---------------------
Enable GPU add-on
---------------------

Install the required GPU operator as follows:

.. code-block:: bash

   sudo microk8s.enable gpu
   mkdir .kube
   microk8s config > ~/.kube/config

Check the GPU count for MicroK8s:

.. code-block:: bash

   kubectl get nodes --show-labels | grep gpu.count

---------------------
Configure MIG
---------------------

Configure MIG devices running the following command:

.. code-block:: bash

   kubectl label nodes blanka nvidia.com/mig.config=all-1g.5gb --overwrite

Check again the GPU count to confirm it has increased:

.. code-block:: bash

   kubectl get nodes --show-labels | grep gpu.count

.. note::

   If no nodes appear in the command output above, uninstall all GPU drivers from K8s nodes and reinstall MicroK8s.

---------------------
Deploy CKF
---------------------

Follow the instructions in :ref:`General installation <general_installation>` for this section.

---------------------
Explore some examples
---------------------

CKF can be run on different types of DGX hardware:

- See `kubeflow-single-node-dgx <https://github.com/canonical/kubeflow-single-node-dgx>`_ for single-node examples.
- See `kubeflow-multi-node-dgx <https://github.com/canonical/kubeflow-multi-node-dgx>`_ for multi-node examples.
