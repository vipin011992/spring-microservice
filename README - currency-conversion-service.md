# Project 5: currency-conversion-service

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
	    
       @GetMapping("/currency-converter/from/{from}/to/{to}/quantity/{quantity}")
	     public CurrencyConversionBean convertCurrency(@PathVariable String from,	@PathVariable String to,
          @PathVariable BigDecimal quantity){
		
		      Map<String, String> uriVariables = new HashMap<>();
		      uriVariables.put("from", from);
		      uriVariables.put("to", to);

		      ResponseEntity<CurrencyConversionBean> responseEntity = new RestTemplate().getForEntity(
				                "http://localhost:8000/currency-exchange/from/{from}/to/{to}", 
				                CurrencyConversionBean.class, 
				                uriVariables );
		
		      CurrencyConversionBean response = responseEntity.getBody();
		
		      return new CurrencyConversionBean(response.getId(),from,to,response.getConversionMultiple(),
				                    quantity,quantity.multiply(response.getConversionMultiple()),response.getPort()); 
	      }
    }
  
  5.3 application.properties
  
    spring.application.name=currency-conversion-service
    server.port=8100
  
  5.4 CurrencyConversionServiceApplication.java
  
     @SpringBootApplication
     public class CurrencyConversionServiceApplication {
        public static void main(String[] args) {
		      SpringApplication.run(CurrencyConversionServiceApplication.class, args);
	      }
     }
  
  
   
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
