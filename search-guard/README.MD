## Instalation of search-quard plugin on Windows OS

### Requirements
* elasticsearch version 5.3.1
* Windows OS
* powershell
* openssl

### Download plugin
At first you have to download plugin from elasticserch repository. Before installation, check if you have correct settings of maven to access central repository.
You have to open powershell pass the following command:

    PS> <ELASTICSEARCH-HOME>/bin/elasticsearch-plugin install -b com.floragunn:search-guard:5-5.3.1-15
    
After instalattion the following warning will be display. But do not worry. In installation command you passed the argument **-b** which means that you automatic confirm of security permissions
    
    [=================================================] 100%  
    @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
    @     WARNING: plugin requires additional permissions     @
    @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
    * java.io.FilePermission /proc/sys/net/core/somaxconn read
    * java.lang.RuntimePermission accessClassInPackage.sun.misc
    * java.lang.RuntimePermission accessClassInPackage.sun.nio.ch
    * java.lang.RuntimePermission accessClassInPackage.sun.security.x509
    * java.lang.RuntimePermission accessDeclaredMembers
    * java.lang.RuntimePermission getClassLoader
    * java.lang.RuntimePermission loadLibrary.*
    * java.lang.RuntimePermission setContextClassLoader
    * java.lang.RuntimePermission shutdownHooks
    * java.lang.reflect.ReflectPermission suppressAccessChecks
    * java.security.SecurityPermission getProperty.ssl.KeyManagerFactory.algorithm
    * java.util.PropertyPermission java.security.debug write
    * java.util.PropertyPermission java.security.krb5.conf write
    * java.util.PropertyPermission javax.security.auth.useSubjectCredsOnly write
    * java.util.PropertyPermission sun.nio.ch.bugLevel write
    * java.util.PropertyPermission sun.security.krb5.debug write
    * java.util.PropertyPermission sun.security.spnego.debug write
    * javax.security.auth.AuthPermission doAs
    * javax.security.auth.AuthPermission modifyPrivateCredentials
    * javax.security.auth.kerberos.ServicePermission * accept
    See http://docs.oracle.com/javase/8/docs/technotes/guides/security/permissions.html
    
Now the plugin is download in your elasticsearch and you can continue

### Create all associated certificates

There are following certificate which you need to create because of searchquard:
* root-ca.pem— the root CA used for signing all other certificates
* esnode.pem— the node certificate used on the transport- and REST-layer.
* esnode-key.pem— the private key for the node certificate
* kirk.pem— the admin certificate, allows full access to the cluster and can be used with sgadmin and the REST management API
* kirk-key.pem— the private key for the admin certificate

Let's create them and add them to particular certificate's stores.

#### root-ca.pem
This certificate is mostly provided by customers or third parties. But if you need to create it then follow the following steps:

1. Create the certification folder and set your path location(for example user/<USER_NAME>/certification-test)
    PS> mkdir C:\Users\<USER_NAME>\certificate-test\root-ca
    PS> mkdir C:\Users\<USER_NAME>\certificate-test\root-ca\db
    PS> mkdir C:\Users\<USER_NAME>\certificate-test\root-ca\private
    
2. Create files .db and .attr
    PS> New-Item C:\Users\<USER_NAME>\certificate-test\root-ca\db\root-ca.db -type file
    PS> New-Item C:\Users\<USER_NAME>\certificate-test\root-ca\db\root-ca.db.attr -type file


