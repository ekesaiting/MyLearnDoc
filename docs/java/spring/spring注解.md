### @Import注解

等同于在要导入的类上加@Configuration(它内部含有@Component)注解，将其交给spring容器管理，主要是导入一些第三方jar中的类

### @Component注解与@Configuration注解

主要体现在内部@Bean直接返回对象的差别

@Component：原始的调用java方法，通过new返回一个新的对象

@Configuration：内部所有的@Bean方法都会被CGLib代理增强，创建对象前会查看容器中是否已经有对象

### @EnableDiscoveryClient

spring cloud标配，表示这是一个服务，要被注册，springcloud屏蔽了底层选择的注册中心，无论是nacos,还是eureka,zookeeper,只要添加这个注解就可以添加到注册中心

### @HystrixCommand

配合@EnableCircuitBreaker使用，服务提供者用来给自己降级用的注解，当标记的方法出现异常或者执行超过指定时间时，将放弃执行该方法而执行另外一个指定的方法。

### @EnableCircuitBreaker

效果等同于@EnableHystrix，后者是对前者的封装，一般使用@EnableHystrix即可