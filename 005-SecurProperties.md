# Secure properties    

## 1. Objectif de la sécurité MuleSoft
Le module Secure Configuration Properties permet de chiffrer des secrets (mots de passe, tokens, clés privées) dans des fichiers .yaml ou .properties, puis de les décrypter automatiquement au runtime Mule.

## 2. Dépendance Maven (obligatoire dans le projet Mule)
Ajouter cette dépendance dans le fichier pom.xml du projet :
```
<dependency>
    <groupId>com.mulesoft.modules</groupId>
    <artifactId>mule-secure-configuration-property-module</artifactId>
    <version>1.3.0</version>
    <classifier>mule-plugin</classifier>
</dependency>
```
📌 Rôle : Ce module permet à Mule Runtime de décrypter les valeurs chiffrées présentes dans **secure-properties.yaml**.    

## 3. Clé de déchiffrement (mule.key)
Tu dois fournir une clé au runtime Mule :    
🔹 En pipeline / CloudHub / déploiement        
```
-M-Dmule.key=xxxxxx
```

🔹 En local (dev), fournir la valeur utilisé comme dans le fichier **secure-properties.yaml**        
```
    <global-property name="mule.key" value="CHANGE-ME-16-KEY" doc:name="Cle de chiffrement (defaut dev local, 16 caracteres)" />
```
⚠️ La clé doit respecter l’algorithme choisi    
AES → clé de 16, 24 ou 32 caractères    
Blowfish → variable    

## 4. Configuration du module de sécurité
Définir la méthode décryptage dans le fichier  **secure-properties.yaml**        
```
    <secure-properties:config name="secure-properties-config" 
        file="secure-properties.yaml" 
        key="${mule.key}">
        <secure-properties:encrypt algorithm="AES" />
    </secure-properties:config>
```
📌 Rôle :    
Charge le fichier secure-properties.yaml    
Décrypte les valeurs ![...]    
Expose les propriétés via secure::xxx    

## 5. Fichier secure-properties.yaml    
Exemple de propriété pour Saleforce Connect    
```
salesforce.jwt.keyStorePassword: "![SILBzI1hvmuGuTJoFmL+Tg==]"
```
📌 La valeur est chiffrée via secure-properties-tool.jar.    
```
java -cp secure-properties-tool.jar com.mulesoft.tools.SecurePropertiesTool encrypt -k mySecretKey -v "MySensitivePassword"
```

## 6. Utilisation dans Mule       
Voila un exemple d'utilisation dans Mule       
```
storePassword="#[p('secure::salesforce.jwt.keyStorePassword')]"
```
📌 Le préfixe **secure**:: est obligatoire pour accéder à une valeur chiffrée.      

Exemple Salesforce:    
```xml
<salesforce:sfdc-config name="Salesforce_Config" doc:name="Salesforce Config">
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



## 7. 👉 Résumé

| Élément                  | Contenu                          |
|--------------------------|----------------------------------|
| Dépendance Maven         | Voir point 2                     |
| Paramètre JVM -Dmule.key | `-M-Dmule.key=xxxxxx`            |
| Valeur par défaut (local)| Voir point 3                     |
| secure-properties.yaml   | Voir point 4                     |
| Chiffrement              | `secure-properties-tool.jar`     |
| Utilisation secure::xxx  | Voir point 6                     |
| Exemple Salesforce       | Voir point 6                     |
