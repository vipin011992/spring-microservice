# spring-microservice
  #3.Project - spring-cloud-config-server
  
  3.1 Link "git-localconfig-repo" to this project as source
  
  3.2 springCloudConfigServerApplication.java
  
      @EnableConfigServer
      @SpringBootApplication
      public class SpringCloudConfigServerApplication {
      
	      public static void main(String[] args) {
		      SpringApplication.run(SpringCloudConfigServerApplication.class, args);
	      } 
      }
      
   3.3 application.properties
      
      spring.application.name=spring-cloud-config-server
      server.port=8888
      spring.cloud.config.server.git.uri=file:///{replace_with_actual_path}/git-localconfig-repo

   3.4 execution url
   
       http://localhost:8888/limit-service/default
       http://localhost:8888/limit-service/qa
       http://localhost:8888/limit-service/dev  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
