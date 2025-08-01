.. _manage_profiles:

Manage profiles
===============

This guide describes how you can manage profiles in Charmed Kubeflow (CKF).
In Kubeflow, a profile denotes a collection of resources, roles, and credentials.
Each profile is owned by one user but can have multiple contributors with specified access.

For each profile, Kubeflow creates a namespace of the same name that encapsulates the resources specific to that profile.
See the `upstream documentation <https://www.kubeflow.org/docs/components/central-dash/profiles/>`_ for more details.

---------------------
Requirements
---------------------

* A CKF deployment. See :ref:`Get started <get_started>` for more details.
* `Juju`_ version >= ``3.4``.
* Juju admin access to the ``kubeflow`` model.
* ``kubectl`` binary.

-----------------------------
Manage profiles automatically
-----------------------------

CKF supports the automatic creation of profiles within a cluster based on a single source of truth.
This can be done by deploying and configuring an operator charm.
This charm is responsible for periodically polling the reference source, and updating the cluster profiles and contributors accordingly:

- When a profile is defined in the source of truth but does not exist in the cluster, the operator creates the profile in the cluster.

- When a profile is defined in the source of truth and already exists in the cluster, the operator only updates the ``resourceQuota``, if needed.

.. note::

   This process is stateless. The operator charm only acts upon the current view of the source of truth, and does not contain the history of previous updates.

Profiles that already exist in the cluster but have been removed from the source of truth are considered ``stale``, but are not automatically deleted to avoid data loss. Instead, all contributors are removed from the profile, so no user has access to the profile.

.. note::

   Stale profiles have to be explicitly deleted using a Juju action.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Configure profiles automatically from GitHub
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You can define profiles and contributors from a specified file in a GitHub repository, and have CKF automatically sync the cluster with that specification.
This can be done by deploying and configuring the ``github-profiles-automator`` charm.

This charm uses a YAML file to configure the cluster profiles.
If the YAML file is updated, for instance, adding new profiles, the charm automatically creates those profiles in the cluster.

The YAML file must follow the following format:

.. code-block:: yaml

    profiles:
    - name: ml-engineers
      owner:
        kind: User
        name: admin@canonical.com
      resources:
        hard:
          limits.cpu: "1"
      contributors:
      - name: kimonas@canonical.com
        role: admin
      - name: michal@canonical.com
        role: edit
      - name: noha@canonical.com
        role: view
    - name: data-engineers
      owner:
        kind: User
        name: admin@canonical.com
      contributors:
      - name: daniela@canonical.com
        role: edit
      - name: manos@canonical.com
        role: view

~~~~~~~~~~~~~~~~~~~
Configure the charm
~~~~~~~~~~~~~~~~~~~

First, deploy the charm on your existing CKF deployment:

.. code-block:: bash

   juju deploy github-profiles-automator --channel=latest/edge --trust

Then, provide the repository URL and YAML file path:

.. code-block:: bash

   juju config github-profiles-automator repository=”<URL ending in .git>”
   juju config github-profiles-automator pmr-yaml-path=”<path-to-file>

.. note::

   The charm supports both HTTPS and SSH GitHub URLs.

You can configure the repository Git revision and the period between sync attempts using ``juju config`` as follows:

.. code-block:: bash

   juju config github-profiles-automator git-revision=”<revision>”
   juju config github-profiles-automator git-revision “<time-in-seconds>”

To confirm the profiles have been added, list the existing profiles with the following command:

.. code-block:: bash

   kubectl get profiles

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Set an SSH key to access private repositories
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

SSH URLs require passing a private SSH key to the charm's configuration.
This guide assumes that a public SSH key has been added to your GitHub account.
See `GitHub documentation <https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account>`_ on how to add SSH keys.

Find the path to the private key that corresponds to the public SSH you added, and create a Juju user secret by running:

.. code-block:: bash

   juju add-secret ssh-key-secret ssh-key="$(cat <path-to-key>)"

Juju creates the secret and prints its unique ID. First, grant access to the ``github-profiles-automator`` charm:

.. code-block:: bash

   juju grant-secret ssh-key-secret github-profiles-automator

Now pass the secret's ID to the charm's configuration:

.. code-block:: bash

   juju config github-profiles-automator ssh-key-secret-id=<secret-id>

The charm is now able to sync with private repositories that you have access to.

.. note::

   For using an SSH key you'll also need to ensure the repository URL is in the form of `git@github.com:...` or `ssh://git@github.com/...`. Else, if it starts with `https://github.com/...` then the charm will fail to pull the repository.

~~~~~~~~~~~~~~~~~~~
Run Juju actions
~~~~~~~~~~~~~~~~~~~

The cluster profiles are synced with the provided YAML file each time the charm's configuration is changed, and periodically when its status is updated.
To manually sync the charm’s profile, run the ``sync-now`` action:

.. code-block:: bash

   juju run github-profiles-automator/0 sync-now

If a profile currently exists in the cluster, but isn't described in the YAML file, it is considered stale.
To list all stale profiles, run the ``list-stale-profiles`` action:

.. code-block:: bash

   juju run github-profiles-automator/0 list-stale-profiles

~~~~~~~~~~~~~~~~~~~
Delete profiles
~~~~~~~~~~~~~~~~~~~

To delete all ``stale`` profiles, run the ``delete-stale-profiles`` action:

.. code-block:: bash

   juju run github-profiles-automator/0 delete-stale-profiles

.. warning::

   This action deletes all resources belonging to the profile's namespace.

------------------------
Manage profiles manually
------------------------

In CKF, profiles can be created manually using `kubectl <https://kubernetes.io/docs/reference/kubectl/>`_.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Create profiles with ``kubectl``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

First, create a YAML file that describes the profile you want to create, and then apply it to your Kubernetes cluster using ``kubectl apply``.
See `Create a profile <https://www.kubeflow.org/docs/components/central-dash/profiles/#create-a-profile>`_ for more details.

~~~~~~~~~~~~~~~~~~~
Delete profiles
~~~~~~~~~~~~~~~~~~~

You can delete a profile as described in the upstream project.
See `Delete a profile <https://www.kubeflow.org/docs/components/central-dash/profiles/#delete-a-profile>`_ for more details.
