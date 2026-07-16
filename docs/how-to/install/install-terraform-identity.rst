.. _install_terraform_identity:

Install with Canonical Identity Platform using Terraform
========================================================

This guide describes how to install Charmed Kubeflow (CKF) integrated with the
`Canonical Identity Platform <https://charmhub.io/topics/canonical-identity-platform>`_ using `Terraform`_.

This solution runs CKF on the `Istio Ambient Mesh`_ and integrates it with the Canonical Identity Platform
(Hydra, Kratos and the Login UI) so that authentication is handled by the Identity Platform.

.. note::

   This integration is available in Istio ambient mode only.

The deployment spans four `Juju models <https://juju.is/docs/juju/model>`_:

* ``istio-system``: the Istio ambient control plane (``istio-k8s``).
* ``iam``: the Canonical Identity Platform bundle (Hydra, Kratos and the Login UI).
* ``iam-core``: the Identity Platform dependencies (``postgresql-k8s``, ``traefik`` and ``self-signed-certificates``).
* ``kubeflow``: the CKF applications, the two ambient ingress gateways (UI and machine-to-machine) and the Identity Platform authentication charms.

.. TODO: Update this guide once the ``feat/iam-integration`` branch of charmed-kubeflow-solutions is merged, including the repository URL, branch and module path.

---------------------
Requirements
---------------------

* A Kubernetes (K8s) cluster that:

  * uses a version supported by Charmed Kubeflow (see :ref:`Supported versions <supported_kubeflow_versions>`), with a default `storage class <https://kubernetes.io/docs/concepts/storage/storage-classes/>`_ configured.
  * has a load balancer provider, so that the ingress gateways and the Identity Platform can be exposed through ``LoadBalancer`` services. For example, `MetalLB <https://metallb.io/>`_ on `MicroK8s`_.
  * meets the `Istio platform prerequisites`_ for the ambient mesh.
* `Terraform CLI <https://developer.hashicorp.com/terraform/cli>`_. You can install it using the `snap`_.
* `kubectl <https://kubernetes.io/docs/reference/kubectl/>`_ configured to access your cluster.

---------------------
Bootstrap Juju
---------------------

CKF is deployed to Kubernetes with Juju.
Before deployment, a Juju controller must be bootstrapped to the K8s cluster.
See `Get started with Juju <https://documentation.ubuntu.com/juju/latest/tutorial/>`_ for more details.

.. note::

   Check :ref:`Supported versions <supported_kubeflow_versions>` for version compatibility between CKF, Juju and K8s.

-------------------------------------
Deploy CKF with the Identity Platform
-------------------------------------

Deploy the solution as follows:

1. Clone the ``charmed-kubeflow-solutions`` repository and change directory to the solution module:

.. code-block:: bash

   git clone https://github.com/canonical/charmed-kubeflow-solutions
   cd charmed-kubeflow-solutions
   git checkout feat/iam-integration
   cd terraform-refactoring/tests/kubeflow-ambient-iam

2. Initialise Terraform. The following command downloads all the required `Terraform modules <https://developer.hashicorp.com/terraform/language/modules>`_ and installs the Terraform `Juju provider <https://registry.terraform.io/providers/juju/juju/latest/docs>`_:

.. code-block:: bash

   terraform init

3. Prepare a Profile Management Representation (PMR) in a Git repository.

The ``github-profiles-automator`` charm keeps the Kubeflow profiles in sync with a YAML file stored in a GitHub repository.
Create a repository (for example, ``https://github.com/example-org/kubeflow-pmr``) containing a ``pmr.yaml`` file at its root, such as:

.. code-block:: yaml

   profiles:
   - name: ml-engineering
     owner:
       kind: User
       name: user1

.. note::

   You can choose any ``name`` for the profile (for example, ``ml-engineering``); Kubeflow creates a namespace of the same name.
   The profile ``owner.name`` must match the Kratos username you create in the :ref:`Create a user <create_user_identity>` section.
   In this solution, the profile owner maps to the Identity Platform (Kratos) username, not to an email address.

   See :ref:`Manage profiles <manage_profiles>` for the full ``pmr.yaml`` format, including contributors and resource quotas.

4. Configure the ``github-profiles-automator`` charm to sync from your PMR repository by creating a ``terraform.tfvars`` file in the current directory:

.. code-block:: terraform

   # github-profiles-automator: sync Kubeflow profiles from this PMR repository.
   github_profiles_automator_config = {
     repository = "https://github.com/example-org/kubeflow-pmr.git"
   }

.. note::

   Terraform automatically loads ``terraform.tfvars`` during ``terraform apply``.

   By default, the ``github-profiles-automator`` charm reads a file named ``pmr.yaml`` at the repository root. To use a different file or path, set ``pmr-yaml-path`` in ``github_profiles_automator_config``.

5. Deploy the solution using Terraform, setting the external hostnames for the ingress gateways and the Identity Platform:

.. code-block:: bash

   terraform apply \
      -var external_ui_hostname="ui.kubeflow.com" \
      -var external_m2m_hostname="api.kubeflow.com" \
      -var external_auth_hostname="auth.kubeflow.com"

.. note::

   These hostnames do not need to be registered with a public DNS provider. You make them resolvable in the :ref:`Configure DNS for the ingress gateways <configure_dns_identity>` section below.

The command above:

* Creates four Juju models: ``istio-system``, ``iam``, ``iam-core`` and ``kubeflow``.
* Deploys the Istio ambient control plane into ``istio-system``.
* Deploys the Canonical Identity Platform bundle into ``iam`` and its dependencies (including ``traefik``) into ``iam-core``.
* Deploys CKF with the two ambient ingress gateways and the Identity Platform authentication charms into ``kubeflow``.
* Deploys the ``github-profiles-automator`` charm, which syncs Kubeflow profiles from your PMR repository.
* Sets the external hostname on the UI gateway (``ui.kubeflow.com``), the machine-to-machine gateway (``api.kubeflow.com``) and the Identity Platform ingress (``auth.kubeflow.com``).

See `kubeflow-ambient-iam deployment <https://github.com/canonical/charmed-kubeflow-solutions/blob/feat/iam-integration/terraform-refactoring/tests/kubeflow-ambient-iam/README.md>`_ for more details.

6. Verify all charms are in ``active`` status by monitoring the Juju models:

.. code-block:: bash

   juju status -m istio-system --watch 1s
   juju status -m iam --watch 1s
   juju status -m iam-core --watch 1s
   juju status -m kubeflow --watch 1s

.. note::

   Deployment may take several minutes to complete, depending on the cluster's node specifications.

.. _configure_dns_identity:

--------------------------------------
Configure DNS for the ingress gateways
--------------------------------------

The ingress gateways and the Identity Platform are exposed through ``LoadBalancer`` services using the hostnames you configured during deployment.
For these hostnames to resolve, you need to configure DNS in two places:

* **In-cluster DNS (CoreDNS)**, so that in-cluster workloads (for example, the authentication redirects between the gateways and the Identity Platform) can resolve the hostnames.
* **Host DNS**, so that the machine running your browser (the cluster host itself or a separate workstation) can reach the gateways.

1. Get the ``LoadBalancer`` IP addresses of the gateways and the Identity Platform ingress:

.. code-block:: bash

   UI_IP=$(kubectl -n kubeflow get svc istio-ingress-k8s-ui-istio \
       -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
   API_IP=$(kubectl -n kubeflow get svc istio-ingress-k8s-m2m-istio \
       -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
   AUTH_IP=$(kubectl -n iam-core get svc traefik-lb \
       -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
   echo "UI   ui.kubeflow.com   -> $UI_IP"
   echo "API  api.kubeflow.com  -> $API_IP"
   echo "AUTH auth.kubeflow.com -> $AUTH_IP"

Configure in-cluster DNS (CoreDNS)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

2. Find the CoreDNS ``ConfigMap`` name and store it in a variable to reuse it in the following steps:

.. code-block:: bash

   COREDNS=$(kubectl -n kube-system get configmap -o name | grep coredns | head -n1 | cut -d/ -f2)
   echo "$COREDNS"

The ``ConfigMap`` name depends on your K8s distribution: ``ck-dns-coredns`` on Canonical Kubernetes and ``coredns`` on MicroK8s.

3. Edit the CoreDNS ``ConfigMap``:

.. code-block:: bash

   kubectl -n kube-system edit configmap "$COREDNS"

Add a ``hosts`` block inside the ``.:53 { ... }`` server block, just before the ``kubernetes`` plugin line, using the IP addresses from step 1:

.. code-block:: text

   hosts {
       <UI_IP>   ui.kubeflow.com
       <API_IP>  api.kubeflow.com
       <AUTH_IP> auth.kubeflow.com
       fallthrough
   }

.. note::

   The ``fallthrough`` directive ensures that any query not matching these hostnames is still resolved by the rest of the CoreDNS configuration.

4. Restart CoreDNS to apply the change:

.. code-block:: bash

   kubectl -n kube-system rollout restart deployment "$COREDNS"

Configure host DNS
~~~~~~~~~~~~~~~~~~~

5. On the machine where you access the CKF dashboard from a browser, add the hostnames to ``/etc/hosts`` using the IP addresses from step 1:

.. code-block:: bash

   echo "$UI_IP ui.kubeflow.com"     | sudo tee -a /etc/hosts
   echo "$API_IP api.kubeflow.com"   | sudo tee -a /etc/hosts
   echo "$AUTH_IP auth.kubeflow.com" | sudo tee -a /etc/hosts

.. _create_user_identity:

---------------------
Create a user
---------------------

The ``github-profiles-automator`` charm creates the Kubeflow profiles defined in your ``pmr.yaml`` file automatically.
Each profile is owned by a user whose identity is provided by the Canonical Identity Platform.
To log in and own a profile, create a matching user in Kratos.

1. Create a Kratos user whose username matches the ``owner.name`` of a profile in your ``pmr.yaml`` file, using the Kratos `create-admin-account <https://charmhub.io/kratos/actions#create-admin-account>`_ action:

.. code-block:: bash

   juju run -m iam kratos/0 create-admin-account \
      username=user1 \
      email=user1@example.com

.. note::

   The ``owner.name`` in ``pmr.yaml`` maps to the Kratos ``username``, so they must be identical (for example, ``user1``).

The action output includes a link that the user opens to set their account password.

2. Confirm that the profile defined in your ``pmr.yaml`` file has been created:

.. code-block:: bash

   kubectl get profiles

See :ref:`Manage profiles <manage_profiles>` for more details on managing profiles and the ``pmr.yaml`` format.

---------------------
Access CKF dashboard
---------------------

Once DNS is configured, you can access the CKF dashboard at ``https://ui.kubeflow.com``.
You are redirected to the Canonical Identity Platform to authenticate with the user you created.

The gateways and the Identity Platform are served with certificates issued by the ``self-signed-certificates`` charm by default.
Because these certificates are not signed by a trusted certificate authority (CA), your browser may warn that the connection is not trusted.

* For a test or development deployment, accept the certificate warning in your browser to proceed to the login page.
* For a production deployment, replace ``self-signed-certificates`` with a certificate provider backed by a trusted CA, so that browsers trust the certificates and no warning is shown.

See the `Canonical Identity Platform documentation <https://canonical-identity.readthedocs-hosted.com/>`_ for more details on securing the Identity Platform.
