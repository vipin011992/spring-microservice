# Introduction of Eureka naming server

Server will be restarted whenever any instance of currency-exchange-service is added or removed in 'currency-exchange-service.ribbon.listOfServer=...' property,so to resolve this problem naming server is required.

Features
	service Registry  - Whenever a server is up, it will be registered to naming server.
	Service Discovery - Whenever a server is required, a request will come to naming server.
  
  
  
  
  
