
## La structure de l'application Mule App

```
src/main/mule/
    common/
        global-configs.xml
        secure-config.xml
        http-config.xml
        error-handling.xml

        db-config.xml
        mq-config.xml
        sf-config.xml
        
    impl/
        api.xml
        business-flows.xml
        customer-flows.xml
        order-flows.xml

    utils/
        common-flows.xml
        shared-transformations.xml

src/main/resources/
    dwl/
        common/
        impl/
    properties/
        dev.yaml
        uat.yaml
        prod.yaml
    secure-properties.yaml
    log4j2.xml
```

## Les fichiers configs 

1. creer une application api-parent-pom
    dans le pom.xml 
    <groupId>xxx</groupId>
    <artifactId>api-parent-pom<artifactId>