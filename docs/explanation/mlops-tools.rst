.. _mlops_tools:

MLOps tools
===========

This guide presents some Machine learning operations (MLOps) tools integrated in Charmed Kubeflow (CKF). 
MLOps are a set of practices that automate and simplify ML workflows and deployments.

Katib
-----

`Katib <https://www.kubeflow.org/docs/components/katib/>`_ is a Kubernetes-native project for automated machine learning (AutoML). 
Katib supports hyperparameter tuning, early stopping and neural architecture search (NAS).

Katib is agnostic to ML frameworks. 
It can tune hyperparameters of applications written in any language of the users' choice and natively supports many ML frameworks, 
such as TensorFlow, MXNet, PyTorch, XGBoost, and others.

Kubeflow Pipelines
------------------

`Kubeflow Pipelines (KFP) <https://www.kubeflow.org/docs/components/pipelines/v1/introduction/>`_ is a workflow engine that allows specifying tasks and their configuration, environment variables and secrets. 
Additionally, KFP provides task execution scheduling. 

MinIO
-----

`MinIO`_ is a secured object storage system. 
It can be used as a standalone product or as a cloud storage gateway. 
For cloud use, it provides an AWS S3-compatible API.  

MLflow
------

`MLflow`_ is an experiment and model repository that enables model tracking including metadata, training results and model comparison.

Seldon Core
-----------

`Seldon Core <https://docs.seldon.io/projects/seldon-core/en/latest/>`_ is a platform to deploy ML models on Kubernetes at scale as microservices. 
It supports REST and gRPC protocols, manual and auto-scaling.