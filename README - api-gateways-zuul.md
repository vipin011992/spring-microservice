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
  
  
  
  
  
  
  
