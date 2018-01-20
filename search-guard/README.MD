## Instalation of search-quard plugin on Windows OS

### Requirements
* elasticsearch version 5.3.1
* Windows OS
* powershell
* openssl
* keytool
* root certificate (if not exists then generate)
* root public key. If you alerady have root certificate then extract key from them with following command

### Download plugin
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
* esnode.pem — the node certificate used on the transport- and REST-layer.
* esnode-key.pem — the private key for the node certificate
* kirk.pem — the admin certificate, allows full access to the cluster and can be used with sgadmin and the REST management API
* kirk-key.pem — the private key for the admin certificate

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

#### node certificate
Generate the certificate using domain csr and key along with the CA Root key. For example for *node1* with pass *pass123*

1. Generate .jks
```
# pattern:

keytool -genkey -alias <ALIAS_KEYSTORE> -keystore <NODE_NAME>-keystore.jks -keyalg RSA -keysize 2048 -validity 712 -sigalg SHA256withRSA -keypass <PASS> -storepass <PASS> -dname "CN=<CERTIFICATE_NAME>, OU=SSL, O=<ORGANIZATION>, L=Prague, C=CZ" -ext san=dns:<DNS_NAME>

# sample:

keytool -genkey -alias node1 -keystore node1-keystore.jks -keyalg RSA -keysize 2048 -validity 712 -sigalg SHA256withRSA -keypass pass123 -storepass pass123 -dname "CN=node1.example.com, OU=SSL, O=Test, L=Test, C=DE" -ext san=dns:localhost
```

2. Generating certificate signing request for node1 - create .csr domain file
```
# pattern:

keytool -certreq -alias <ALIAS_KEYSTORE> -keystore <NODE_NAME>-keystore.jks -file <CSR_FILE>.csr -keyalg rsa -keypass <PASS> -storepass <PASS> -dname "CN=<CERTIFICATE_NAME>, OU=SSL, O=<ORGANIZATION>, L=Prague, C=CZ" -ext san=dns:<DNS_NAME>

# sample:

keytool -certreq -alias node1 -keystore node1-keystore.jks -file node1.csr -keyalg rsa -keypass pass123 -storepass pass123 -dname "CN=node1.example.com, OU=SSL, O=Test, L=Test, C=DE" -ext san=dns:localhost
```

3. Sign certificate request by CA
```
# pattern:

PS> openssl x509 -req -in <CSR_FILE>.csr -CA <ROOT_CRT>.crt -CAkey <ROOT_PRIVATE_KEY>.key -CAcreateserial -out <OUT_CRT>.crt -days 500 -sha256

# sample:

PS> openssl x509 -req -in node1.csr -CA rootCA.crt -CAkey rootCA.key -CAcreateserial -out node1.crt -days 500 -sha256
```

4. Import certificate back to keystore
```
# pattern:
PS> keytool -import -file <IMPORTED_CERT> -alias <ALIAS> -keystore <DEST_KEYSTORE> -storepass <PASS>

# sample:

PS> keytool -import -file node1.crt -alias node1 -keystore node1-keystore.jks -storepass pass123
```


