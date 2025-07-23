.. _system_architecture_managed_kf:

System architecture 
===================

This guide provides a high-level overview of the Managed Kubeflow system architecture, 
with a particular focus on how access control and networking are configured during deployment.

Access control model
--------------------

When you deploy Managed Kubeflow from the Azure Marketplace, a new managed resource group is created within your Azure subscription. This group contains all the infrastructure components required to operate the platform and governs access as follows:

- Canonical access: The Canonical Managed Solutions team is granted full administrative access to manage and support the deployment.
- Customer access: You have read-only visibility into the managed resource group. You retain the ability to terminate the deployment at any time, ensuring full control over your cloud environment.
- Cost management: You can manage infrastructure sizing and cost by setting quotas and budgets on the Azure subscription used for the deployment.

Network architecture
--------------------

The deployment provisions a flat virtual network (VNet) that hosts two main subnets. 
See the following diagram for more details:

.. figure:: /_static/managed_kf_system_arch.jpg
   :align: center
   :width: 80%
  
Control plane subnet
~~~~~~~~~~~~~~~~~~~~

This subnet hosts Canonical's control services, including:

- A Juju controller: Canonical's orchestration layer.
- A jump host: used to securely manage and tunnel into the environment.

This subnet is protected by a Network Security Group (NSG) configured to:

- Deny all public inbound traffic.
- Allow access only from Canonical's internal infrastructure.

This ensures that the core management services are isolated from the public internet and reachable only through trusted internal channels.

Kubernetes cluster subnet
~~~~~~~~~~~~~~~~~~~~~~~~~

This subnet is automatically created as part of the Azure Kubernetes Service (AKS) deployment. It is:

- Peered with the control plane subnet to allow Canonical-managed services to communicate securely with the Kubernetes cluster.
- Protected by the same NSG rules, ensuring consistent traffic policies across both networks.

This architecture ensures that internal communication flows between the control and workload components remain secure and isolated from the public internet.

Public access and load balancers
--------------------------------

By default, only two services are publicly exposed:

- Kubeflow dashboard.
- Grafana Monitoring User Interface (UI).

These are each served via an Azure ``LoadBalancer`` service, which allocates a public IP address to make them accessible over HTTPS.

All other workloads and services within the Kubernetes cluster are internal-only.

IP restrictions
---------------

If you require restricted access to the public endpoints, our team can help you configure IP allowlists for Kubeflow through a post-deployment request. 

To do so, `open a support case <https://support-portal.canonical.com/>`_.

Custom integrations with Azure and private services
---------------------------------------------------

If you need to connect your deployment with other Azure services, such as storage accounts, key vaults, private DNS, or your own private infrastructure, you can request custom integrations by submitting a `support ticket <https://support-portal.canonical.com/>`_.

Our team will incorporate your requests in a way that ensures:

- They persist across rollouts and upgrades.
- They are tracked and managed declaratively, so they are not accidentally lost during future maintenance.

This approach ensures that any customizations you request are safely preserved throughout the lifecycle of your deployment.

Default networking configuration
--------------------------------

Here's an overview of the default networking configuration:

+------------------------------+-----------------------+----------------+-------------------------------+
| Component                    | Subnet                | Public access  | Notes                         |
+==============================+=======================+================+===============================+
| Juju controller & jump host  | Control plane subnet  | Denied         | Canonical-only access         |
+------------------------------+-----------------------+----------------+-------------------------------+
| AKS cluster                  | AKS subnet            | Denied         | Peered with control plane     |
+------------------------------+-----------------------+----------------+-------------------------------+
| Kubeflow dashboard           | AKS subnet            | Allowed        | Can be restricted via support |
+------------------------------+-----------------------+----------------+-------------------------------+
| Grafana UI                   | AKS subnet            | Allowed        | Can be restricted via support |
+------------------------------+-----------------------+----------------+-------------------------------+
| Other services               | AKS subnet            | Denied         | Internal only                 |
+------------------------------+-----------------------+----------------+-------------------------------+
| Additional integrations      | Optional              | Customizable   | Request via support ticket    |
+------------------------------+-----------------------+----------------+-------------------------------+

See also
--------

- Read about :ref:`node scheduling architecture <node_scheduling_architecture>` in Managed Kubeflow deployments.

