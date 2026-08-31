# 📘 MuleSoft Secure Properties

This document provides a concise overview of how Uprizon Mule applications encrypt and manage sensitive configuration values using MuleSoft Secure Configuration Properties.

📌 Note: Examples reference the sample application sf-case-api-impl for clarity.

## 1. Purpose of Secure Properties
MuleSoft Secure Configuration Properties allow you to **encrypt sensitive values** (passwords, tokens, private keys) inside `.yaml` or `.properties` files and **decrypt them automatically at runtime**.

This ensures:
* No plaintext secrets in your Mule code
* No secrets in version control
* Consistent encryption across environments

## 2. Required Maven Dependency
Add the Secure Properties module to your Mule application:

```xml
<dependency>
    <groupId>com.mulesoft.modules</groupId>
    <artifactId>mule-secure-configuration-property-module</artifactId>
    <version>1.3.0</version>
    <classifier>mule-plugin</classifier>
</dependency>
```
📌 Role: Enables Mule Runtime to decrypt values stored in secure-properties.yaml.

## 3. Decryption Key (`mule.key`)
Mule requires a decryption key to read encrypted values.

### CloudHub / CI/CD / Runtime
Pass the key as a JVM parameter:

```Code
-M-Dmule.key=xxxxxx
```
### Local Development
Provide a default key via global property:

```xml
<global-property 
    name="mule.key" 
    value="CHANGE-ME-16-KEY"
    doc:name="Default dev encryption key (16 characters for AES)" />
```
📌 **AES key length**: 16, 24, or 32 characters    
📌 **Blowfish**: variable length

## 4. Secure Properties Configuration
Declare the secure properties configuration in your Mule app:

```xml
<secure-properties:config 
    name="secure-properties-config"
    file="secure-properties.yaml"
    key="${mule.key}">
    <secure-properties:encrypt algorithm="AES" />
</secure-properties:config>
```
**What this does**:
* Loads secure-properties.yaml
* Decrypts values starting with ![...]
* Makes them available via secure::propertyName

## 5. secure-properties.yaml Example

Example encrypted Salesforce password:
```    
salesforce.jwt.keyStorePassword: "![SIxxxxxxxxxxxxFmL+Tg==]"
```

Encrypt a value using MuleSoft’s tool:
```
java -cp secure-properties-tool.jar \
  com.mulesoft.tools.SecurePropertiesTool encrypt \
  -k mySecretKey -v "MySensitivePassword"
```

## 6. Using Secure Properties in Mule
Access encrypted values using the `secure::` prefix:

```xml
storePassword="#[p('secure::salesforce.jwt.keyStorePassword')]"
```
📌 The `secure::` prefix is mandatory.    


Example: Salesforce JWT Configuration
```xml
<salesforce:sfdc-config name="Salesforce_Config">
    <salesforce:jwt-connection
        consumerKey="#[p('salesforce.jwt.consumerKey')]"
        principal="#[p('salesforce.jwt.username')]"
        keyStore="#[p('salesforce.jwt.keyStorePath')]"
        storePassword="#[p('secure::salesforce.jwt.keyStorePassword')]"
        certificateAlias="#[p('salesforce.jwt.certificateAlias')]"
        tokenEndpoint="#[p('salesforce.loginUrl') ++ '/services/oauth2/token']"
        audienceUrl="#[p('salesforce.jwt.audienceUrl')]" />
</salesforce:sfdc-config>
```

## 7. Summary
  
| Element                | Description                                  |
|------------------------|----------------------------------------------|
|Maven dependency        | Required Secure Properties module            |
|JVM parameter           | `-M-Dmule.key=xxxxxx`                        |
|Local default key       | Global property in Mule config               |
|secure-properties.yaml  | Contains encrypted values                    |
|Encryption tool         | `secure-properties-tool.jar`                 |
|Access prefix           | `secure::propertyName`                       |
|Example usage           | Salesforce JWT config                        |
