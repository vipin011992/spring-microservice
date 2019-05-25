# Project 6: netflix-eureka-naming-server

  6.1 NetflixEurekaNamingServerApplication.java 
    
    @SpringBootApplication
    @EnableEurekaServer
    public class NetflixEurekaNamingServerApplication {
	public static void main(String[] args) {
		SpringApplication.run(NetflixEurekaNamingServerApplication.class, args);
	}
    }
 
  6.2 application.properties
  
    spring.application.name=netflix-eureka-naming-server
    server.port=8761
    eureka.client.register-with-eureka=false
    eureka.client.fetch-registry=false
  
  6.3 execution url
  
  	http://localhost:8761
  
  
  
  
  
  
  
  
