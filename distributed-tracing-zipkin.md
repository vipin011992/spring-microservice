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
          
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
