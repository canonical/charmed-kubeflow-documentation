# Charmed Kubeflow documentation

This is the code repository for Charmed Kubeflow (CKF) documentation.
It is based on [Canonical's Sphinx starter pack](https://github.com/canonical/sphinx-docs-starter-pack).

See [CKF official documentation](https://charmed-kubeflow.io/docs) for the rendered documentation.

## Get started

### Build this documentation

Run the following command within the `docs` folder to build the documentation: 

```
   make run
```

### Local checks

Run various checks locally with the following commands:

- Check links 

```
   make linkcheck
```

- Check spelling 

```
   make spelling
```

- Check inclusive language 

```
   make woke
```

- Check accessibility

```
   make pa11y
```

See [Canonical's Sphinx starter pack documentation](https://canonical-starter-pack.readthedocs-hosted.com/latest/) for more details.