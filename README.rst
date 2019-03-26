Tips
========

This cheat sheet will contain shortcuts or tips to Gluu installation and configuration.


Start server and login
----------------------

- # /sbin/gluu-serverd-3.1.5 start
- # /sbin/gluu-serverd-3.1.5 login

Restart services
----------------

- # service httpd restart
- # service identity restart
- # service idp restart
- # service oxauth restart
- # service oxauth-rp restart
- # service passport restart

Certificate Management
----------------------


.. important::

  Default installation comes with self-signed certificate
  
  Better to use a proper certificate from Let's Encrypt
  
  
- Read: https://gluu.org/docs/ce/3.1.5/admin-guide/certificate/

.. code-block:: none

  [root@sso ~]# /sbin/gluu-serverd-3.1.5 login
  [root@localhost certs]# cd /opt/jdk1.8.0_181/jre/lib/security/
  [root@localhost security]# cp cacerts cacerts.SELF-SIGNED

  [root@localhost ~]# cd /etc/certs/
  [root@localhost certs]# cp httpd.crt httpd.crt.SELF-SIGNED


- [root@localhost certs]# cat httpd.csr 
- Copy the content of .csr
- Go to https://zerossl.com/free-ssl/#crt
- Follow instructions from AZLABS WIKI (http://192.168.0.13/wiki/doku.php?id=noc:letsencrypt)
- Rename domain-crt.txt to httpd.crt
- Upload to Gluu server and copy to /opt/gluu-server-3.1.5/etc/certs/


- [root@localhost certs]# cp /root/httpd.crt .
- [root@localhost certs]# openssl x509 -outform der -in httpd.crt -out httpd.der

- [root@localhost certs]# keytool -list -keystore /opt/jdk1.8.0_181/jre/lib/security/cacerts -storepass changeit | grep sso
.. code-block:: java

  sso.azlabs.sg_passport-sp, 25 Mar, 2019, trustedCertEntry, 
  sso.azlabs.sg_idp-signing, 25 Mar, 2019, trustedCertEntry, 
  sso.azlabs.sg_idp-encryption, 25 Mar, 2019, trustedCertEntry, 
  sso.azlabs.sg_asimba, 25 Mar, 2019, trustedCertEntry, 
  sso.azlabs.sg_opendj, 25 Mar, 2019, trustedCertEntry, 
  sso.azlabs.sg_shibidp, 25 Mar, 2019, trustedCertEntry, 
  **sso.azlabs.sg_httpd**, 25 Mar, 2019, trustedCertEntry, 


- [root@localhost certs]# keytool -delete -alias sso.azlabs.sg_httpd -keystore /opt/jdk1.8.0_181/jre/lib/security/cacerts -storepass changeit
- [root@localhost certs]# keytool -importcert -file ./httpd.der -alias sso.azlabs.sg_httpd -keystore /opt/jdk1.8.0_181/jre/lib/security/cacerts -storepass changeit


- [root@localhost certs]# exit
- [root@sso azlabs]# /sbin/gluu-serverd-3.1.5 restart

Configure Reverse Proxy
-----------------------
- [root@localhost]# cd /etc/httpd/conf.d
- [root@localhost conf.d]# cp https_gluu.conf https_gluu.conf.ORIG
- [root@localhost]# vi https_gluu.conf
.. highlight:: html
::

  <Location /ciam>
    ProxyPass http://192.168.1.176:8080/ciam retry=5 connectiontimeout=60 timeout=60
    Order deny,allow
    Allow from all
  </Location>

.. highlight:: none
- # service httpd restart


Support
-------

If you are having issues, please let us know.
We have a mailing list located at: jd@ic.sg

License
-------

The project is licensed under the `MIT License (MIT) <https://github.com/GluuFederation/oxAuth/blob/master/LICENSE>`__.
