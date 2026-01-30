.. _run_spark_jobs:

Run Spark jobs
=====================

.. note::

   This feature is experimental and therefore should not to be used in a production environment.

This guide describes how to run Spark jobs from inside Kubeflow Notebooks and in Kubeflow Pipeline steps.

.. _run_spark_jobs_requirements:

---------------------
Requirements
---------------------

* An active CKF deployment with Apache Spark integration and access to the Kubeflow dashboard. See 
  :ref:`Integrate with Charmed Apache Spark <integrate_spark>` for more details.


.. _run_spark_jobs_on_kf_notebooks:

------------------------------------------
Run Spark job on Kubeflow Notebooks
------------------------------------------

This section describes how Spark jobs can be run from inside a Kubeflow Notebook environment.

.. _create_notebook_for_spark_job:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Create a notebook for running Spark jobs
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Create a :ref:`Kubeflow notebook <kubeflow_notebooks>`. This notebook is the workspace from which you run commands. 
When creating the notebook, make sure to:

1. Use Charmed Spark Jupyterlab OCI image (``ghcr.io/canonical/charmed-spark-jupyterlab:3.5-22.04_edge``) as the 
   notebook image.
2. Check the "Configure PySpark for Kubeflow notebooks" option under the "Configurations" section inside 
   "Advanced Options" to apply necessary PodDefaults to the notebook pod.

Connect to the notebook and start a new Python 3 notebook session from the Launcher.

.. _run_spark_job_using_notebook:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Run Spark jobs in notebook cells
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Once you are inside the notebook, you can access the Spark context using the in-built variable ``sc``. You can also 
import ``pyspark`` which comes pre-installed in the image, and then create Spark context by yourself.

To verify that Spark jobs can indeed be run from inside the notebook environment, run the following example code:

.. code-block:: python

    rdd = sc.parallelize([1, 2, 3, 4, 5], numSlices=2)
    result = rdd.map(lambda x: x * x).collect()
    print("Squared values:", result)


.. _run_spark_job_in_kf_pipeline:

-------------------------------------
Run Spark jobs in Kubeflow Pipeline
-------------------------------------

`Kubeflow Pipelines <https://www.kubeflow.org/docs/components/pipelines/concepts/pipeline/>`_ provide 
`steps <https://www.kubeflow.org/docs/components/pipelines/concepts/step/>`_ inside of which you can run Spark jobs 
by adding the ``access-spark-pipeline: true`` label to a step during the pipeline's definition. 
See the detailed steps below.

Create a :ref:`Kubeflow notebook <kubeflow_notebooks>` using the default notebook image. 
This notebook is the workspace from which you run commands.

Once you are inside the notebook, you can now define Kubeflow 
`components <https://www.kubeflow.org/docs/components/pipelines/concepts/component/>`_ that run Spark jobs using 
``pyspark``. It is recommended to use the ``SparkSession`` context manager in the ``spark8t`` Python package that 
comes along with the Charmed Spark Jupyterlab image to create Spark sessions to ensure that they are correctly 
closed at the end of the context manager. 

Make sure the `KFP SDK <https://kubeflow-pipelines.readthedocs.io/en/master/>`_ is 
installed in the Notebook's environment to be able to define and configure Kubeflow Pipelines in Python.

.. code-block:: bash

   !pip install kfp[kubernetes]

If you don't have a notebook ready, use the following code as an example. It creates a Pipeline with a single component 
that runs a trivial Spark job. 

.. note::

   Please make sure to use the correct Charmed Apache Spark image for Apache Spark version of your choice. 
   In the code below (see ``CHARMED_SPARK_OCI_IMAGE``), the image for Apache Spark 3.5 is used. 
   OCI image corresponding to other versions of Spark can be found 
   `here <https://github.com/canonical/charmed-spark-rock/pkgs/container/charmed-spark>`_.

.. code-block:: python

    from kfp import dsl, kubernetes

    CHARMED_SPARK_OCI_IMAGE = "ghcr.io/canonical/charmed-spark:3.5.5-22.04_edge"
    @dsl.component(
        base_image=CHARMED_SPARK_OCI_IMAGE,
    )
    def spark_test_component() -> None:
        import logging
        import os
        from spark8t.session import SparkSession

        with SparkSession(
            app_name="square_numbers`",
            namespace=os.environ["SPARK_NAMESPACE"],
            username=os.environ["SPARK_SERVICE_ACCOUNT"]
        ) as session:
            rdd = sc.parallelize(
                ["spark is fast", "spark is simple", "spark works"],
                numSlices=3
            )
            word_counts = (
                rdd
                .flatMap(lambda line: line.split())
                .map(lambda word: (word, 1))
                .reduceByKey(lambda a, b: a + b)
                .collect()
            )
            logging.info(f"Word counts: {word_counts}")


    @dsl.pipeline(name="spark-test-pipeline")
    def spark_pipeline():
        task = spark_test_component()
        kubernetes.pod_metadata.add_pod_label(
            task,
            label_key="access-spark-pipeline",
            label_value="true",
        )

Note that the label ``access-spark-pipeline: true`` has been added to the pipeline task pod, which is a necessary step
for us to refer to the ``SPARK_NAMESPACE`` and ``SPARK_SERVICE_ACCOUNT`` environment variables in the component definition
above.

Submit and run the Pipeline with the following code:

.. code-block:: python

    from kfp.client import Client
    client = Client()
    run = client.create_run_from_pipeline_func(
        spark_pipeline,
        experiment_name="Test Spark Job",
        enable_caching=False,
    )

Once the pipeline starts running, navigate to the output ``Run details``. In its logs, you can see the result as shown below:


.. code-block:: text

    Word counts: [('is', 2), ('fast', 1), ('simple', 1), ('works', 1), ('spark', 3)]