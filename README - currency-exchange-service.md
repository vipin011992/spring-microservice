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
      
