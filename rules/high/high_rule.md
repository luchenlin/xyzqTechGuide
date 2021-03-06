# 高可用系统架构设计



### 1 什么是高可用
高可用描述的是一个系统在大部分时间都是可用的，可以为我们提供服务的。高可用代表系统即使在发生硬件故障或者系统升级的时候，服务仍然是可用的。**即：高并发、高可用、高性能。**  
**如何衡量高可用**  
假设你的系统全年都是正常提供服务，那么就是说你系统的可用性是100%，当然这个值是理想状态下，一般都是以几个9来表示系统的可用性，99.99的可用性较多，9越多就代表可用性越强：   
**可用性=平均故障间隔/(平均故障间隔 + 故障恢复平均时间)**  
高可用性指的是系统在架构上通过某些手段，使得当发生异常时，最大程度减少服务中断时间，从而保证了服务的高度可用性。

### 2 什么情况下系统会不可用？
1、黑客攻击；  
2、硬件故障，例如服务器故障。  
3、并发量/用户请求量激增导致整个服务宕掉或者部分服务不可用。  
4、代码中存在问题导致内存泄漏或者其他问题导致程序不可用。  
5、网站架构某个重要的角色比如 Nginx 或者数据库突然不可用。  
6、自然灾害或者人为破坏。  
7、......

### 3 如何提高系统高可用性
#### 3.1 注重代码质量，严格测试全流程
代码质量有问题比如比较常见的内存泄漏、循环依赖都是对系统可用性极大的损害。虽然我们的应用都进行了灾备备份，也设计了各种异常机制，但从代码质量这个源头把关是首先要做好的一件很重要的事情。如何提高代码质量？除了在编码中注意各种规范外，比较实际可用的就是 CodeReview，团队内部对重要系统代码定期进行CodeReview是非常重要的。  
#### 3.2 业务分层
一个大型平台业务是很复杂的，而且各个业务间还有复杂的调用关系。如果把所有业务的实现全部放在一起（代码上放在一起、部署时放在一起）的话，那耦合度就太高了，很容易导致一个问题影响整个平台的稳定性。  
所以我们在架构实施初期就会考虑将业务分层，根据业务不同将整个系统横向切割为多个部分，比如：用户中心、订单系统、支付系统、后台系统、消息系统、日志系统等。  
另外站在技术角度上我们可以将整个系统再划分为三大层，即：应用层、服务层、数据层，这3层的具体定位是：  
+应用层：前端视图展示，前后端用户看到的界面性操作都划到此类；  
+服务层：为应用层提供服务支持与调用的，比如API；  
+数据层：底层数据的存储与交互，主要是：各类数据库、各类缓存等。  
分层的好处就是方便后期的扩展，比如说分布式部署。  
#### 3.3 分布式部署
将大型系统进行模块和业务上的分层，当分层后我们就可以分布式部署了。分布式部署能够解决传统集中式部署带来的服务器性能瓶颈问题。  
分布式的实施有很多方面，如：
+应用及服务的分布式：将不同模块分别部署在不同服务器上、将数据库和API部署在不同服务器上；
+动静分离：将网站系统上所用到的静态资源（如：CSS/JS/图片/文件）等使用单独的域名来部署，不和当前系统根域名相同，以此将动态脚本请求与静态资源请求分离，这样就可以减小应用服务器的负载了。
+数据库分布式：将服务库分布式部署，去中心化。


#### 3.4 使用集群，减少单点故障
想要高可用就要避免使用单点，单台服务器再强应用优化的再极致，只要它宕机，系统就将不可用，所以需要多台机器也就是需要集群，方法论中叫冗余。  
只是有了集群是不能完全满足复杂业务的高可用的，我们要让系统在当前节点宕机的情况下，自己进行切换到好的节点去，这即所谓的故障转移。  
下图所示为一个典型的高可用集群架构，通过对应用、数据库、缓存等设计集群实现整个系统的高可用。
<div align=center> 
<img src="./rules/high/high_avail.png" width = 80%/>
</div>

#### 3.5 限流  
流量控制（flow control），其原理是监控应用流量的 QPS 或并发线程数等指标，当达到指定的阈值时对流量进行控制，以避免被瞬时的流量高峰冲垮，从而保障应用的高可用性。

#### 3.6 超时和重试机制设置  
一旦用户请求超过某个时间的得不到响应，就抛出异常。这个是非常重要的，很多线上系统故障都是因为没有进行超时设置或者超时设置的方式不对导致的。我们在读取第三方服务的时候，尤其适合设置超时和重试机制。一般我们使用一些 RPC 框架的时候，这些框架都自带的超时重试的配置。如果不进行超时设置可能会导致请求响应速度慢，甚至导致请求堆积进而让系统无法在处理请求。重试的次数一般设为 3 次，再多次的重试没有好处，反而会加重服务器压力（部分场景使用失败重试机制会不太适合）。Lisa默认重试次数为3次。

#### 3.7 熔断机制
超时和重试机制设置之外，熔断机制也是很重要的。 **熔断机制说的是系统自动收集所依赖服务的资源使用情况和性能指标，当所依赖的服务恶化或者调用失败次数达到某个阈值的时候就迅速失败，让当前系统立即切换依赖其他备用服务。** 比较常用的是流量控制和熔断降级框架是 Netflix 的 Hystrix 和 alibaba 的 Sentinel。

#### 3.8 异步调用
**异步调用是指调用后不需要关心最后的结果，这样就可以在用户请求完成之后立即返回结果，而具体处理可以后续再做**，例如比较常见的秒杀场景。但是，使用异步之后可能需要适当修改业务流程进行配合，比如用户在提交订单之后，不能立即返回用户订单提交成功，需要在消息队列的订单消费者进程真正处理完该订单之后，甚至出库后，再通过电子邮件或短信通知用户订单成功。除了可以在程序中实现异步之外，我们常常还使用消息队列，消息队列可以通过异步处理提高系统性能（削峰、减少响应所需时间）并且可以降低系统耦合性。

#### 3.9 使用缓存
利用缓存，可以将热点数据缓存下来，这样在一段时间内就避免去数据库中查询。缓存的目的就是减轻服务器的计算压力。  
常见缓存实施种类也很多，比如说有：  
+静态资源走CDN加速；  
+客户端缓存策略，可以减少客户端对于服务器的请求次数；  
+NoSQL存储热点数据，缓解数据库压力，就算数据库挂了，在缓存期内不必去查询数据库，这样调用方不会出错。  



