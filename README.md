# eureka server config description
## Common configuration

    ...

## About eureka server cluster
* 单节点注册<br>
如果所有微服务均只注册到同一eureka server服务节点A，由该节点A同步微服务注册信息到其他eureka server节点。那么当该注册节点A挂掉之后，其他节点将不会收到已注册微服务的同步信息，例如心跳和注销，这将导致其他节点(无论是否开启自我保护)在一定的时间间隔后注销所有来自于节点A的微服务信息，不论这些微服务是否存活。当然，其他节点也不会获取新的注册到已经挂掉的节点A的微服务信息。<br>
试想，生产者服务P只注册到eureka server节点A，消费者服务C只注册到eureka server节点B，两个eureka server节点构成集群，共享微服务信息。正常运行情况下，服务C可以访问到服务P；当节点A挂掉时，则在经过`服务P心跳间隔 + 服务P心跳超时 + 节点B清理无效服务间隔`的时间后(未验证)，节点B(无论是否开启自我保护)会将来自于节点A的服务P注销。因此当服务C再次向节点B拉取最新的注册信息时，它将访问不到服务P，即使服务P还活着。但在节点B注销服务P的时间延迟内，服务C仍然可以访问到服务P。
* 注册信息拉取<br>
微服务的注册信息不仅复制在eureka server端，也会复制到eureka client端。即便eureka server全部挂掉，eureka client间仍然是互通的，客户端会根据自己本地的备份进行连接，这依赖于配置属性：`eureka.client.fetch-registry: true` 表示eureka client是否向eureka server拉取微服务注册信息，它默认为true。如果消费者服务将此项设置为false，它将访问不到注册在eureka server上的生产者服务。<br>
但是，如果eureka client各自注册到不同的eureka server，当其中一个eureka server挂掉后，正如`单节点注册`所述，其它eureka server节点在一定的时间间隔后将注销所有来自于已挂掉的节点的微服务信息，当其它eureka server节点的微服务再次拉取新的注册信息时，它们和来自于已挂掉节点的微服务将不再互通。所以，为了保证在eureka server节点挂掉时eureka client之间仍可以通过注册信息备份互通，请将eureka client注册到所有eureka server节点。配置：`registry-fetch-interval-seconds: 30`表示eureka client向eureka server拉取微服务注册信息的时间隔，默认30秒。<br>
另外，前述是否向eureka server拉取注册信息的配置对于eureka server cluster而言是没有意义的，无论设置为true或者false，微服务信息在各eureka server节点之间都会进行同步。因为eureka server节点之间会进行相互注册，它们之间会共享所有的微服务信息。
