# eureka server config description
## Common configuration

    ...

## About eureka server cluster
* 单节点注册<br>
如果所有微服务均只注册到同一eureka server服务节点A，由该节点A同步微服务注册信息到其他eureka server节点。那么当该注册节点A挂掉之后，其他节点将不会收到已注册微服务的同步信息，例如心跳和注销，这将导致其他节点在关闭自我保护的情况下注销所有来自于节点A的微服务信息，不论这些微服务是否存活。当然，其他节点也不会获取到新的微服务的注册信息，因为它们只注册到了节点A。
* 注册信息获取<br>
微服务的注册信息不仅复制在eureka server端，也会复制到eureka client端。即便eureka server全部挂掉，eureka client间仍然是互通的，客户端会根据自己本地的备份进行连接，这依赖于eureka client 的配置属性：`eureka.client.fetch-registry: true` 表示eureka client是否获取eureka server注册表的微服务注册信息，它默认为true。如果服务消费者将此项设置为false，它将访问不到注册在eureka server上的微服务。这项配置对于eureka server cluster而言是没有意义的，无论设置为true或者false，微服务信息在各eureka server节点之间都会进行同步。因为eureka server节点之间会进行相互注册，它们之间会共享所有的微服务信息。
