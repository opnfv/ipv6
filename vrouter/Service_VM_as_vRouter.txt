==================================
Exercising Service VM as a vRouter
==================================

There are 3 steps to set up a service VM as a vRouter:

- Step 1: `Get a service VM running`_

- Step 2: `Handling Neutron Security Group Feature`_

- Step 3: `Set up an IPv6 vRouter on the Service VM`_

***************************
_`Get a Service VM Running`
***************************

Please click `Set up Service VM`_ page for instructions to get a service VM running.

.. _`Set up Service VM`: ./setup_service_vm.html

******************************************
_`Handling Neutron Security Group Feature` 
******************************************

------------------------------
Disable Security Group Feature
------------------------------

If Open Stack is integrated and running with Open Daylight, we need to completely disable Security Group feature in Open Stack because Open Daylight doesn’t support it.

----------------------------------------------------------
Use Neutron ML2 Port Security Extension (Kilo and Liberty)
----------------------------------------------------------

For Open Stack Kilo or Liberty with ML2 OVS only (without Open Daylight), we need to use Port Security Extension of Neutron and disable Anti-spoofing Rule on the service VM. 

*******************************************
_`Set up an IPv6 vRouter on the Service VM`
*******************************************

Please click `Set up IPv6 vRouter`_ page for instructions to set up an IPv6 vRouter on a Service VM.

.. _`Set up IPv6 vRouter`: ./setup_ipv6_vrouter.html

