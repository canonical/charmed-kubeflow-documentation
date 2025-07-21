:orphan:

.. _enable_microsoft_providers:

Enable Microsoft providers
==========================

This guide describes how to register your ``Microsoft.Capacity``, ``Microsoft.Storage``, 
and ``Microsoft.Compute`` providers required to set up a Canonical Managed application.

.. important::

   If you are already on the ``Resource Providers`` page, skip to step 4.

1. Go to the Azure ``Subscriptions`` page by searching on the top bar:

.. image:: https://lh7-rt.googleusercontent.com/docsz/AD_4nXf_L0-kjaW4d944K9njio6O0UAZSFMMnB9Bz0Oiog_Y26KVuLl1-tvwzcjwmOzlCnBu9Q42Qwz8P46lAqX8leIjkv4pMBLWLNpgu7sdVaymrb__p--N06-Q32ivDxG-SDH4JfLRxg?key=kZJjKrS7KBF2-Owe_pHRK2w3

2. Click on the subscription you want to use for the setup:

.. image:: https://lh7-rt.googleusercontent.com/docsz/AD_4nXcwLJbEUV38UrUKtkpXVfWeInc8B-tW09JNNa_JiGV-wR8UwuN3757gzMdz0BAChMClHdTN1wI6yuZ8htdaCpUsjiIl6tamUO3Dp1Ofwww2CZNyORKwbyf9k6cq8vmqB9Zuxa9f?key=kZJjKrS7KBF2-Owe_pHRK2w3

3. Search for ``providers`` on the menu search box and click on ``Resource Providers``:

.. image:: https://lh7-rt.googleusercontent.com/docsz/AD_4nXctJdTmKv3B_79pefbrJDgQbfo7OkmIBWB9nZ0bmv2gAj1QJd2ktzzYWR8Z2F5jV_Hd5v252UdUEgfRVoKb5JdjwUsk8lut8ZWceJfMtGjAP-FPDS1QtsstkfZGvZOlykkCZJLMAQ?key=kZJjKrS7KBF2-Owe_pHRK2w3

4. Search for ``capacity`` using the ``Filter by name`` field:

.. image:: https://lh7-rt.googleusercontent.com/docsz/AD_4nXcvOk6ODQ2aKm9fHcmGtpY-Svebij_ktHK0h8kdNPCF-kQLM2jKiJnqOs0bCsbDvTxDIt004YxGFq7EBjiycTlbvTc0bCtVJMReTL2j08umOnvPflV70TzkWh8wKqlE0hkCpw22xg?key=kZJjKrS7KBF2-Owe_pHRK2w3

5. You should see that ``Microsoft.Capacity`` status is ``notRegistered`` or ``unregistered``:

.. image:: https://lh7-rt.googleusercontent.com/docsz/AD_4nXdLY6I2cIfqx_y0LolsQ-VeduQ0de4IdZ-EHNcDwiDk0nndi5K_ST3sjK4Hq0sDRNVKjpclQ3EdEIJIS05sQT6-dHEPd1uVOKA8Mfum3xAByF0RBJA4hlnb8FJVGXQaVv42XtiC?key=kZJjKrS7KBF2-Owe_pHRK2w3

6. Select it by checking the circle before the name and click ``Register`` on the top:

.. image:: https://lh7-rt.googleusercontent.com/docsz/AD_4nXd7oyyVR70675FlfKCfliGLgZFEdbscCRyw5GIF47eQvCMGhG9hSvTGM176sOjoC6NRSbWYrTYxfmze9KL03_4BqX0EElJQL2oRc4_LSd_aKS1334pUJtHXbD1qF6iNAfaibwJHMw?key=kZJjKrS7KBF2-Owe_pHRK2w3

7. Wait until the provider status changes to ``Registered``:

.. image:: https://lh7-rt.googleusercontent.com/docsz/AD_4nXebYnd2grfqgqN0YzwPj_C_REsRnzGwEVyJfHg66ItVf1eYouwo97SRot5R0XfB9kTXsmw52BuTiJKgKjvFzHqtAk7G5XCWySO_MdtnkaQ09tvsGM4zt8sr9kSxK1GeOTHZaBJqhg?key=kZJjKrS7KBF2-Owe_pHRK2w3

8. Now do the same for ``Microsoft.Compute``.
9. Search for it using the ``Filter by name``.
10. Select it by checking the circle before the name, and click the ``Register`` button.
11. Wait until the provider changes to ``Registered``:

.. image:: https://lh7-rt.googleusercontent.com/docsz/AD_4nXfAk3p6OiEqu-5aoE3KAWTzDKnGSvAOsYxMK6RjcADnl4ALhcmad7FnDp6Ju6SK5V-abqOhLSpfViraB9OliUrZxs8WO8DvPc7CTA5RDlOrYmJXT4miHvTVhwmwEgxiRcbacfjdLg?key=kZJjKrS7KBF2-Owe_pHRK2w3

12. Now do the same for ``Microsoft.Storage``.
13. Search for it using the ``Filter by name``.
14. Select it by checking the circle before the name, and click the ``Register`` button.
15. Wait until the provider changes to ``Registered``:

.. image:: https://lh7-rt.googleusercontent.com/docsz/AD_4nXeVYgexNeAhjD93qnRMoXH8m1v6oxAXRlLeacesAnxZT3PmLMTGS5UkAezQ4BT34sZjMCgApg6Sn59EjPy233ZgvLy0-C4RFEaq3vP4SZdJHlS152gBRkzW53PrI0icIB3yrsuvTw?key=kZJjKrS7KBF2-Owe_pHRK2w3

16. Once all providers are registered, you need to restart your setup. The easiest way is to close the setup tab in your browser and restart it from the `Azure Marketplace <https://portal.azure.com/#create/canonical.managed-kubeflowkubeflow-metered>`_.

