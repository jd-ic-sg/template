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


Install Memcached
-----------------
- [root@localhost]# yum clean all
- [root@localhost]# yum -y update
- [root@localhost]# yum -y install memcached

.. highlight:: html
::

  TO DO: Document down how to integre with Gluu https://www.liquidweb.com/kb/how-to-install-memcached-on-centos-7/

.. highlight:: none



Tuning
------

- Location: /etc/default

.. code-block:: none

  [root@localhost default]# ls
  identity  idp  nss  oxauth  oxauth-rp  passport



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


.. code-block:: none

  [root@localhost certs]# cat httpd.csr 
  
  
- Copy the content of .csr
- Go to https://zerossl.com/free-ssl/#crt
- Follow instructions from AZLABS WIKI (http://192.168.0.13/wiki/doku.php?id=noc:letsencrypt)
- Rename domain-crt.txt to httpd.crt
- Upload to Gluu server and copy to /opt/gluu-server-3.1.5/etc/certs/


.. code-block:: none

  [root@localhost certs]# cp /root/httpd.crt .
  [root@localhost certs]# openssl x509 -outform der -in httpd.crt -out httpd.der

  [root@localhost certs]# keytool -list -keystore /opt/jdk1.8.0_181/jre/lib/security/cacerts -storepass changeit | grep sso
  
  
.. code-block:: java

  sso.azlabs.sg_passport-sp, 25 Mar, 2019, trustedCertEntry, 
  sso.azlabs.sg_idp-signing, 25 Mar, 2019, trustedCertEntry, 
  sso.azlabs.sg_idp-encryption, 25 Mar, 2019, trustedCertEntry, 
  sso.azlabs.sg_asimba, 25 Mar, 2019, trustedCertEntry, 
  sso.azlabs.sg_opendj, 25 Mar, 2019, trustedCertEntry, 
  sso.azlabs.sg_shibidp, 25 Mar, 2019, trustedCertEntry, 
  **sso.azlabs.sg_httpd**, 25 Mar, 2019, trustedCertEntry, 


.. code-block:: none

  [root@localhost certs]# keytool -delete -alias sso.azlabs.sg_httpd -keystore /opt/jdk1.8.0_181/jre/lib/security/cacerts -storepass changeit
  [root@localhost certs]# keytool -importcert -file ./httpd.der -alias sso.azlabs.sg_httpd -keystore /opt/jdk1.8.0_181/jre/lib/security/cacerts -storepass changeit


.. code-block:: none

  [root@localhost certs]# exit
  [root@sso azlabs]# /sbin/gluu-serverd-3.1.5 restart



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


Upgrade from 3.1.x to 3.1.6
---------------------------

.. code-block:: none

  [root@localhost ~]# wget https://repo.gluu.org/upd/3-1-6-upg.sh
  [root@localhost ~]# sh 3-1-6-upg.sh 
    Creating directory /opt/upd/3.1.6upg/
    Verifying archive integrity...  100%   MD5 checksums are OK. All good.

    Installed:
      python-ldap.x86_64 0:2.4.15-2.el7                                   python2-jsonschema.noarch 0:2.5.1-3.el7                                  
    Dependency Installed:
      python-repoze-lru.noarch 0:0.4-3.el7                                                                                                          
    Complete!
    Restarting program
    Starting upgrade. CONTINUE? (y|N): y

    Would you like to replace all the default Gluu Server scripts WITH SCRIPTS FROM 3.1.6?
    (This will replace any customization you may have made to these default script entries) (Y|n)


.. code-block:: none

    Starting Upgrade...
    Current Gluu Server version 3.1.5
    Stopping Jetty: OK
    Stopping Jetty: OK
    Updating ldap schema
    Stopping LDAP Server
    Stopping OpenDJ
    Executing /etc/init.d/opendj stop
    [01/Apr/2019:21:24:49 +0800] category=PLUGGABLE severity=NOTICE msgID=org.opends.messages.backend.370 msg=The backend metric is now taken offline
    [01/Apr/2019:21:24:49 +0800] category=PLUGGABLE severity=NOTICE msgID=org.opends.messages.backend.370 msg=The backend site is now taken offline
    [01/Apr/2019:21:24:50 +0800] category=PLUGGABLE severity=NOTICE msgID=org.opends.messages.backend.370 msg=The backend userRoot is now taken offline
    [01/Apr/2019:21:24:50 +0800] category=CORE severity=NOTICE msgID=org.opends.messages.core.203 msg=The Directory Server is now stopped
    /opt/opendj/config/schema/101-ox.ldif
    Backing up /opt/opendj/config/schema/101-ox.ldif
    Copying new_schema /opt/upd/3.1.6upg/ldap/opendj/101-ox.ldif
    Copying new_schema /opt/upd/3.1.6upg/ldap/opendj/96-eduperson.ldif
    Starting LDAP Server
    Starting OpenDJ
    Executing /etc/init.d/opendj start
    oxAuthLogoutURI modified
    oxAuthPostLogoutRedirectURI modified
    Backing up current scripts
    Deleting current script inum=@!4CDC.D57C.C87D.1D6D!0001!1F07.55B8!2124.0CF1,ou=scripts,o=@!4CDC.D57C.C87D.1D6D!0001!1F07.55B8,o=gluu
    Adding new script inum=@!4CDC.D57C.C87D.1D6D!0001!1F07.55B8!2124.0CF1,ou=scripts,o=@!4CDC.D57C.C87D.1D6D!0001!1F07.55B8,o=gluu
    :
    :
    Backing up oxauth.war to /opt/upd/3.1.6upg/backup_2019-04-01.21:24:27
    Updating oxauth.war
    Backing up identity.war to /opt/upd/3.1.6upg/backup_2019-04-01.21:24:27
    Updating identity.war
    Backing up idp.war to /opt/upd/3.1.6upg/backup_2019-04-01.21:24:27
    Updating idp.war
    checking /opt/shibboleth-idp/metadata/idp-metadata.xml
    Updating jetty
    chown: cannot access ‘/opt/jetty-9.4/temp/jetty-localhost-8086-idp.war-_idp-any-5944512476372526077.dir’: No such file or directory
    Updating Passport
    Stopping passport: OK
    tar: Removing leading `/' from member names
    Extracting passport.tgz into /opt/gluu/node/passport
    Extracting passport node modules
    oxAuthenticationMode was set to auth_ldap_server
    oxTrustAuthenticationMode was set to auth_ldap_server
    oxCacheConfiguration was modified as {"cacheProviderType": "IN_MEMORY", "nativePersistenceConfiguration": {"defaultPutExpiration": 60}, "redisConfiguration": {"useSSL": false, "defaultPutExpiration": 60, "servers": "localhost:6379", "sslTrustStoreFilePath": "", "decryptedPassword": null, "password": null, "redisProviderType": "STANDALONE"}, "memcachedConfiguration": {"servers": "localhost:11211", "defaultPutExpiration": 60, "bufferSize": 32768, "maxOperationQueueLength": 100000, "connectionFactoryType": "DEFAULT"}, "inMemoryConfiguration": {"defaultPutExpiration": 60}}
    Updating oxAuthConfDynamic
    Updating oxTrustConfApplication
    Updating oxAuthConfErrors
    Backing up /opt/shibboleth-idp to /opt/upd/3.1.6upg/backup_2019-04-01.21:24:27
    Updating idp-metadata.xml
    Updadting shibboleth-idp


  Please Note: oxAuthenticationMode and oxTrustAuthenticationMode was
  set to auth_ldap_server in case custom authentication script fails.
  Please review your scripts and adjust default authentication method

  Update is complete, please exit from container and restart gluu server


.. code-block:: none

  [root@localhost ~]# exit
  logout
  [root@sso azlabs]# /sbin/gluu-serverd-3.1.5 restart


.. important::

  Scripts and directories outside the Chroot will still reflect the version from which you upgraded. For example, if you started with version 3.1.3, the directory will still be gluu-server-3.1.3 even after upgrading to 3.1.6.
  
.. important::

  It is good to maintain a README.LATEST manually
  
  [root@sso]# cd /opt

  [root@sso opt]# cat README.LATEST 
  
  Current Gluu Server version 3.1.5
  
  Current Gluu Server version 3.1.6 <-- 1.APR.2019



Support
-------

If you are having issues, please let us know.
We have a mailing list located at: jd@ic.sg

License
-------

The project is licensed under the `MIT License (MIT) <https://github.com/GluuFederation/oxAuth/blob/master/LICENSE>`__.
