============
Installation
============

1. VM Preparation

    * https://gluu.org/docs/ce/3.1.5/installation-guide/
    
.. important::

   Tuning file descriptors is very important as the underlying OpenDJ requires this to run efficiently
    

.. important::

   FQDN is a must. localhost is not supported


2. Installation

    * Follow CentOS 7.x instruction
    
    * # cd /install/community-edition-setup
    
    * # ./setup.py

    * # /sbin/gluu-serverd-3.1.5 enable (once only, right after installation)


3. Sign in via web browser

.. tip::

   Wait about 10 mins in total for server to restart and finalize its configuration
   
   
    * https://sso.azlabs.sg
    
