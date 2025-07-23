:orphan:
:tocdepth: 2

.. _faq_managed_kf:

Support and FAQ
===========================

This guide answers some of the most common asked questions about Managed Kubeflow on Azure.

------------------------------
What does the service include?
------------------------------

Canonical offers managed services for every managed application cluster deployed on Azure marketplace. This significantly accelerates time-to-market and ensures operational excellence. The following services are provided:

~~~~~~~~~~~~~~~~~~
Business-as-usual
~~~~~~~~~~~~~~~~~~

* Monitoring.
* Alerting.
* Integrated observability leveraging `COS <https://ubuntu.com/blog/tag/canonical-observability-stack>`_.

~~~~~~~~~~~~~~
Planned events
~~~~~~~~~~~~~~

* Upgrades.
* Patching.
* Migration.
* Cluster scaling.
* Security hardening.

~~~~~~~~~~~~~~~~~~
Unplanned events
~~~~~~~~~~~~~~~~~~

* Incident response.
* Bug fixing.
* Backup & restoration.

~~~~~~~~~~~~~~~~~~~~
Additional services
~~~~~~~~~~~~~~~~~~~~

* Upstream Kubeflow contributions, such as bug and CVE reports.
* Automated ticketing system.
* Multi/hybrid cloud integration and compatibility.

---------------------------------------
Why do you need an Ubuntu One account?
---------------------------------------

Ubuntu One is Canonical's identity manager, where every formal interaction with Canonical uses it for authentication. This helps us identify who you are and what our relationship is in a faster and easier way.

.. important::

   When creating your Ubuntu One account, make sure the email address you use matches the email address associated with the Azure account from which you deployed the Managed Kubeflow service.

If you do not use an Ubuntu One account, Canonical will not be able to upgrade or maintain your environments, monitor your usage, bill you adequately, or stay in touch.

--------------------------
How is the service priced?
--------------------------

See Canonical's `pricing model <https://pages.ubuntu.com/rs/066-EOV-335/images/Managed%20Apps%20on%20Public%20Cloud%20Pricing%20Explanation%20Datasheet.pdf?version=0&_ga=2.166526398.1658000811.1730716752-1990936172.1718804489>`_ to better understand how our managed services are priced for all managed applications provided on Azure marketplace.

------------------------------------------
What tools do you get with the deployment?
------------------------------------------

Your deployment is made up of Charmed Kubeflow and Charmed MLflow:

.. image:: https://lh7-rt.googleusercontent.com/docsz/AD_4nXfEVULlcLqEJpNN2LxLz1SBFA9tK_DiR_gtsMrXkYuJBNW97uGboVx0NJCqTlYxLhAN9gohtdCKL2oCeBxr63X6fUZUl2BkbFp8PrneMOBT2Lw3EatZSxKll714woy1BCO48Bdp_Q?key=BHCZxQMTEo6n6w4x2fqyLQ

For customised deployments, such as different flavors of the provided applications or integrations with other open source tooling, please `contact our sales department <https://ubuntu.com/managed>`_.

------------------------------------
Where can you raise issues and bugs?
------------------------------------

Issues and bugs can only be raised through a support ticket using `Canonical Support Portal <https://support-portal.canonical.com/>`_.

.. important::

   An Ubuntu One account is required to access the portal.

You receive a direct link to the support portal in the welcome email, after your Managed Kubeflow deployment is complete.

----------------------------------
How do you raise a support ticket?
----------------------------------

Go to the `Canonical Support Portal <https://support-portal.canonical.com/>`_. Once you're signed in with your Ubuntu One account, click on the ``New Ticket`` button on the portal's homepage:

.. image:: https://lh7-rt.googleusercontent.com/docsz/AD_4nXdZlS-g1fzcqmp-Isw0Mn6W7jPjngjOoseLCNazWqPSCJ1FZiBMTV3ylWsRKb_iPv7Dq1nItNP5HmcNpeP2xmLmqI4oSprAdr9Ou2dwau-Nisc13fnBV6ymcLDmE_z55wYVL-14ri8tljG7kgeHLZ1W7XEs?key=BHCZxQMTEo6n6w4x2fqyLQ

Follow the steps in the pop-up window to submit your ticket.

---------------------------------------
How do you transfer out of the service?
---------------------------------------

You can transfer out of Canonical's managed services on Azure in two different ways:

1. Complete system decommissioning.

   This option involves a complete deletion of your cluster, along with all the data that runs on top of it.

   You can do this directly from Azure marketplace.

2. Managed Services decommissioning.

   With this option, Canonical transfers the resource group management capabilities to your team and will stop managing your clusters after that. Your data and workloads will remain active, and will be transferred to a repository where only you have access and admin rights.

   This option would allow you to continue using your Canonical application, without managed services. Please reach out using `Canonical Support Portal <https://support-portal.canonical.com/>`_ for this request.

----------------------------------------------
What is the best way to delete the deployment?
----------------------------------------------

See `Azure documentation <https://learn.microsoft.com/en-us/marketplace/create-manage-private-azure-marketplace-new>`_ for more details.

-------------------------------------------------
Can you use spot instances with Managed Kubeflow?
-------------------------------------------------

Yes, you can. See :ref:`this guide <integrate_azure_spot_vms>` to use Charmed Kubeflow with spot instances.

----------------------------------
How can you scale in or scale out?
----------------------------------

Your deployment will auto-scale depending on your workload requirements, within the limits you set at the beginning of your deployment. You can change these limits whenever you want on the Azure portal.

If you want to perform a larger scale-in or scale-out transaction, such as a migration or the addition of a separate service, please raise a ticket using `Canonical Support Portal <https://support-portal.canonical.com/>`_.

-------------------------------
Can you customise the solution?
-------------------------------

The service provided on Azure marketplace offers a version of Charmed Kubeflow and Charmed MLflow that should cover most of the relevant Machine Learning use cases nowadays. This flavor of Charmed Kubeflow cannot be changed on the marketplace or Microsoft Azure portal.

However, Canonical offers fully customised solutions for any customer via private offering. To obtain a quote for a customized solution, please reach out to our `Sales department <https://ubuntu.com/managed>`_.

-------------------------------------------------
Can you use your already existing AKS deployment?
-------------------------------------------------

You cannot since Charmed Kubeflow only works on new deployments.

If you've got a redundant AKS deployment that you wish to run Kubeflow on, Canonical recommends decommissioning it, and starting a new deployment via Azure Marketplace listing. Follow :ref:`this tutorial <install_aks>` to ensure your deployment is properly set up.

----------------------
Who can you contact?
----------------------

If you encounter an issue with Azure marketplace listing and offer, or would like to know more details of the offered services, please `reach out to our sales department <https://ubuntu.com/managed>`_.

If you encounter an issue while deploying the service, please refer to :ref:`Managed Kubeflow on Azure documentation <index_managed_kubeflow>`. If your question or problem is not addressed there, get in touch with `support@canonical.com <mailto:support@canonical.com>`_.

If you have already deployed your managed cluster on Azure, please raise any concern or issue by opening a ticket using `Canonical Support Portal <https://support-portal.canonical.com/>`_.

----------------
Get further help
----------------

Contact `Canonical Managed Services <https://ubuntu.com/managed>`_ for any additional questions.
