# Project 5: Enhancement: currency-conversion-service

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
  
  5.4 CurrencyConversionServiceApplication.java
  
     @SpringBootApplication
     @EnableFeignClients("{replace_this_with_package_of_proxy_interface_CurrencyExchangeServiceProxy}")
     public class CurrencyConversionServiceApplication {
        public static void main(String[] args) {
		      SpringApplication.run(CurrencyConversionServiceApplication.class, args);
	      }
     }
     
  5.5 CurrencyExchangeServiceProxy.java
  
    @FeignClient(name="currency-exchange-service", url="localhost:8000")
    public interface CurrencyExchangeServiceProxy {
	      @GetMapping("/currency-exchange/from/{from}/to/{to}")
	      public CurrencyConversionBean retrieveExchangeValue(@PathVariable("from") String from, @PathVariable("to") String to);
    }
  
  
   
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
