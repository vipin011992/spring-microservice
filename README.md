# spring-microservice

# Objectives

- Establishing Communication between Microservices
- Centralized Microservice Configuration with Spring Cloud Config Server
- Using Spring Cloud Bus to exchange messages about Configuration updates
- Simplify communication with other Microservices using Feign REST Client
- Implement client side load balancing with Ribbon
- Implement dynamic scaling using Eureka Naming Server and Ribbon
- Implement API Gateway with Zuul
- Implement Distributed tracing with Spring Cloud Sleuth and Zipkin
- Implement Fault Tolerance with Zipkin	

## Ports

|     Application       |     Port          |
| ------------- | ------------- |
| Limits Service | 8080, 8081, ... |
| Spring Cloud Config Server | 8888 |
|  |  |
| Currency Exchange Service | 8000, 8001, 8002, ..  |
| Currency Conversion Service | 8100, 8101, 8102, ... |
| Netflix Eureka Naming Server | 8761 |
| Netflix Zuul API Gateway Server | 8765 |
| Zipkin Distributed Tracing Server | 9411 |


## URLs

|     Application       |     URL          |
| ------------- | ------------- |
| Limits Service | http://localhost:8080/limits POST -> http://localhost:8080/actuator/refresh|
|Spring Cloud Config Server| http://localhost:8888/limits-service/default http://localhost:8888/limits-service/dev |
|  Currency Converter Service - Direct Call| http://localhost:8100/currency-converter/from/USD/to/INR/quantity/10|
|  Currency Converter Service - Feign| http://localhost:8100/currency-converter-feign/from/EUR/to/INR/quantity/10000|
| Currency Exchange Service | http://localhost:8000/currency-exchange/from/EUR/to/INR http://localhost:8001/currency-exchange/from/USD/to/INR|
| Eureka | http://localhost:8761/|
| Zuul - Currency Exchange & Exchange Services | http://localhost:8765/currency-exchange-service/currency-exchange/from/EUR/to/INR http://localhost:8765/currency-conversion-service/currency-converter-feign/from/USD/to/INR/quantity/10|
| Zipkin | http://localhost:9411/zipkin/ |
| Spring Cloud Bus Refresh | http://localhost:8080/bus/refresh |

## Zipkin Installation

Quick Start Page
- https://zipkin.io/pages/quickstart

Downloading Zipkin Jar
- https://search.maven.org/remote_content?g=io.zipkin.java&a=zipkin-server&v=LATEST&c=exec

Command to run
```
RABBIT_URI=amqp://localhost java -jar zipkin-server-2.5.2-exec.jar
```

## VM Argument

-Dserver.port=8001

## Commands

```
mkdir git-configuration-repo
cd git-configuration-repo/
git init
git add -A
git commit -m "first commit"
```

## Spring Cloud Configuration

```
spring.cloud.config.failFast=true

```

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
    @EnableDiscoveryClient
    public class CurrencyExchangeServiceApplication {
	    public static void main(String[] args) {
		    SpringApplication.run(CurrencyExchangeServiceApplication.class, args);
	    }
	    
	    @Bean
	    public Sampler defaultSampler(){
		return Sampler.ALWAYS_SAMPLE;
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
      eureka.client.service-url.default-zone=http://localhost:8761/eureka
       
  
  # Project 5: currency-conversion-service
  
  Implementation of load balancer using Ribbon - 
	If multiple instance of currency-exchange-service is executing then instead of configuring hard coded url/port, load balancer           ribbon will be used.
	
 Setting up Zuul API Gateway between microservice invocations -
      
       'currency-conversion-service' will invoked zuul api gateway 'netflix-zuul-api-gateway-server' and this will redirect invocation           to 'currency-exchange-service'
	

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
    #currency-exchange-service.ribbon.listOfServers=http://localhost:8000,http://localhost:8001
    eureka.client.service-url.default-zone=http://localhost:8761/eureka
  
  5.4 CurrencyConversionServiceApplication.java
  
     @SpringBootApplication
     @EnableFeignClients("{replace_this_with_package_of_proxy_interface_CurrencyExchangeServiceProxy}")
     @EnableDiscoveryClient
     public class CurrencyConversionServiceApplication {
        public static void main(String[] args) {
		SpringApplication.run(CurrencyConversionServiceApplication.class, args);
	}
	
	@Bean
	public Sampler defaultSampler(){
		return Sampler.ALWAYS_SAMPLE;
	}
     }
  
   5.5 CurrencyExchangeServiceProxy.java
  
    //@FeignClient(name="currency-exchange-service")
    @FeignClient(name="netflix-zuul-api-gateway-server")
    @RibbonClient(name="currency-exchange-service")
    public interface CurrencyExchangeServiceProxy {
	 //@GetMapping("/currency-exchange/from/{from}/to/{to}")
	 @GetMapping("/currency-exchange-service/currency-exchange/from/{from}/to/{to}")
	 public CurrencyConversionBean retrieveExchangeValue(@PathVariable("from") String from, @PathVariable("to") String to);
    }
  
   # Eureka naming server

	Server will be restarted whenever any instance of currency-exchange-service is added or removed in 'currency-exchange-			service.ribbon.listOfServer=...' property,so to resolve this problem naming server is required.

	Features
		service Registry  - Whenever a server is up, it will be registered to naming server.
		Service Discovery - Whenever a server is required, a request will come to naming server.
  
  
  # Project 6: netflix-eureka-naming-server

  6.1 NetflixEurekaNamingServerApplication.java 
    
    @SpringBootApplication
    @EnableEurekaServer
    public class NetflixEurekaNamingServerApplication {
	public static void main(String[] args) {
		SpringApplication.run(NetflixEurekaNamingServerApplication.class, args);
	}
	
	@Bean
	public Sampler defaultSampler(){
		return Sampler.ALWAYS_SAMPLE;
	}
    }
 
  6.2 application.properties
  
    spring.application.name=netflix-eureka-naming-server
    server.port=8761
    eureka.client.register-with-eureka=false
    eureka.client.fetch-registry=false
  
  6.3 execution url
  
  	http://localhost:8761
  
  
  # Introduction of API Gateways

   Some of the provided features are;
   	
      Authentication, authorisation and security
      Rate Limits
      Fault Toleration
      Service Aggregation

  # Project 7 - netflix-zuul-api-gateway-server

Introduction of api-gateway using zuul.
    
  7.1 NetflixZuulApiGatewayServerApplication
      
      @EnableZuulProxy
      @EnableDiscoveryClient
      @SpringBootApplication
      public class NetflixZuulApiGatewayServerApplication {
        public static void main(String[] args) {
		      SpringApplication.run(NetflixZuulApiGatewayServerApplication.class, args);
	      }
      }
      
   7.2 application.properties
      
      spring.application.name=netflix-zuul-api-gateway-server
      server.port=8765
      eureka.client.service-url.default-zone=http://localhost:8761/eureka
      
   7.3 ZuulLoggingFilter.java
   
      @Component
      public class ZuulLoggingFilter extends ZuulFilter{
        private Logger logger = LoggerFactory.getLogger(this.getClass());
        
        @Override
        public boolean shouldFilter() { return true; }
        
        @Override
        public Object run() {
          HttpServletRequest request = RequestContext.getCurrentContext().getRequest();
          logger.info("request -> {} request uri -> {}", request, request.getRequestURI());
          return null;
        }

	      @Override
	      public String filterType() { return "pre"; }
        
        @Override
        public int filterOrder() { return 1; }
      }

shouldFilter() :- This method will decide, filter is applied or not, if method return true it means filter will be applied.
filterOrder :- This method is useful to define filter order execution when multiple filter are declared.
filterType :- 
    return value can be of pre,post and error.
    pre - filter will be executed before filter execution
    post - filter will be executed after filter execution
    error - filter will be executed when exception raise.
  
  7.4 To intercept url on this filter below url will be constructed :
      
      http://localhost:8765/{application-name}/{uri}
      
      For Example:
      http://localhost:8765/currency-conversion-service/currency-conversion/from/INR/to/USD/quantity/10
      currency-conversion-service - application name
      currency-conversion/from/INR/to/USD/quantity/10 - uri
  
  # Distributed Tracing

   In microservice, logger are stored for services separately in separate project structure, so it is very hard to trace
   and debug any error, To resolve this distributed tracing comes in the picture.
   
# Distributed tracing via spring-cloud-slueth

    It assign an uniqueId for all the request by which logger can be traced through this id.
  
  
  # Distributed Tracing via zipkin

A central location where all the logs are captured from different project, can be implemented through zipkin.

-All the services will be registered to rabbitMQ
-RabbitMQ will load logger to zipkin UI

Implementation steps -
      
       1. Install and run RabbitMQ
       2. Download and run zipkin server ; jav -jar zipkin-server-2.5.2-exec.jar
          configuration:
            RABBIT_URI=amap://localhost
          
          zipkin url: localhost:9411/zipkin
          
       3. add dependency 
          {org.springframework.cloud,spring-cloud-sleuth}
          {org.springframework.cloud,spring-cloud-zipkin}
          {org.springframework.cloud,spring-cloud-bus-amap}
          
          in netflix-zuul-api-gateway-server, currency-exchange-service, currency-conversion-service.
          
       4. execute url   http://localhost:8100/currency-conversion-service/....
       
       5. Log can be traced on zipkin UI dashboard
          http://localhost:9411/zipkin/
          
  
  
  # Spring cloud Bus
  
  	1. Execute multiple instance of limit-service and make changes in limit-service-qa.properties and commit to the git.
	In this case, changes will not be reflected without restarting of limit-service.
	
	To reflect changes, below url can be executed instead of restarting limit-service-
	http://localhost:8080/application/refresh
	http://localhost:8081/application/refresh
		These calling must be performed for every instance.
			
	2. To get rid of limit-service restarting or api called multiple times, spring cloud bus mechanism can be used to reflect
	changes.
	
	3. Steps to use Spring cloud bus :-
	   3.1 Run rabbitMQ server
	   3.2 Add dependencies in spring-cloud-config-server
	   	{org.springframework.cloud, spring-cloud-starter-config}
	   3.3 Now execute API to reflect changes in all the instances
	   	http://localhost:8080/bus/refresh
		
	
   # Fault tolerance using Hysterix
   	1. Dependency to be added {org.springframework.cloud, spring-cloud-starter-hystrix}
	2. @SpringbootApplication
	   @EnableHystrix
	   public class LimitserviceApplication{...}
	
	3. LimitConfigurationcontroller.java
		@GetMapping("/fault-tolerance-example")
		@HystrixCommand(fallbackmethod="fallbackRetrieve")
		public Limitconfiguration retrieveConfiguration{...}
		
		public LimitConfiguration fallbackRetrieve(){...}
	
	
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  

  
  
  

  
  
  

  
  
  
  
  

  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  

  
  
  
  
  
  
  
  
  
  

  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
