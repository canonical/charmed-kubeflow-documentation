.. _integrate_azure_blob_storage:

Integrate with Azure Blob Storage
=================================

This guide describes how to install Charmed Kubeflow (CKF) using `Terraform <https://www.terraform.io/>`_.

You can do so by using any `CNCF certified <https://www.cncf.io/training/certification/software-conformance/#logos>`_ Kubernetes (K8s) cluster 
and deploying Kubeflow using the Terraform, `Kubernetes <https://kubernetes.io/>`_ and `Juju`_ Command Line Interfaces (CLIs).

---------------------
Requirements
---------------------

* :ref:`A Charmed Kubeflow deployment <get_started>`.
* `An Azure account <https://azure.microsoft.com/en-us/free>`_.
* `An Azure Storage account <https://learn.microsoft.com/en-us/azure/storage/common/storage-account-create?tabs=azure-portal>`_.

---------------------
Assign permissions
---------------------

Before starting to work with Azure Blob Storage, ensure that you have the necessary permissions to access the storage account. 
Follow `this quickstart guide <https://learn.microsoft.com/en-us/azure/storage/blobs/storage-quickstart-blobs-python?tabs=managed-identity%2Croles-azure-portal%2Csign-in-azure-cli&pivots=blob-storage-quickstart-scratch#assign-roles-to-your-microsoft-entra-user-account>`_ to self-assign the ``Storage Blob Data Contributor`` role.

------------------------------------
Create and connect to a new notebook
------------------------------------

From the Kubeflow dashboard, navigate to ``Notebooks``, and click on ``New Notebook``. 
Select a ``JupyterLab`` environment, and connect to the newly created notebook.

-------------------------
Install required packages
-------------------------

On the JupyterLab launcher, click on ``Terminal`` to start a new terminal session. 
Next, install the packages required to connect to your Azure account and interact with it using the Python client library:

.. code-block:: bash

   pip install azure-cli azure-storage-blob azure-identity

.. note::

   The installation may take a few minutes to complete.

-----------------------------
Sign in to your Azure account
-----------------------------

Sign in to Azure through the Azure CLI using the following command:

.. code-block:: bash

   az login

Confirm that you have successfully logged in to your account:

.. code-block:: bash

   az account show

---------------------------------------------------
Connect to your Azure account via the Python client
---------------------------------------------------

Add a new tab in your JupyterLab environment, and then create a new Python3 notebook. 
Within a notebook cell, run the following code to connect to your Azure account:

.. code-block:: python

   import os, uuid
   from azure.identity import AzureCliCredential
   from azure.storage.blob import BlobServiceClient, BlobClient, ContainerClient

   try:
       print("Azure Blob Storage Python quickstart sample")

       account_url = "https://<storageaccountname>.blob.core.windows.net"
       default_credential = AzureCliCredential()

       blob_service_client = BlobServiceClient(account_url, credential=default_credential)

   except Exception as ex:
        print('Exception:')
       print(ex)

Replace the ``<storageaccountname>`` token with the storage account name you want to interact with.

---------------------------
Create a new blob container
---------------------------

You can create a new blob container by creating a new text file in the ``data`` directory and upload it as follows:

.. code-block:: python

   local_path = "./data"
   os.mkdir(local_path)

   local_file_name = str(uuid.uuid4()) + ".txt"
   upload_file_path = os.path.join(local_path, local_file_name)

   file = open(file=upload_file_path, mode='w')
   file.write("Hello, World!")
   file.close()

   container_name = str(uuid.uuid4())
   blob_client = blob_service_client.get_blob_client(container=container_name, blob=local_file_name)

   print("\nUploading to Azure Storage as blob:\n\t" + local_file_name)

   with open(file=upload_file_path, mode="rb") as data:
       blob_client.upload_blob(data)

Establish the local file name defining the ``local_file_name`` variable and the new container name defining the ``container_name`` variable.

.. note::

   See `Naming and Referencing Containers, Blobs, and Metadata <https://learn.microsoft.com/en-us/rest/api/storageservices/naming-and-referencing-containers--blobs--and-metadata>`_ for more information about naming containers.

-----------------------------
List the blobs in a container
-----------------------------

You can list all blobs in a specified container as follows:

.. code-block:: python

   print("\nListing blobs...")

   blob_list = container_client.list_blobs()
   for blob in blob_list:
       print("\t" + blob.name)

---------------------
Download blobs
---------------------

You can download blobs and save them to your local file system. Use the following code to download the blob specified by its name:

.. code-block:: python

   download_file_path = os.path.join(local_path, str.replace(local_file_name ,'.txt', 'DOWNLOAD.txt'))
   container_client = blob_service_client.get_container_client(container= container_name)
   print("\nDownloading blob to \n\t" + download_file_path)

   with open(file=download_file_path, mode="wb") as download_file:
       download_file.write(container_client.download_blob(blob.name).readall())

---------------------
Clean up resources
---------------------

Clean up the resources created throughout this guide by running the following code:

.. code-block:: python

   print("\nPress the Enter key to begin clean up")
   input()

   print("Deleting blob container...")
   container_client.delete_container()

   print("Deleting the local source and downloaded files...")
   os.remove(upload_file_path)
   os.remove(download_file_path)
   os.rmdir(local_path)

   print("Done")

Alternatively, you can also use the `Azure CLI <https://learn.microsoft.com/en-us/azure/storage/blobs/storage-quickstart-blobs-cli#clean-up-resources>`_ to do so.
