# Charmed Kubeflow documentation

This is the code repository for Charmed Kubeflow (CKF) documentation.
It is based on [Canonical's Sphinx starter pack](https://github.com/canonical/sphinx-docs-starter-pack).

See [CKF official documentation](https://charmed-kubeflow.io/docs) for the rendered documentation.

## Build the documentation

Run the following command within the `docs` folder to build the documentation: 

.. code-block:: none

   make run

## Local checks

Before committing and pushing changes, run various checks locally with the following commands:

- Check links 

.. code-block:: shell

   make linkcheck

- Check spelling 

.. code-block:: shell

   make spelling

- Check inclusive language 

.. code-block:: shell

   make woke

- Check accessibility

.. code-block:: shell

   make pa11y

See [Canonical's Sphinx starter pack documentation](https://canonical-starter-pack.readthedocs-hosted.com/latest/) for more details.