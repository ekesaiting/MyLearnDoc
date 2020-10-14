# 服务注册

## Eureka

传统的配置中心，Netflix现已放弃更新，所以不推荐使用，

**使用方式**

* 创建一个Eureka服务端java进程，需导入spring-cloud-starter-eureka-server的jar包，在启动类上@EnbaleEurekaServer的注解，配置yml文件，不将自己注册，并设置访问注册中心的地址，将访问地址设置为其他Eureka服务器可搭建注册中心集群
* 客户端导入spring-cloud-starter-euraka-client的jar包，在启动类上添加@EnableEurekaClient的注解，配置yml文件，将自己注册到注册服务中心，设置服务注册中心地址，设置多个地址（用`,`分隔）表示向整个注册中心集群注册（向设置的第一个Eureka注册中心注册，Eureka注册中心之间彼此通信保持数据一致，当第一个注册中心宕机，Eureka客户端会向第二个注册中心发送请求，以此类推）。

## Zookpeer

作为Eureka不维护的替代品，或搭配Dubbo使用，实际使用较少，不推荐使用

**使用方法**

* 首先安装zookeeper，并且成功启动
* java程序导入spring-cloud-starter-zookeeper-discovery的jar包,在启动类上添加@EnableDiscovery注解，在yml配置文件中配置zookeeper的地址

## Consul

使用go语言实现的一个类型nacos的服务注册与配置中心，带有web界面

**使用方式**

* 安装consul,并成功启动
* java程序导入spring-cloud-starter-consul-discovery的jar包,在启动类上添加@EnableDiscovery注解，在yml配置文件中配置consul的地址



# 服务调用/负载均衡

## RestTemplate+Ribbon

传统的服务调用方式，但Ribbon已不再更新，不推荐使用，spring将推出替代品LoadBlance

**使用方式**

* 一般的注册中心内部都整合Ribbon，不需要额外添加jar包，spring-starter-web中带有RestTemplate,不需要额外导入
* 向IOC容器注册一个RestTemplate，并且在其上添加@LoadBlance注解
* 可以yml配置文件中指定负载均衡策略类（全局），或者创建一个配置类向容器注册一个代表负载均衡策略的对象，并且在启动类上添加@EnableRibbon注解并指定服务名与选择的配置类（可以实现向指定的服务采用特定的负载均衡算法，注意该配置类不可在包扫描路径中，否则全局使用）

**自定义负载均衡算法**

创建一个实现类实现LoadBlancer接口，在controller中注入该实现类对象与DiscoverClient对象,通过DisCoverrClient对象获取指定服务名的所有注册的实例列表，通过实现类的instance方法在所有实例列表中选择一个实例的ip和端口返回，再通过RestTemplate访问该服务

## Feign/OpenFeign

Feign已不再更新，Spring官方创建OpenFeign代替，二者功能差不多，OpenFeign有所增强

**使用方式**

* 在消费者端导入spring-cloud-starter-openfeign的jar包，在启动类上添加@EnableFeignClients注解
* 创建一个接口，添加@FeignClient注解，并指定要调用的服务名，创建要调用的方法（注意要完全匹配路径与形参名和@Pathvariable）
* Feign自带Ribbon，可在配置文件中指定Ribbon的超时时间（拉取可用实例列表与建立连接的时间，默认一秒）
* 向容器中注册一个指定日志级别的对象可以控制feign调用服务时输出的日志级别

# 服务降级/熔断/限流

## Hystrix

已停止更新

**服务降级使用方式**

* 可以在消费端和服务提供端都进行服务降级，导入spring-cloud-starter-netflix-hystrix的jar包，并在需要的方法上添加@HystrixCommand注解，指定服务降级后执行的方法，超时时间等。在类上添加@DefaultProperties可以指定该类上所有方法的默认降级处理方法
* 与Feign整合，在@FeignClient中通过fallback指定服务降级的处理类，实现该接口与其中所有方法

**服务熔断使用方法**

* 在启动类上添加@EnableHystrix或@EnableCircuitBreaker注解，在@HystrixConmand中开启断路器，指定时间串口，调用次数，失败率等

**web界面**

* 导入spring-cloud-starter-netflix-hystrix-dashboard的jar包，在启动类上添加@EnableHystrixDashBoard
* 启动后，进入web界面指定要监控的ip和端口（注意，所有与web和监控相关的都需要导入web和actuator的jar包）

# 网关

zuul已停止更新，且gateway使用webflux,netty等非阻塞组件，性能更高

## Gateway

**使用方式**

* 导入spring-cloud-starter-gateway的jar包
* 在yml文件中开启路由，并且设置路由id,url,predicates，filter等
* 向容器中注入实现GlobalFilter和order接口的实现类，通过filter方法过滤请求头，cookie等信息决定是否放行

# 配置中心

## config

**使用方法**

服务端：

* 创建一个config的服务端，导入spring-cloud-config-server的jar包，在主启动类上添加@EnableConfigServer注解
* 在配置文件中指定github地址，分支等信息

客户端：

* 导入spring-cloud-starter-config的jar包，
* 创建bootstrap.yml配置文件，指定config-server的地址，分支，名称，后缀等

手动修改：

* 引入actuator的jar包，暴露指定端点（嫌麻烦指定 *），在读取yml文件的类上添加@RefreshScope注解
* 访问客户端/actuator/refresh接口，config客户端会从config服务端同步配置信息。（可以配合webhooks使用）

## Bus

自动修改：

只支持RabbitMQ和Kafka

* 安装RabbitMQ，并成功启动
* config客户端和服务端都导入spring-cloud-start-bus-amqp的依赖，在配置文件中指定RabbitMQ的地址，端口，用户名。密码等，在config-server的配置文件中还要指定暴露的刷新RabbitMQ的端点。
* 修改配置文件提交后，访问指定的端点（如：/actuator/bus-refresh）,即可刷新所有config客户端配置文件
* 可以在访问时在端点后添加config客户端服务名加端口实现只通知特定服务更新

# 消息中间件

## Stream

屏蔽底层不同MQ的API的不同，统一使用Stream的API操作消息队列，目前仅支持RabbitMQ和Kafka，实际可能用处不大(不会无聊使用多个消息中间件)，可能是使用spring amqp 等更底层的api, 

**使用方法**

* 导入spring-cloud-starter-stream-rabbit依赖
* 配置yml文件，指定mq类型，地址，queue/topic名称，账号密码等信息，设置output/input等binder
* 创建生产者/消费者的实现类,在其上添加注解@EnableBinding(Source.class/Sink.class)
* 在生产者实现类内部注入MessageChannel对象，通过该对象发送，
* 在消费者实现类内部方法上添加@StreamListener(Sink.INPUT),并且添加方法参数（Message<String> message）当有消息时，Spring会自动注入该对象，通过该对象获取消息内容
* 通过在配置文件中指定同一个组，可以避免消息的重复消费
* 设置了group,可以避免消息的丢失，配置了group后消费者重启后将拉取它宕机时未曾消费的消息，未配置group导致消息丢失

# 服务链路追踪

## sleuth/zipkin

追踪每一次请求在各个微服务之间调用的关系与延迟等属性

**使用方式**

* 下载zipkin的jar包，通过java命令启动。访问：9411/zipkin 进入web界面。
* 应用引入spring-cloud-start-zipkin依赖，配置yml文件，指定本地zipkin的地址与采样率（注意，每一个需要追踪的微服务都需要引入该依赖并且配置相关参数）

# spring cloud alibaba

## Nacos

服务注册与配置中心，代替Eureka,config,bus,支持AP与CP的切换

**注册中心**

使用方式：

* 下载安装nacos,并成功启动，进入:8848/nacos 的web界面

* 导入spring-cloud-starter-alibaba-nacos-discovery依赖，配置nacos的地址

**配置中心：**

* 导入spring-cloud-starter-alibaba-nacos-config
* 配置nacos地址，命名空间，配置文件拓展名
* 要支持动态刷新，在读取配置文件的properties类上添加@RefreshScope注解

nacos**集群部署**

nacos自带内嵌的debry数据库（非常小只有2MB,用java语言编写，可以内嵌进应用，不是内存型数据库，所以nacos停掉以后重启不会丢失数据）

* 在MySql数据库中运行指定的sql文件创建对应的表，
* 修改conf目录下的application.properties文件，添加mysql配置
*  从cluster.conf.example复制出 cluster.conf文件，添加各个nacos的地址（不能是本机回环地址）和端口
* 修改startup.sh脚本，添加启动端口选项，使可以通过命令行控制启动端口
* nginx添加端口转发到各个nacos服务的端口
* 修改本地应用的yml配置文件中nacos的地址为nginx的拦截转发地址

## Sentinel

就是一个类似hystrix的流量监控工具，提供了更强大的web管理界面

**使用方式**

* 下载对应dashboard jar包，通过java命令启动，访问：8080端口，进入web界面
* 应用内导入spring-cloud-starter-alibaba-sentinel 依赖，yml文件中配置dashboard的地址（再次提醒，所有与监控有关的都需要导入actuator的依赖并暴露端点）
* 在web界面配置各种规则
* 通过在方法上添加@SentinelResource注解设置限流后的处理方法，服务降级的处理方法，blockHandle属性控制违背web界面规则的处理逻辑，fallback属性控制java代码中出现异常的处理逻辑，流量控制的处理级别高于java代码异常（在访问前就进行拦截），通过exceptionIgnore排除固定的异常，不进行降级。

**sentinel持久化**

没有配置持久化，应用重启后对于该应用的限流规则在sentinel中将消失

* 添加sentinel-datasource-nacos依赖
* 在yml中配置nacos的地址，group，id，限流规则等信息。
* 在naocs中创建对应服务名的配置文件，一般格式为json,内容为流控规则，如接口名，qps等。

## Seata

分布式事务的解决方案，看网上风评不推荐使用，直接使用MQ，一般情况下不建议使用分布式事务，分布式事务并不能保证事务的一定回滚，如果事务执行失败，但回滚前其他事务修改了原数据，必须人工手动回滚（俗称人肉补偿）

Seata由全局唯一事务id（XID）和三个其他组件组成

* TC 事务协调者：维护全局事务的状态，驱动全局事务的提交或回滚 （记录全局事务状态）
* TM 事务管理者：定义全局事务的范围：开始全局事务、提交或回滚全局事务。（决定全局事务提交与回滚）
* RM 资源管理器: 管理分支事务处理的资源，与TC交谈以注册分支事务和报告分支事务的状态，并驱动分支事务提交或回滚。(负责每一个微服务内部的事务管理并与全局事务交互)

**使用方式**

本地seata-server安装：

* 官网下载Seata,解压后修改file.cong文件
* 修改默认事务组名称，事务日志存储模式，mysql地址，账号密码等
* 创建一个名为seata的mysql数据库，运行解压包中的db_store.sql文件
* 修改register.conf文件，选择注册中心为nacos,并填写nacos的地址
* 进入/bin目录，启动seata

java应用使用：

需要引入seata分布式事务的微服务都需要执行一下配置

* 导入spring-cloud-starter-alibaba-seata依赖，注意依赖的版本与本地的seata版本一致，不一致要剔除seata-all依赖再独立导入
* 在yml文件中配置事务组名称，与file.conf中配置的一致
* 在resource中创建一个file.cong文件（可从seate解压包中将修改后的文件复制）
* 在resource中创建一个resgister.conf文件（可从seate解压包中将修改后的文件复制）
* 为每一个微服务的数据库创建一张undo_log表，记录分支事务信息
* 在需要调用其他多个微服务的方法上（全局事务开启的地方）添加@GlobalTransactional注解，指定一个全局事务名与全局事务回滚后执行的方法。



