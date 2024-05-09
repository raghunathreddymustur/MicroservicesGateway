# External Traffic - API Gateway

Setting up gateway using spring api gateway
---------------------------------------
1. Add the following dependencies
    ![img.png](img.png)
2. Set the configuration properties - Give info about config server
    ````yaml
    spring:
      application:
        name: "gatewayserver"
      config:
        import: "optional:configserver:http://localhost:8071/"
      cloud:
        gateway:
          discovery:
            locator:
              enabled: false
              lowerCaseServiceId: true
    
    management:
      endpoints:
        web:
          exposure:
            include: "*"
      endpoint:
        gateway:
          enabled: true
      info:
        env:
          enabled: true
    
    info:
      app:
        name: "gatewayserver"
        description: "Eazy Bank Gateway Server Application"
        version: "1.0.0"
    
    logging:
      level:
        com:
          eazybytes:
            gatewayserver: DEBUG
   ````
3. Connect with eureka server -- place these config properties inside the config server
    ```yaml
           server:
           port: 8072
        
           eureka:
           instance:
           preferIpAddress: true
           client:
           registerWithEureka: true
           fetchRegistry: true
           serviceUrl:
           defaultZone: "http://localhost:8070/eureka/"
    ```
4. Start the server once all the dependent microservices are started
5. Hit `http://localhost:8072/actuator/gateway/routes `to find info about predicates and filter
    ![img_1.png](img_1.png)
6. Invoking the gateway instead of microservice
   Ex: `http://localhost:8072/<ServiceName>/<ApiPath> - ` 
    ![img_2.png](img_2.png)

Custom Routing and adding Filters - Defining custom routing paths
---------------------------------------------
1. Add the RouteLocator defining the bean
   ````java
        @SpringBootApplication
        public class GatewayserverApplication {
        
            public static void main(String[] args) {
                SpringApplication.run(GatewayserverApplication.class, args);
            }
        
            @Bean
            public RouteLocator eazyBankRouteConfig(RouteLocatorBuilder routeLocatorBuilder) {
                return routeLocatorBuilder.routes()
                                .route(p -> p
                                        .path("/eazybank/accounts/**")
                                        .filters( f -> f.rewritePath("/eazybank/accounts/(?<segment>.*)","/${segment}")
                                                .addResponseHeader("X-Response-Time", LocalDateTime.now().toString()))
                                        .uri("lb://ACCOUNTS"))
                            .route(p -> p
                                    .path("/eazybank/loans/**")
                                    .filters( f -> f.rewritePath("/eazybank/loans/(?<segment>.*)","/${segment}")
                                            .addResponseHeader("X-Response-Time", LocalDateTime.now().toString()))
                                    .uri("lb://LOANS"))
                            .route(p -> p
                                    .path("/eazybank/cards/**")
                                    .filters( f -> f.rewritePath("/eazybank/cards/(?<segment>.*)","/${segment}")
                                            .addResponseHeader("X-Response-Time", LocalDateTime.now().toString()))
                                    .uri("lb://CARDS")).build();
        
        
            }
        
        
        }
    ````
2. Examples
   ![img_3.png](img_3.png)
3. Response having custom header added using filter
    ![img_4.png](img_4.png)


Adding Filters
-----------------
