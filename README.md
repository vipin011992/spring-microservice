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
  
  
  # Connecting limit-service to spring-cloud-config-server

  a. In limit-service project make below changes
  
      remove application.properties
      add bootstrap.properties 
          spring.application.name=limit-service
          spring.cloud.config.uri=http://localhost:8888
      
  b. On the basis of spring.application.name property, it pick configration file from 'spring-cloud-config-server'
  c. Here profile is not configured so default configuration file will be picked, configuration profile can be configured as 
         
      spring.profiles.active=dev      
  If any property is not found in this file then default will be configured.
  
  # 4. Project - currency-exchange-service
   
  4.1 ExchangeValueRepository.java
  
     public interface ExchangeValueRepository extends JpaRepository<ExchangeValue, Long>{
	      ExchangeValue findByFromAndTo(String from, String to);
     }
      
  4.2 CurrencyExchangeServiceApplication.java
  
    @SpringBootApplication
    public class CurrencyExchangeServiceApplication {
	    public static void main(String[] args) {
		    SpringApplication.run(CurrencyExchangeServiceApplication.class, args);
	    }
    }
  
  4.3 ExchangeValue.java
  
    @Entity
    @Getter @Setter @No-arg-constructor @arg-constructor
    public class ExchangeValue {
	
	    @Id
	    private Long id;
	
	    @Column(name="currency_from")
	    private String from;
	
	    @Column(name="currency_to")
	    private String to;
	
	    private BigDecimal conversionMultiple;
	    private int port;
    }
  
  4.4 CurrencyExchangeController.java
  
    @RestController
    public class CurrencyExchangeController {	
	    @Autowired
	    private Environment environment;
	
	    @Autowired
	    private ExchangeValueRepository repository;
	
	    @GetMapping("/currency-exchange/from/{from}/to/{to}")
	    public ExchangeValue retrieveExchangeValue (@PathVariable String from, @PathVariable String to){
		    ExchangeValue exchangeValue = repository.findByFromAndTo(from, to);
		    exchangeValue.setPort(Integer.parseInt(environment.getProperty("local.server.port")));
		    return exchangeValue;
	    }
    } 
  
  4.5 application.properties
      
      spring.application.name=currency-exchange-service
      server.port=8000
      spring.jpa.show-sql=true
      spring.h2.console.enabled=true
       
  
  # Project 5: currency-conversion-service
  
  Implementation of load balancer using Ribbon - 
	If multiple instance of currency-exchange-service is executing then instead of configuring hard coded url/port, load balancer           ribbon will be used.

  5.1 CurrencyConversionBean.java (pojo)
    
    @Getter @Setter @No-arg-constructor @arg-constructor
    public class CurrencyConversionBean {
	    private Long id;
	    private String from;
	    private String to;
	    private BigDecimal conversionMultiple;
	    private BigDecimal quantity;
	    private BigDecimal totalCalculatedAmount;
	    private int port;
    }


  5.2 CurrencyConversionController.java
  
    @RestController
    public class CurrencyConversionController {
    
        @Autowired
	private CurrencyExchangeServiceProxy proxy;    
        
        @GetMapping("/currency-converter-feign/from/{from}/to/{to}/quantity/{quantity}")
	public CurrencyConversionBean convertCurrencyFeign(@PathVariable String from, @PathVariable String to,
			    @PathVariable BigDecimal quantity) {

		CurrencyConversionBean response = proxy.retrieveExchangeValue(from, to);
		return new CurrencyConversionBean(response.getId(), from, to, response.getConversionMultiple(), quantity,
				        quantity.multiply(response.getConversionMultiple()), response.getPort());
	 }
    }
  
  5.3 application.properties
  
    spring.application.name=currency-conversion-service
    server.port=8100
    currency-exchange-service.ribbon.listOfServers=http://localhost:8000,http://localhost:8001
  
  5.4 CurrencyConversionServiceApplication.java
  
     @SpringBootApplication
     @EnableFeignClients("{replace_this_with_package_of_proxy_interface_CurrencyExchangeServiceProxy}")
     public class CurrencyConversionServiceApplication {
        public static void main(String[] args) {
		      SpringApplication.run(CurrencyConversionServiceApplication.class, args);
	      }
     }
  
   5.5 CurrencyExchangeServiceProxy.java
  
    @FeignClient(name="currency-exchange-service")
    @RibbonClient(name="currency-exchange-service")
    public interface CurrencyExchangeServiceProxy {
	 @GetMapping("/currency-exchange/from/{from}/to/{to}")
	 public CurrencyConversionBean retrieveExchangeValue(@PathVariable("from") String from, @PathVariable("to") String to);
    }
  
   # Eureka naming server

	Server will be restarted whenever any instance of currency-exchange-service is added or removed in 'currency-exchange-			service.ribbon.listOfServer=...' property,so to resolve this problem naming server is required.

	Features
		service Registry  - Whenever a server is up, it will be registered to naming server.
		Service Discovery - Whenever a server is required, a request will come to naming server.
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  

  
  
  
  
  
  
  
  
  
  

  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
