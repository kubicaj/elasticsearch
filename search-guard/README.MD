# Instalation of search-quard plugin on Windows OS

### Requirements
* elasticsearch version 5.3.1
* Windows OS
* powershell
* openssl
* keytool
* root certificate (if not exists then generate)
* root private key for sign the node certificate
* intermediate certificate

## Download plugin
At first you have to download plugin from elasticserch repository. Before installation, check if you have correct settings of maven to access central repository.
You have to open powershell pass the following command:

```
    PS> <ELASTICSEARCH-HOME>/bin/elasticsearch-plugin install -b com.floragunn:search-guard:5-5.3.1-15
```   

After instalattion the following warning will be display. But do not worry. In installation command you passed the argument **-b** which means that you automatic confirm of security permissions

```   
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

```    
Now the plugin is download in your elasticsearch and you can continue

### Create all associated certificates

There are following certificate which you need to create because of searchquard:
* the root CA used for signing all other certificates
* the node certificate used on the transport- and REST-layer.
* the admin certificate, allows full access to the cluster and can be used with sgadmin and the REST management API

Let's create them and add them to particular certificate's stores.

#### root certificat
This certificate is mostly provided by customers or third parties. But if you need to create it then follow the following steps:
   
1. Create root key. Pass the following command and follow instruction in command line

```
    PS> openssl genrsa -des3 -out rootCA.key 2048
```

2. Create self sign the Root Certificate. Pass the following command and follow instruction in command line

```
PS> openssl req -x509 -new -nodes -key rootCA.key -sha256 -days 1024 -out rootCA.crt
```

#### root certificate trustore

```
PS> keytool -import -file rootCA.crt -alias rootCa -keystore truststore.jks
```

#### node certificate
Generate the certificate using domain csr and key along with the CA Root key. For example for *node1* with pass *pass123*

1. Generate .jks
```
# pattern:

PS> keytool -genkey -alias <ALIAS_KEYSTORE> -keystore <NODE_NAME>-keystore.jks -keyalg RSA -keysize 2048 -validity 712 -sigalg SHA256withRSA -keypass <PASS> -storepass <PASS> -dname "CN=<CERTIFICATE_NAME>, OU=SSL, O=<ORGANIZATION>, L=Prague, C=CZ" -ext san=dns:<DNS_NAME>

# sample:

PS> keytool -genkey -alias node1 -keystore node1-keystore.jks -keyalg RSA -keysize 2048 -validity 712 -sigalg SHA256withRSA -keypass pass123 -storepass pass123 -dname "CN=node1, OU=Org, O=Org, L=Prague, C=CZ, ST=Czech Republic" -ext san=dns:localhost
```

2. Generating certificate signing request for node1 - create .csr domain file
```
# pattern:

PS> keytool -certreq -alias <ALIAS_KEYSTORE> -keystore <NODE_NAME>-keystore.jks -file <CSR_FILE>.csr -keyalg rsa -keypass <PASS> -storepass <PASS> -dname "CN=<CERTIFICATE_NAME>, OU=SSL, O=<ORGANIZATION>, L=Prague, C=CZ" -ext san=dns:<DNS_NAME>

# sample:

PS> keytool -certreq -alias node1 -keystore node1-keystore.jks -file node1.csr -keyalg rsa -keypass pass123 -storepass pass123 -dname "CN=node1, OU=Org, O=Org, L=Prague, C=CZ, ST=Czech Republic" -ext san=dns:localhost
```

3. Sign certificate request by CA
```
# pattern:

PS> openssl x509 -req -in <CSR_FILE>.csr -CA <ROOT_CRT>.crt -CAkey <ROOT_PRIVATE_KEY>.key -CAcreateserial -out <OUT_CRT>.crt -days 500 -sha256

# sample:

PS> openssl x509 -req -in node1.csr -CA rootCA.crt -CAkey rootCA.key -CAcreateserial -out node1.crt -days 500 -sha256
```

4. Import certificate back to keystore. At first import trusted certificate and then reimport client certificate
```
# pattern (two commands which are the same):

PS> keytool -import -file <IMPORTED_CERT> -alias <ALIAS> -keystore <DEST_KEYSTORE> -storepass <PASS> -trustcacerts -file <TRSUSTSTORE>
PS> keytool -import -file <IMPORTED_CERT> -alias <ALIAS> -keystore <DEST_KEYSTORE> -storepass <PASS> -trustcacerts -file <TRSUSTSTORE>

# sample:

PS> keytool -import -keystore node1-keystore.jks -file rootCA.crt -alias theCARoot -storepass pass123
PS> keytool -import -keystore node1-keystore.jks -file node1.crt -alias node1 -storepass pass123
```

#### admin certificate
The process if the same as for node certificate. The admin cetificate named krik.

Generate the certificate using domain csr and key along with the CA Root key. For example for *krik* with pass *pass123*

1. Generate .jks
```
# pattern:

PS> keytool -genkey -alias <ALIAS_KEYSTORE> -keystore krik.jks -keyalg RSA -keysize 2048 -validity 712 -sigalg SHA256withRSA -keypass <PASS> -storepass <PASS> -dname "CN=<CERTIFICATE_NAME>, OU=SSL, O=<ORGANIZATION>, L=Prague, C=CZ" -ext san=dns:<DNS_NAME>

# sample:

PS> keytool -genkey -alias krik -keystore krik.jks -keyalg RSA -keysize 2048 -validity 712 -sigalg SHA256withRSA -keypass pass123 -storepass pass123 -dname "CN=krik, OU=Org, O=Org, L=Prague, C=CZ, ST=Czech Republic" -ext san=dns:localhost
```

2. Generating certificate signing request for node1 - create .csr domain file
```
# pattern:

PS> keytool -certreq -alias krik -keystore krik.jks -file <CSR_FILE>.csr -keyalg rsa -keypass <PASS> -storepass <PASS> -dname "CN=<CERTIFICATE_NAME>, OU=SSL, O=<ORGANIZATION>, L=Prague, C=CZ" -ext san=dns:<DNS_NAME>

# sample:

PS> keytool -certreq -alias krik -keystore krik.jks -file krik.csr -keyalg rsa -keypass pass123 -storepass pass123 -dname "CN=krik, OU=Org, O=Org, L=Prague, C=CZ, ST=Czech Republic" -ext san=dns:localhost
```

3. Sign certificate request by CA
```
# pattern:

PS> openssl x509 -req -in <CSR_FILE>.csr -CA <ROOT_CRT>.crt -CAkey <ROOT_PRIVATE_KEY>.key -CAcreateserial -out <OUT_CRT>.crt -days 500 -sha256

# sample:

PS> openssl x509 -req -in krik.csr -CA rootCA.crt -CAkey rootCA.key -CAcreateserial -out krik.crt -days 500 -sha256
```

4. Import certificate back to keystore. At first import trusted certificate and then reimport client certificate
```
# pattern (two commands which are the same):

PS> keytool -import -file <IMPORTED_CERT> -alias <ALIAS> -keystore krik.jks -storepass <PASS> -trustcacerts -file <TRSUSTSTORE>
PS> keytool -import -file <IMPORTED_CERT> -alias <ALIAS> -keystore krik.jks -storepass <PASS> -trustcacerts -file <TRSUSTSTORE>

# sample:

PS> keytool -import -keystore krik.jks -file rootCA.crt -alias theCARoot -storepass pass123
PS> keytool -import -keystore krik.jks -file krik.crt -alias krik -storepass pass123
```

## Initialization of environment

### Copy certificates

1. The first you have to add all certificates into config folder of ES. The path is following: <ES>\config\
2. Create the empty *scripts* in configuration folder of ES. You can use the following command
    
```
PS> mkdir <ES_HOME>\elasticsearch\config\scripts     
```    

Then the content of config file has to looks like this:
 

    Directory: <ES_HOME>\elasticsearch\config

```
Mode                LastWriteTime     Length Name
----                -------------     ------ ----
d----         20.1.2018     11:41            scripts
-a---         20.1.2018     20:40       3753 elasticsearch.yml
-a---         17.4.2017     15:50       3117 jvm.options
-a---         20.1.2018     20:19       4262 krik.jks
-a---         17.4.2017     15:50       4456 log4j2.properties
-a---         20.1.2018     19:19       2245 node1-keystore.jks
-a---         20.1.2018     19:19       1060 truststore.jks
```

### Update config file
Into *<ES_HOME>\elasticsearch\config\elasticsearch.yml*  add the following lines (Edit as you need)
```
############ The settings of transport protocol ############
# Relative path to the keystore file
searchguard.ssl.transport.keystore_filepath: node1-keystore.jks
# Relative path to the truststore file
searchguard.ssl.transport.truststore_filepath: truststore.jks
# Enforce hostname verification (default: true). If true then the certification domain is check as well
#searchguard.ssl.transport.enforce_hostname_verification: true
# keystore pass
searchguard.ssl.transport.keystore_password: pass123
# trsutstore pass
searchguard.ssl.transport.truststore_password: pass123

############ The settings of http protocol ############
# enable/disable http interface
searchguard.ssl.http.enabled: true
# Relative path to the keystore file
searchguard.ssl.http.keystore_filepath: node1-keystore.jks
# Relative path to the truststore file
searchguard.ssl.http.truststore_filepath: truststore.jks
# keystore pass
searchguard.ssl.http.keystore_password: pass123
# trsutstore pass
searchguard.ssl.http.truststore_password: pass123

# the kirk certification field
searchguard.authcz.admin_dn:
  - CN=kirk,OU=Org,O=Org,L=Prague, C=CZ, ST=Czech Republic 
```

