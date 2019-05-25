# Connecting limit-service to spring-cloud-config-server

  4.1 In limit-service project make below changes
  
      remove application.properties
      add bootstrap.properties 
          spring.application.name=limit-service
          spring.cloud.config.uri=http://localhost:8888
      
  4.2 On the basis of spring.application.name property, it pick configration file from 'spring-cloud-config-server'
  4.3 Here profile is not configured so default configuration file will be picked, configuration profile can be configured as 
         
      spring.profiles.active=dev
      
      If any property is not found in this file then default will be configured.
