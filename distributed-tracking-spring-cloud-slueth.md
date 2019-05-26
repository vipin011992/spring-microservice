# Distributed Tracing

   In microservice, logger are stored for services separately in separate project structure, so it is very hard to trace
   and debug any error, To resolve this distributed tracing comes in the picture.
   
# Distributed tracing via spring-cloud-slueth

    It assign an uniqueId for all the request by which logger can be traced through this id.
    
    Implementation
    
    NetflixZuulApiGatewayServerApplication.java
    
         @EnableZuulProxy
         @EnableDiscoveryClient
         @SpringBootApplication
         public class NetflixZuulApiGatewayServerApplication {
            public static void main(String[] args) {
		          SpringApplication.run(NetflixZuulApiGatewayServerApplication.class, args);
	          }
	
	          @Bean
	          public Sampler defaultSampler(){
		          return Sampler.ALWAYS_SAMPLE;
	          }
          }
          
          
      CurrencyConversionServiceApplication.java    


          @SpringBootApplication
          @EnableFeignClients("com.in28minutes.microservices.currencyconversionservice")
          @EnableDiscoveryClient
          public class CurrencyConversionServiceApplication {

	          public static void main(String[] args) {
		          SpringApplication.run(CurrencyConversionServiceApplication.class, args);
	          }

	          @Bean
	          public Sampler defaultSampler() {
		          return Sampler.ALWAYS_SAMPLE;
	          }
          }

     CurrencyExchangeServiceApplication.java

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
