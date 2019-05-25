# spring-microservice
Step wise microservice tutorial


  #1.Project - limit-service
  
  1.1 LimitConfiguration.java (pojo)
      
      class LimitConfiguration{
          @Getter @setter
          int maximum,minimum;
      }
      
   1.2 application.properties
      
      spring.application.name=limit-service
      limit-service.minimum=9
      limit-service.maximum=999
      
   1.3 Configuration.java
   
      @Component
      @ConfigurationProperties("limit-service")
      class configuration{
        @Getter @Setter
        private int minimum,maximum;
      }
  
  1.4 LimitConfigurationController.java
  
      @RestController
      class LimitConfigurationController{
        
        @Autowired private Configuration configuration;
        
        @GetMapping("/limits")
        public LimitConfiguration retrieveData(){
            return new LimitConfiguration(configuration.getMaximum(), configurtion.getMinimum());
        }
        
      }
      
      
   1.5 LimitServiceApplication.java
       
       @SpringBootApplication
       class LimitServiceApplication{
           public static void main(String ...s){
              SpringApplication.run(LimitServiceApplication.class,args);
           }
       }
   
  # Repository: git-localconfig-repo
  #2.Project - git-localconfig-repo
  
  2.1 /git-localconfig-repo/limits-service-dev.properties
      
      limits-service.minimum=1
      limits-service.maximum=111
      
   2.2 /git-localconfig-repo/limits-service-qa.properties
      
      limits-service.minimum=2
      limits-service.maximum=222
      
   2.3 /git-localconfig-repo/limits-service.properties
      
      limits-service.minimum=8
      limits-service.maximum=888
  
  # 3.Project - spring-cloud-config-server
  
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
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  

  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
