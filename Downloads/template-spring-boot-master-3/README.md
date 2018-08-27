# SpringBoot project template

This project provides a template to quickly start a new SpringBoot project that comes pre-configured
with a number of useful features. It is intended to be customizable, extensible, and to be updated as
the underlying template is updated.

## Getting started

### Create the project

#### Clone the project

To get started clone this repo into a local folder, preferably one with a different name:

```bash
git clone --single-branch {url} {new_name}
```

**OR**

#### Create the toolchain

Create the toolchain and clone the project into a new repository. There are three different destinations for 
repositories in the IBM toolchain so there are three different branches/buttons. Click the appropriate link below
then click the subsequent 'Create toolchain'.

* [GitHub](tree/toolchain_github)
* [IBM Hosted GitLab](tree/toolchain_gitlab)
* [GitHub Enterprise Whitewater](tree/toolchain_ibmghe)


### Set up the project from the template

Next, run the `setup-template.sh` script. It will ask for you project name and the url to your new 
repo. If no project name is provided the script will default to using the folder into which the repo 
was checked out.

```bash
./setup-template.sh
```

### (Optionally) Setup the IBM Cloud Continuous Delivery pipeline

Customize the toolchain config by running the `setup-bluemix-toolchain.sh` script. It will ask several
questions related to the project repository and name. When finished, a new version of `.bluemix/toochain.yml`
will have been generated and should be pushed to the repo.

Next, navigate to the `PIPELINE.md` file from the repo url and click
the `Create toolchain` button.


### Periodically update from the template

Finally, the template components can be periodically updated by running the following:

```bash
./update-template.sh
```

## More information

Once the `setup-template.sh` script has been run, this README.md file will be replaced with a new 
file for the project. More information about the template features and how they are used can be 
found in [TEMPLATE.md](TEMPLATE.md)

# Mutual Auth How-To
* ###Overview	
* ###Modify the Application to require Security	
* ###Keytool commands	
* ###Modify the application.yml	
* ###Reference Links	

## Overview

####Set up Mutual Authentication for client to service endpoints

##### High Level Interaction between Client and Server
![High Level Interaction for Mutual Authentication](https://www.opencodez.com/wp-content/uploads/2017/08/2-Way-Authentication.png)


####Enable the gradle dependency for security

Modify build.grade (top level file)
Add the gradle dependency line like this:
```
apply from:   'gradle/build-security.gradle'
```

Then, in the “gradel” directory, add a new file called “build-security.gradle”
```yaml
dependencies {
   compile group: 'org.springframework.boot', name: 'spring-boot-starter-security'
}
```
Remember to import changes

####Modify the Application to require Security


Example in the Application java class (“src/main/java/com.ibm/app/hello/app/application.java)

Add the following annotation to the application class
```
@EnableWebSecurity

```
####Modify the application.yml

Add the following lines to your application.yml (src/main/java/resources)
```yaml
spring:
  application:
    name: "template-spring-boot"

  security:
    user:
      name: admin
      password: admin


server:
  port: 9080
  ssl:
    enabled: true
    trust-store: config/MyServer.jks
    trust-store-password: password
    key-store: config/MyServer.jks
    key-store-password: password


```

#### Generate a jks file for Self Signed Certs on client and server

#### Generate a server keystore
* ##### keytool -genkey -alias MyServer -keyalg RSA -validity 1825 -keystore "MyServer.jks" 
-storetype JKS 

Gives this dialog
```
Enter keystore password: 
Re-enter new password: 
What is your first and last name?
  [Unknown]:  
What is the name of your organizational unit?
  [Unknown]:  
What is the name of your organization?
  [Unknown]:  
What is the name of your City or Locality?
  [Unknown]:  
What is the name of your State or Province?
  [Unknown]:  
What is the two-letter country code for this unit?
  [Unknown]:  
Is CN=Unknown, OU=Unknown, O=Unknown, L=Unknown, ST=Unknown, C=Unknown correct?
  [no]:  yes
Enter key password for <MyServer>
	(RETURN if same as keystore password):  
Re-enter new password: 
```
####Export the server certificate

* ##### keytool -exportcert -alias MyServer -keystore MyServer.jks -file MyServer.cer


Dialog given

```
Enter keystore password:  
Certificate stored in file <MyServer.cer>

Warning:
The JKS keystore uses a proprietary format. It is recommended to migrate to PKCS12 which is an industry standard format using "keytool -importkeystore -srckeystore MyServer.jks -destkeystore MyServer.jks -deststoretype pkcs12".

```
#### Generate a client keystore

* #####keytool -genkey -alias MyClient -keyalg RSA -validity 1825 -keystore MyClient.jks -storetype JKS 

Dialog given
```
Enter keystore password:  
Re-enter new password: 
What is your first and last name?
  [Unknown]:  
What is the name of your organizational unit?
  [Unknown]:  
What is the name of your organization?
  [Unknown]:  
What is the name of your City or Locality?
  [Unknown]:  
What is the name of your State or Province?
  [Unknown]:  
What is the two-letter country code for this unit?
  [Unknown]:  
Is CN=Unknown, OU=Unknown, O=Unknown, L=Unknown, ST=Unknown, C=Unknown correct?
  [no]:  yes

Enter key password for <MyClient>
	(RETURN if same as keystore password):  
Re-enter new password: 

Warning:
The JKS keystore uses a proprietary format. It is recommended to migrate to PKCS12 which is an industry standard format using "keytool -importkeystore -srckeystore MyClient.jks -destkeystore MyClient.jks -deststoretype pkcs12".
```
####Export the client cert
* #####keytool -exportcert -alias MyClient -keystore MyClient.jks -file MyClientPublic.cer

Dialog given
```
Enter keystore password:  
Certificate stored in file <MyClientPublic.cer>

Warning:
The JKS keystore uses a proprietary format. It is recommended to migrate to PKCS12 which is an industry standard format using "keytool -importkeystore -srckeystore MyClient.jks -destkeystore MyClient.jks -deststoretype pkcs12".
```
####Add the certs to the keystores
###Add Server certificate to client truststore
* #####keytool -importcert -alias MyServer -keystore MyClient.jks -file MyServer.cer
```Enter keystore password:  
   Owner: CN=Unknown, OU=Unknown, O=Unknown, L=Unknown, ST=Unknown, C=Unknown
   Issuer: CN=Unknown, OU=Unknown, O=Unknown, L=Unknown, ST=Unknown, C=Unknown
   Serial number: 7376051d
   Valid from: Wed Aug 22 15:50:48 EDT 2018 until: Mon Aug 21 15:50:48 EDT 2023
   Certificate fingerprints:
   	 MD5:  75:21:26:DA:A7:AC:50:FD:29:80:CF:EC:2F:59:A9:41
   	 SHA1: B8:78:8C:1A:3C:29:6E:89:29:A2:AE:13:62:A2:E6:3D:9C:56:21:3E
   	 SHA256: B1:F2:82:12:9A:87:F7:59:10:26:5B:24:2A:35:C1:6E:26:AA:0F:2A:9F:F5:57:30:85:6C:DF:96:A7:0A:F9:6E
   Signature algorithm name: SHA256withRSA
   Subject Public Key Algorithm: 2048-bit RSA key
   Version: 3
   
   Extensions: 
   1: ObjectId: 2.5.29.14 Criticality=false
   SubjectKeyIdentifier [
   KeyIdentifier [
   0000: 2B 48 9D 17 E9 88 E2 07   02 AC 4F 19 D1 62 A3 5E  +H........O..b.^
   0010: C5 CC E2 96                                        ....
   ]
   ]
   
   Trust this certificate? [no]:  yes
   Certificate was added to keystore
   
   Warning:
   The JKS keystore uses a proprietary format. It is recommended to migrate to PKCS12 which is an industry standard format using "keytool -importkeystore -srckeystore MyClient.jks -destkeystore MyClient.jks -deststoretype pkcs12". 
  ```
####Add client certificate to server truststore
* #####keytool -importcert -alias MyClient -keystore MyServer.jks -file MyClientPublic.cer
```Enter keystore password:  
   Owner: CN=Unknown, OU=Unknown, O=Unknown, L=Unknown, ST=Unknown, C=Unknown
   Issuer: CN=Unknown, OU=Unknown, O=Unknown, L=Unknown, ST=Unknown, C=Unknown
   Serial number: ba67e08
   Valid from: Wed Aug 22 15:54:43 EDT 2018 until: Mon Aug 21 15:54:43 EDT 2023
   Certificate fingerprints:
   	 MD5:  54:F1:06:6F:13:49:6D:39:75:D4:11:CA:BB:CF:F9:AD
   	 SHA1: 7B:CC:A7:3C:F1:62:4D:77:4C:90:17:74:A4:AB:F0:D7:19:2E:42:99
   	 SHA256: C2:42:30:D7:14:BE:72:11:BF:AF:4F:8C:9E:0D:97:48:11:57:B8:D4:5E:61:7B:D4:9F:EE:88:5B:88:AE:49:09
   Signature algorithm name: SHA256withRSA
   Subject Public Key Algorithm: 2048-bit RSA key
   Version: 3
   
   Extensions: 
   
   1: ObjectId: 2.5.29.14 Criticality=false
   SubjectKeyIdentifier [
   KeyIdentifier [
   0000: CC 15 DB FC C0 75 A1 15   42 92 9D 04 98 B2 9B AB  .....u..B.......
   0010: 48 B7 1A 89                                        H...
   ]
   ]
   
   Trust this certificate? [no]:  yes
   Certificate was added to keystore
   
   Warning:
   The JKS keystore uses a proprietary format. It is recommended to migrate to PKCS12 which is an industry standard format using "keytool -importkeystore -srckeystore MyServer.jks -destkeystore MyServer.jks -deststoretype pkcs12".
```

##Client Code

####Mutual Authentication

```
@SpringBootApplication

public class MutualAuthenticationClientApplication {
    static
    {
        System.setProperty("javax.net.debug", "all");
        System.setProperty("jdk.tls.client.protocols", "TLSv1.2");
        System.setProperty("https.protocols", "TLSv1.2");
        System.setProperty("javax.net.ssl.trustStore", "config/MyClient.jks");
        System.setProperty("javax.net.ssl.trustStorePassword", "password");
        System.setProperty("javax.net.ssl.keyStore",  "config/MyClient.jks");
        System.setProperty("javax.net.ssl.keyStorePassword", "password");    ```

javax.net.ssl.HttpsURLConnection.setDefaultHostnameVerifier(
                new javax.net.ssl.HostnameVerifier() {

                    public boolean verify(String hostname,
                                          javax.net.ssl.SSLSession sslSession) {
                        if (hostname.equals("localhost")) {
                            return true;
                        }
                        return false;
                    }
                });
    }

    @Bean
    public RestTemplate template() throws Exception{
        RestTemplate template = new RestTemplate();
        return template;
    }
    public static void main(String[] args) {
        SpringApplication.run(MutualAuthenticationClientApplication.class, args);
    }
}

```


* #### HttpClient.java implements CommandLineRunner

```java
@Component

public class HttpClient implements CommandLineRunner {
   @Autowired
   private RestTemplate template;

   @Override
   public void run(String... args) throws Exception {
       ResponseEntity<String> response = template.getForEntity("https://localhost:9080/hello",
               String.class);
       System.out.println(response.getBody());
   }
}
```

#Reference Links

[How to](https://www.naschenweng.info/2018/02/01/java-mutual-ssl-authentication-2-way-ssl-authentication/)

[Springboot Mutual Auth](https://www.opencodez.com/java/implement-2-way-authentication-using-ssl.htm)

[SSL In Liberty](https://www.ibm.com/support/knowledgecenter/en/SSEQTP_liberty/com.ibm.websphere.wlp.doc/ae/twlp_sec_ssl.html)

[Basic auth in Liberty](https://www.ibm.com/support/knowledgecenter/SSD28V_9.0.0/com.ibm.websphere.wlp.express.doc/ae/twlp_sec_cache.html)

[Mutual Auth in Liberty](https://www.baeldung.com/x-509-authentication-in-spring-security)

[Keytool commands](https://asardana.com/2016/10/11/spring-boot-mutual-authentication-2-way-ssltls/)

[Example](https://www.codenotfound.com/spring-ws-mutual-authentication-example.html)




