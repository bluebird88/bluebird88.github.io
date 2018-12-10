title: 汇总Spring-cloud的主要组件和相关特性

date: 2018-12-10 10:33:45
tags: spring-cloud 组件 特性

## 汇总Spring-cloud的主要组件和相关特性

-----

​	这里记录的是Spring-cloud的主要组件和它的相关特性，以便备忘和查询

参考文档：

 - https://blog.csdn.net/forezp/column/info/15197
 - https://blog.csdn.net/u012702547/article/details/77982571

### -- Eureka

 + 功能： 服务注册中心，用于服务的注册、发现

 + 服务器端：

    + 运行时依赖

      ```xml
      
      		<!--eureka server -->
      		<dependency>
      			<groupId>org.springframework.cloud</groupId>
      			<artifactId>spring-cloud-starter-eureka-server</artifactId>
      		</dependency>	
      ```

   + 启动服务器 :只需要一个注解@EnableEurekaServer，这个注解需要在springboot工程的启动application类上加：

     ```java
     @EnableEurekaServer
     @SpringBootApplication
     public class EurekaserverApplication {
     
     	public static void main(String[] args) {
     		SpringApplication.run(EurekaserverApplication.class, args);
     	}
     }	
     ```

+ 客户端

  + 运行时依赖：

  ```xml
  <!-- client lib -->
  <dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-eureka</artifactId>
  </dependency>
  ```

  + 启动客户端：通过注解@EnableEurekaClient 表明自己是一个eurekaclient.

    ```java
    @SpringBootApplication
    @EnableEurekaClient
    @RestController
    public class ServiceHiApplication {
    
    	public static void main(String[] args) {
    		SpringApplication.run(ServiceHiApplication.class, args);
    	}
    
    	@Value("${server.port}")
    	String port;
    	@RequestMapping("/hi")
    	public String home(@RequestParam String name) {
    		return "hi "+name+",i am from port:" +port;
    	}
    
    }
    ```



### -- 服务消费

​	如何实现服务的消费？ 通过特定的服务消费者实现。

​	同时为了屏蔽服务定位的复杂性，通过上述eureka客户端、注释可以获得注册的服务列表，由于可能的服务提供者是多个。为了更加简单的开发，spring-cloud提供了ribbon和feign两个实现负载均衡算法的客户端，简化上述过程。同时feign还实现了直接的服务映射：通过接口和注释，直接将接口定义与服务映射起来，类似jpa的DAO映射

```java
@FeignClient(value = "service-hi")
public interface SchedualServiceHi {
    @RequestMapping(value = "/hi",method = RequestMethod.GET)
    String sayHiFromClientOne(@RequestParam(value = "name") String name);
} 
```



参考：

- feign客户端： https://blog.csdn.net/forezp/article/details/69808079
- feigh代码解读：https://blog.csdn.net/forezp/article/details/73480304
- ribbon+resttemplate: https://blog.csdn.net/forezp/article/details/69788938
- 注意：feign已经内置了ribbon支持
- 

### -- 熔断器（Hystrix）

- 作用：在服务不可用（设定的判定策略）的情况下，实现服务的熔断并返回指定的消息，便于客户端知晓或规避，同时保证服务的可用性（不因为依赖传递导致整个服务体系瘫痪）

- 使用：

  - 引入hystrix

    ```xml
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-hystrix</artifactId>
    </dependency>
    ```

    代码实现：

    ```java
    
        @Autowired
        RestTemplate restTemplate;
    
        @HystrixCommand(fallbackMethod = "hiError")
        public String hiService(String name) {
            return restTemplate.getForObject("http://SERVICE-HI/hi?name="+name,String.class);
        }
    
        public String hiError(String name) {
            return "hi,"+name+",sorry,error!";
        }
    } 
    ```

  - 在feign客户端中，内置了hystrix的支持，需要实现并指定fallback类

    ```java
    @FeignClient(value = "service-hi",fallback = SchedualServiceHiHystric.class)
    public interface SchedualServiceHi {
        @RequestMapping(value = "/hi",method = RequestMethod.GET)
        String sayHiFromClientOne(@RequestParam(value = "name") String name);
    }
    
    // 另外实现的fallbakc类(bean)，实现通用的接口!
    @Component
    public class SchedualServiceHiHystric implements SchedualServiceHi {
        @Override
        public String sayHiFromClientOne(String name) {
            return "sorry "+name;
        }
    } 
    ```


- 可以开启hystrix仪表盘，便于监控服务状态和熔断状态，参考：
  - https://blog.csdn.net/forezp/article/details/69934399

### -- 路由网关(zuul)：

- 作用：实现服务接口的汇聚、分拆、鉴权、日志等，实现统一的接口访问管理功能; 实现路由转发和过滤器。路由功能是微服务的一部分

- zuul有以下功能：

  + Authentication
  + Insights
  + Stress Testing
  + Canary Testing
  + Dynamic Routing
  + Service Migration
  + Load Shedding
  + Security
  + Static Response handling

  - Active/Active traffic management

   

- 基本微服务架构： 

  参考： https://blog.csdn.net/forezp/article/details/69939114

  ​	https://books.studygolang.com/The-Golang-Standard-Library-by-Example/chapter10/10.1.html

  ![微服务架构](/Users/jbi/Documents/my-me/myblog/source/_posts/Technology/Spring-Family/Spring-cloud/images/cloud-arch.png)


### -- 新版本gateway：spring cloud gateway

- 还在开发中，用于取代zuul
- 参考 https://www.fangzhipeng.com/springcloud/2018/12/05/sc-f-gateway2/