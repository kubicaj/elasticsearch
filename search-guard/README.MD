## Instalation of search-quard plugin on Windows OS

### Requirements
* elasticsearch version 5.3.1
* Windows OS

### Download plugin
At first you have to download plugin from elasticserch repository. Before installation, check if you have correct settings of maven to access central repository.
You have to pass the following command:

    $> <ELASTICSEARCH-HOME>/bin/elasticsearch-plugin install -b com.floragunn:search-guard:5-5.3.1-15
    
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



