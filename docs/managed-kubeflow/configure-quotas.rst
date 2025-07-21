:orphan:

.. _configure_quotas_azure:

Configure quotas on Azure
=========================

This guide describes how to configure Azure quotas to set up a Canonical Managed application.

.. note::

   All information needed to change the quota is in the error message.

1. Get started by visiting `the quota page <https://portal.azure.com/#view/Microsoft_Azure_Capacity/QuotaMenuBlade/~/myQuotas>`_.
2. Find the region where you need additional quota: 

.. image:: https://lh7-rt.googleusercontent.com/docsz/AD_4nXfDORUbBI8SwdAXunt8HEZhQOz1BSs6rXvD2SPYaGqtLHoVHwluowUB7qTrS0HJDPtS6Fms76J5L5OnFwy_EkY7fSOH00NDmXQk0TilUDmpWwHs8WoHKcXuz2umx1QU07U6PXw7?key=vfYKUiktMu-Yv0oKyjtkUbdU

3. Use ``Region`` to filter the information in the quota page as shown below:

.. image:: https://assets.ubuntu.com/v1/acb7de82-Azure%20quotas.png

Now that only the CPU families available in that region are shown, request the additional quota for all the CPU/GPU families listed in the error message:

.. image:: https://lh7-rt.googleusercontent.com/docsz/AD_4nXcnZA5LWASDErtsjcGBMDua8kprLXE2fi5jKmCDFmjLCQOUSc7axUaC32BA-Wih43v0FQ3dW4LJ0mkSJvaA9mSB08jV9Q9kV6NizVfKK-aVtdY4LZ__LG6xogC_YoT_Qczzfn7p?key=vfYKUiktMu-Yv0oKyjtkUbdU

4. From the error message, copy the CPU/GPU family:

.. image:: https://lh7-rt.googleusercontent.com/docsz/AD_4nXffB3aVdKl3J04ylATiT9Wua_uoX00zdqLXcewgoL805ZSEn9dD5DVaNwREW2rtEUbWXPnWVREOqTqW3jdcHLQSq7-DHKLi2bFz_X9o2S_jeKr6UR449zTO042InuMmZxIlwhAk?key=vfYKUiktMu-Yv0oKyjtkUbdU

5. Paste the family in the search box of the quota page:

.. image:: https://lh7-rt.googleusercontent.com/docsz/AD_4nXd8EeYDZX4lDWmdx5RfS7GFUokIsO-6WX40-Taany354jeE7wLhZzC7GmOuWGNI40TSVhOiCuIzAQNtuFbMQ1Z4J_DQgH154ICy3jPJYIesTw9qH9Fqn2ZbmIof1sMccwYi8tIm?key=vfYKUiktMu-Yv0oKyjtkUbdU

6. Click on the pen icon at the far end of the line, a new screen will appear:

.. image:: https://lh7-rt.googleusercontent.com/docsz/AD_4nXfxUN9ePtwVVp3Wd_B_YFEcTdRlEs3jBzZ61jRZC_lfzoM7nw4cCCaOuANjngX6vFY0UIClUrauNGuRRKvoaWxfwh46tozxSC_TiLRTjQqfokc0KOZoU7jQl3xUsU3ChsdjosR9?key=vfYKUiktMu-Yv0oKyjtkUbdU

7. Set ``New limit`` to the suggested value:

.. image:: https://lh7-rt.googleusercontent.com/docsz/AD_4nXcxZpJ2o5bU9SQ76W--28IYQ4vbSsq6oS9iJSUGwFriEN_69uluF1aFazaFVcg1SPodECJDYXlAlu5cz1cSkC1rX7cC-rBaL48tGnhJr2onQBYGRB2UwtYGfHMtBbV4MbK1j_H-zQ?key=vfYKUiktMu-Yv0oKyjtkUbdU

8. Click ``Submit`` for the quota to be granted.
9. Do this for all listed CPU/GPU families within the error message.

You can continue with your setup after all CPU/GPU families have the required quota.

.. note::

   Quota request processing time depends on Microsoft and may vary. 
   Approval could take anywhere from a few minutes to several days, or your request might be rejected. If Azure doesn’t grant your requested quotas, see `how to request an increase for non-adjustable quotas <https://learn.microsoft.com/en-us/azure/quotas/per-vm-quota-requests#request-an-increase-for-non-adjustable-quotas>`_.


