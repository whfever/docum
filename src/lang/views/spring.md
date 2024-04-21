# Spring Cloud

[spring 官网](https://spring.io/projects/spring-boot)



## Spring Cloud 是如何实现热更新的

Jul 25, 2017

作为一篇源码分析的文章，本文虽然介绍 Spring Cloud 的热更新机制，但是实际全文内容都不会与 Spring Cloud Config 以及 Spring Cloud Bus 有关，因为前者只是提供了一个远端的配置源，而后者也只是提供了集群环境下的事件触发机制，与核心流程均无太大关系。



### ContextRefresher

顾名思义，`ContextRefresher` 用于刷新 Spring 上下文，在以下场景会调用其 `refresh` 方法。

1. 请求 `/refresh` Endpoint。
2. 集成 Spring Cloud Bus 后，收到 `RefreshRemoteApplicationEvent` 事件（任意集成 Bus 的应用，请求 `/bus/refresh` Endpoint 后都会将事件推送到整个集群）。

这个方法包含了整个刷新逻辑，也是本文分析的重点。

首先看一下这个方法的实现：

```
public synchronized Set<String> refresh() {
  Map<String, Object> before = extract(
      this.context.getEnvironment().getPropertySources());
  addConfigFilesToEnvironment();
  Set<String> keys = changes(before,
      extract(this.context.getEnvironment().getPropertySources())).keySet();
  this.context.publishEvent(new EnvironmentChangeEvent(keys));
  this.scope.refreshAll();
  return keys;
}
```

首先是第一步 `extract`，这个方法接收了当前环境中的所有属性源（PropertySource），并将其中的非标准属性源的所有属性汇总到一个 Map 中返回。

这里的标准属性源指的是 `StandardEnvironment` 和 `StandardServletEnvironment`，前者会注册系统变量（System Properties）和环境变量（System Environment），后者会注册 Servlet 环境下的 Servlet Context 和 Servlet Config 的初始参数（Init Params）和 JNDI 的属性。个人理解是因为这些属性无法改变，所以不进行刷新。

第二步 `addConfigFilesToEnvironment` 是核心逻辑，它创建了一个新的 Spring Boot 应用并初始化：

```
SpringApplicationBuilder builder = new SpringApplicationBuilder(Empty.class)
    .bannerMode(Banner.Mode.OFF).web(false).environment(environment);
// Just the listeners that affect the environment (e.g. excluding logging
// listener because it has side effects)
builder.application()
    .setListeners(
        Arrays.asList(new BootstrapApplicationListener(),
            new ConfigFileApplicationListener()));
capture = builder.run();
```

这个应用只是为了重新加载一遍属性源，所以只配置了 `BootstrapApplicationListener` 和 `ConfigFileApplicationListener`，最后将新加载的属性源替换掉原属性源，至此属性源本身已经完成更新了。

此时属性源虽然已经更新了，但是配置项都已经注入到了对应的 Spring Bean 中，需要重新进行绑定，所以又触发了两个操作：

1. 将刷新后发生更改的 Key 收集起来，发送一个 `EnvironmentChangeEvent` 事件。
2. 调用 `RefreshScope.refreshAll` 方法。

### EnvironmentChangeEvent

在上文中，`ContextRefresher` 发布了一个 `EnvironmentChangeEvent` 事件，接下来看看这个事件产生了哪些影响。

> The application will listen for an EnvironmentChangeEvent and react to the change in a couple of standard ways (additional ApplicationListeners can be added as @Beans by the user in the normal way). When an EnvironmentChangeEvent is observed it will have a list of key values that have changed, and the application will use those to:
>
> 1. Re-bind any @ConfigurationProperties beans in the context
> 2. Set the logger levels for any properties in logging.level.*

官方文档的介绍中提到，这个事件主要会触发两个行为：

1. 重新绑定上下文中所有使用了 `@ConfigurationProperties` 注解的 Spring Bean。
2. 如果 `logging.level.*` 配置发生了改变，重新设置日志级别。

这两段逻辑分别可以在 `ConfigurationPropertiesRebinder` 和 `LoggingRebinder` 中看到。

#### ConfigurationPropertiesRebinder

这个类乍一看代码量特别少，只需要一个 `ConfigurationPropertiesBeans` 和一个 `ConfigurationPropertiesBindingPostProcessor`，然后调用 `rebind` 每个 Bean 即可。但是这两个对象是从哪里来的呢？

```
public void rebind() {
  for (String name : this.beans.getBeanNames()) {
    rebind(name);
  }
}
```

`ConfigurationPropertiesBeans` 需要一个 `ConfigurationBeanFactoryMetaData`， 这个类逻辑很简单，它是一个 `BeanFactoryPostProcessor` 的实现，将所有的 Bean 都存在了内部的一个 Map 中。

而 ConfigurationPropertiesBeans 获得这个 Map 后，会查找每一个 Bean 是否有 `@ConfigurationProperties` 注解，如果有的话就放到自己的 Map 中。

绕了一圈好不容易拿到所有需要重新绑定的 Bean 后，绑定的逻辑就要简单许多了：

```
public boolean rebind(String name) {
  if (!this.beans.getBeanNames().contains(name)) {
    return false;
  }
  if (this.applicationContext != null) {
    try {
      Object bean = this.applicationContext.getBean(name);
      if (AopUtils.isCglibProxy(bean)) {
        bean = getTargetObject(bean);
      }
      this.binder.postProcessBeforeInitialization(bean, name);
      this.applicationContext.getAutowireCapableBeanFactory()
          .initializeBean(bean, name);
      return true;
    }
    catch (RuntimeException e) {
      this.errors.put(name, e);
      throw e;
    }
  }
  return false;
}
```

其中 `postProcessBeforeInitialization` 方法将 Bean 重新绑定了所有属性，并做了校验等操作。

而 `initializeBean` 的实现如下：

```
protected Object initializeBean(final String beanName, final Object bean, RootBeanDefinition mbd) {
  Object wrappedBean = bean;
  if(mbd == null || !mbd.isSynthetic()) {
    wrappedBean = this.applyBeanPostProcessorsBeforeInitialization(bean, beanName);
  }

  try {
    this.invokeInitMethods(beanName, wrappedBean, mbd);
  } catch (Throwable var6) {
    throw new BeanCreationException(mbd != null?mbd.getResourceDescription():null, beanName, "Invocation of init method failed", var6);
  }

  if(mbd == null || !mbd.isSynthetic()) {
    wrappedBean = this.applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
  }

  return wrappedBean;
}
```

其中主要做了三件事：

1. `applyBeanPostProcessorsBeforeInitialization`：调用所有 `BeanPostProcessor` 的 `postProcessBeforeInitialization` 方法。
2. `invokeInitMethods`：如果 Bean 继承了 `InitializingBean`，执行 `afterPropertiesSet` 方法，或是如果 Bean 指定了 `init-method` 属性，如果有则调用对应方法
3. `applyBeanPostProcessorsAfterInitialization`：调用所有 `BeanPostProcessor` 的 `postProcessAfterInitialization` 方法。

之后 `ConfigurationPropertiesRebinder` 就完成整个重新绑定流程了。

#### LoggingRebinder

相比之下 `LoggingRebinder` 的逻辑要简单许多，它只是调用了 `LoggingSystem` 的方法重新设置了日志级别，具体逻辑就不在本文详述了。

### RefreshScope

首先看看这个类的注释：

> Note that all beans in this scope are *only* initialized when first accessed, so the scope forces lazy initialization semantics. The implementation involves creating a proxy for every bean in the scope, so there is a flag
>
> If a bean is refreshed then the next time the bean is accessed (i.e. a method is executed) a new instance is created. All lifecycle methods are applied to the bean instances, so any destruction callbacks that were registered in the bean factory are called when it is refreshed, and then the initialization callbacks are invoked as normal when the new instance is created. A new bean instance is created from the original bean definition, so any externalized content (property placeholders or expressions in string literals) is re-evaluated when it is created.

这里提到了两个重点：

1. 所有 `@RefreshScope` 的 Bean 都是延迟加载的，只有在第一次访问时才会初始化
2. 刷新 Bean 也是同理，下次访问时会创建一个新的对象

再看一下方法实现：

```
public void refreshAll() {
  super.destroy();
  this.context.publishEvent(new RefreshScopeRefreshedEvent());
}
```

这个类中有一个成员变量 `cache`，用于缓存所有已经生成的 Bean，在调用 `get` 方法时尝试从缓存加载，如果没有的话就生成一个新对象放入缓存，并通过 `getBean` 初始化其对应的 Bean：

```
public Object get(String name, ObjectFactory<?> objectFactory) {
  if (this.lifecycle == null) {
    this.lifecycle = new StandardBeanLifecycleDecorator(this.proxyTargetClass);
  }
  BeanLifecycleWrapper value = this.cache.put(name,
      new BeanLifecycleWrapper(name, objectFactory, this.lifecycle));
  try {
    return value.getBean();
  }
  catch (RuntimeException e) {
    this.errors.put(name, e);
    throw e;
  }
}
```

所以在销毁时只需要将整个缓存清空，下次获取对象时自然就可以重新生成新的对象，也就自然绑定了新的属性：

```
public void destroy() {
  List<Throwable> errors = new ArrayList<Throwable>();
  Collection<BeanLifecycleWrapper> wrappers = this.cache.clear();
  for (BeanLifecycleWrapper wrapper : wrappers) {
    try {
      wrapper.destroy();
    }
    catch (RuntimeException e) {
      errors.add(e);
    }
  }
  if (!errors.isEmpty()) {
    throw wrapIfNecessary(errors.get(0));
  }
  this.errors.clear();
}
```

清空缓存后，下次访问对象时就会重新创建新的对象并放入缓存了。

而在清空缓存后，它还会发出一个 `RefreshScopeRefreshedEvent` 事件，在某些 Spring Cloud 的组件中会监听这个事件并作出一些反馈。

### Zuul

Zuul 在收到这个事件后，会将自身的路由设置为 dirty 状态：

```
private static class ZuulRefreshListener implements ApplicationListener<ApplicationEvent> {

  @Autowired
  private ZuulHandlerMapping zuulHandlerMapping;
  
  @Override
  public void onApplicationEvent(ApplicationEvent event) {
    if (event instanceof ContextRefreshedEvent
        || event instanceof RefreshScopeRefreshedEvent
        || event instanceof RoutesRefreshedEvent) {
      this.zuulHandlerMapping.setDirty(true);
    }
  }
}
```

并且当路由实现为 `RefreshableRouteLocator` 时，会尝试刷新路由：

```
public void setDirty(boolean dirty) {
  this.dirty = dirty;
  if (this.routeLocator instanceof RefreshableRouteLocator) {
    ((RefreshableRouteLocator) this.routeLocator).refresh();
  }
}
```

当状态为 dirty 时，Zuul 会在下一次接受请求时重新注册路由，以更新配置：

```
if (this.dirty) {
  synchronized (this) {
    if (this.dirty) {
      registerHandlers();
      this.dirty = false;
    }
  }
}
```

### Eureka

在 Eureka 收到该事件时，对于客户端和服务端都有不同的处理方式：

```
protected static class EurekaClientConfigurationRefresher {

  @Autowired(required = false)
  private EurekaClient eurekaClient;

  @Autowired(required = false)
  private EurekaAutoServiceRegistration autoRegistration;

  @EventListener(RefreshScopeRefreshedEvent.class)
  public void onApplicationEvent(RefreshScopeRefreshedEvent event) {
    //This will force the creation of the EurkaClient bean if not already created
    //to make sure the client will be reregistered after a refresh event
    if(eurekaClient != null) {
      eurekaClient.getApplications();
    }
    if (autoRegistration != null) {
      // register in case meta data changed
      this.autoRegistration.stop();
      this.autoRegistration.start();
    }
  }
}
```

对于客户端来说，只是调用了下 `eurekaClient.getApplications`，理论上这个方法是没有任何效果的，但是查看上面的注释，以及联想到 `RefreshScope` 的延时初始化特性，这个方法调用应该只是为了强制初始化新的 `EurekaClient`。

事实上这里很有趣的是，在 `EurekaClientAutoConfiguration` 中，实际为了 `EurekaClient` 提供了两种初始化方案，分别对应是否有 `RefreshScope`，所以以上的猜测应该是正确的。

而对于服务端来说，`EurekaAutoServiceRegistration` 会将服务端先标记为下线，在进行重新上线。

### 总结

至此，Spring Cloud 的热更新流程就到此结束了，从这些源码中可以总结出以下结论：

1. 通过使用 `ContextRefresher` 可以进行手动的热更新，而不需要依靠 Bus 或是 Endpoint。
2. 热更新会对两类 Bean 进行配置刷新，一类是使用了 `@ConfigurationProperties` 的对象，另一类是使用了 `@RefreshScope` 的对象。
3. 这两种对象热更新的机制不同，前者在同一个对象中重新绑定了所有属性，后者则是利用了 `RefreshScope` 的缓存和延迟加载机制，生成了新的对象。
4. 通过自行监听 `EnvironmentChangeEvent` 事件，也可以获得更改的配置项，以便实现自己的热更新逻辑。
5. 在使用 Eureka 的项目中要谨慎的使用热更新，过于频繁的更新可能会使大量项目频繁的标记下线和上线，需要注意。



## spring  整合 sentinel

如果 Gateway 用 Sentinel ,

建议在 Sentinel 控制台对 网关模块 进行具体的限流，熔断降级配置。

否则还是推荐直接用 Gateway 内置的 RequestRateLimiter 跟 Hystrix 进行熔断限流配置。

> 自定义限流

```Java
/**
 * @description: 网关限流配置
 * @create: 2020-08-26 12:23
 **/
@Configuration
public class GatewaySentinelConfiguration {
     
       
    private final List<ViewResolver> viewResolvers;
    private final ServerCodecConfigurer serverCodecConfigurer;

    public GatewaySentinelConfiguration(ObjectProvider<List<ViewResolver>> viewResolversProvider,
                                        ServerCodecConfigurer serverCodecConfigurer) {
     
       
        this.viewResolvers = viewResolversProvider.getIfAvailable(Collections::emptyList);
        this.serverCodecConfigurer = serverCodecConfigurer;
    }


    @Bean
    @Order(Ordered.HIGHEST_PRECEDENCE)
    public SentinelGatewayBlockExceptionHandler sentinelGatewayBlockExceptionHandler() {
     
       
        // Register the block exception handler for Spring Cloud Gateway.
        return new SentinelGatewayBlockExceptionHandler(viewResolvers, serverCodecConfigurer);
    }

    @Bean
    @Order(Ordered.HIGHEST_PRECEDENCE)
    public GlobalFilter sentinelGatewayFilter() {
     
       
        return new SentinelGatewayFilter();
    }

    /**
     * Spring 容器初始化的时候执行该方法
     */
    @PostConstruct
    public void doInit() {
     
       
        // 加载网关限流规则
        initGatewayRules();
        // 加载自定义限流异常处理器
        initBlockHandler();
    }

    /**
     * 网关限流规则
     * 建议直接在 Sentinel 控制台上配置
     */
    private void initGatewayRules() {
     
       
        Set<GatewayFlowRule> rules = new HashSet<>();
        /*
            resource：资源名称，可以是网关中的 route 名称或者用户自定义的 API 分组名称
            count：限流阈值
            intervalSec：统计时间窗口，单位是秒，默认是 1 秒
         */
        // rules.add(new GatewayFlowRule("order-service")
        //         .setCount(3) // 限流阈值
        //         .setIntervalSec(60)); // 统计时间窗口，单位是秒，默认是 1 秒
        // --------------------限流分组----------start----------
        rules.add(new GatewayFlowRule("url-proxy-1")
                .setCount(1) // 限流阈值
                .setIntervalSec(60)); // 统计时间窗口，单位是秒，默认是 1 秒
        rules.add(new GatewayFlowRule("api-service")
                .setCount(5) // 限流阈值
                .setIntervalSec(60)); // 统计时间窗口，单位是秒，默认是 1 秒
        // --------------------限流分组-----------end-----------

        // 加载网关限流规则
        GatewayRuleManager.loadRules(rules);
        // 加载限流分组
        initCustomizedApis();

        // ---------------熔断-降级配置-------------
        DegradeRule degradeRule = new DegradeRule("api-service") // 资源名称
                .setGrade(RuleConstant.DEGRADE_GRADE_EXCEPTION_RATIO) // 异常比率模式(秒级)
                .setCount(0.5) // 异常比率阈值(50%)
                .setTimeWindow(10); // 熔断降级时间(10s)
        // 加载规则.
        DegradeRuleManager.loadRules(Collections.singletonList(degradeRule));
    }

    /**
     * 自定义限流异常处理器
     */
    private void initBlockHandler() {
     
       
        BlockRequestHandler blockRequestHandler = new BlockRequestHandler() {
     
       
            @Override
            public Mono<ServerResponse> handleRequest(ServerWebExchange serverWebExchange, Throwable throwable) {
     
       
                Map<String, String> result = new HashMap<>(3);
                result.put("code", String.valueOf(HttpStatus.TOO_MANY_REQUESTS.value()));
                result.put("message", HttpStatus.TOO_MANY_REQUESTS.getReasonPhrase());
                result.put("x","xx");
                return ServerResponse.status(HttpStatus.TOO_MANY_REQUESTS)
                        .contentType(MediaType.APPLICATION_JSON)
//                        .body(BodyInserters.fromValue(result));
                        .body(BodyInserters.fromObject(result));
            }
        };

        // 加载自定义限流异常处理器
        GatewayCallbackManager.setBlockHandler(blockRequestHandler);
    }

    /**
     * 分组限流
     */
    private void initCustomizedApis() {
     
       
        //demo
        Set<ApiDefinition> definitions = new HashSet<>();
        // product-api 组
        ApiDefinition api1 = new ApiDefinition("product-api")
                .setPredicateItems(new HashSet<ApiPredicateItem>() {
     
       {
     
       
                    // 匹配 /product-service/product 以及其子路径的所有请求
                    add(new ApiPathPredicateItem().setPattern("/product-service/product/**")
                            .setMatchStrategy(SentinelGatewayConstants.URL_MATCH_STRATEGY_PREFIX));
                }});

        // order-api 组
        ApiDefinition api2 = new ApiDefinition("order-api")
                .setPredicateItems(new HashSet<ApiPredicateItem>() {
     
       {
     
       
                    // 只匹配 /order-service/order/index
                    add(new ApiPathPredicateItem().setPattern("/order-service/order/index"));
                }});
        definitions.add(api1);
        definitions.add(api2);
        // 加载限流分组
        GatewayApiDefinitionManager.loadApiDefinitions(definitions);
    }
}

```





# 分布式

## 分布式事务

## **什么是分布式事务**

分布式事务是指事务的参与者、支持事务的服务器、资源服务器以及事务管理器**「分别位于不同的分布式系统的不同节点之上」**。

一个大的操作由N多的小的操作共同完成。而这些小的操作又分布在不同的服务上。针对于这些操作，**「要么全部成功执行，要么全部不执行」**。

## **为什么会有分布式事务？**

举个例子：

![img](https://ask.qcloudimg.com/http-save/8352478/lz1m3t1xg9.png)

转账是最经典的分布式事务场景，假设用户 A 使用银行 app 发起一笔跨行转账给用户 B，银行系统首先扣掉用户 A 的钱，然后增加用户 B 账户中的余额。

如果其中某个步骤失败，此时就有可能会出现 2 种**「异常」**情况：

- 1.用户 A 的账户扣款成功，用户 B 账户余额增加失败
- 2.用户 A 账户扣款失败，用户 B 账户余额增加成功。

对于银行系统来说，以上 2 种情况都是**「不允许发生」**，此时就需要事务来保证转账操作的成功。

在**「单体应用」**中，我们只需要贴上[**@Transactional**](https://cloud.tencent.com/developer/tools/blog-entry?target=http%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzI3ODcxMzQzMw%3D%3D%26mid%3D2247492167%26idx%3D1%26sn%3D02f4f177efef3d689e825c544d7e25b5%26chksm%3Deb506771dc27ee67afc5a0a4d5d2d2082d2706744343bbccba657537ca7bb2403a7583601307%26scene%3D21%23wechat_redirect) 注解就可以开启事务来保证整个操作的**「原子性」**。

但是看似以上简单的操作，在实际的应用架构中，不可能是单体的服务，我们会把这一系列操作交给**「N个服务」**去完成，也就是拆分成为**「分布式微服务架构」**。

![img](https://ask.qcloudimg.com/http-save/8352478/tgotlh7cio.png)

比如下订单服务，扣库存服务等等，必须要**「保证不同服务状态结果的一致性」**，于是就出现了分布式事务。

## **分布式理论**

### **CAP定理**

在一个分布式系统中，以下三点特性无法同时满足，**「鱼与熊掌不可兼得」**

> 一致性（C）： 在分布式系统中的所有[数据备份](https://cloud.tencent.com/solution/backup?from_column=20065&from=20065)，**「在同一时刻是否拥有同样的值」**。（等同于所有节点访问同一份最新的数据副本）
>
> 可用性（A）： 在集群中一部分节点**「故障」**后，集群整体**「是否还能响应」**客户端的读写请求。（对数据更新具备高可用性）
>
> 分区容错性（P）： 即使出现**「单个组件无法可用,操作依然可以完成」**。

具体地讲在分布式系统中，在任何[数据库](https://cloud.tencent.com/solution/database?from_column=20065&from=20065)设计中，一个Web应用**「至多只能同时支持上面的两个属性」**。显然，任何横向扩展策略都要依赖于数据分区。因此，设计人员必须在一致性与可用性之间做出选择。

### **BASE理论**

在分布式系统中，我们往往追求的是可用性，它的重要程序比一致性要高，那么如何实现高可用性呢？

前人已经给我们提出来了另外一个理论，就是BASE理论，它是用来对CAP定理进行进一步扩充的。BASE理论指的是：

- **「Basically Available（基本可用）」**
- **「Soft state（软状态）」**
- **「Eventually consistent（最终一致性）」**

BASE理论是对CAP中的一致性和可用性进行一个权衡的结果，理论的核心思想就是：我们无法做到强一致，但每个应用都可以根据自身的业务特点，采用适当的方式来使系统达到最终一致性（Eventual consistency）。

## **分布式事务解决方案**

### **两阶段提交（2PC）**

熟悉mysql的同学对两阶段提交应该颇为熟悉，mysql的事务就是通过**「日志系统」**来完成两阶段提交的。

两阶段协议可以用于单机集中式系统，由事务管理器协调多个资源管理器；也可以用于分布式系统，**「由一个全局的事务管理器协调各个子系统的局部事务管理器完成两阶段提交」**。

![img](https://ask.qcloudimg.com/http-save/8352478/4sf9lptbpv.png)

这个协议有**「两个角色」**，

A节点是事务的协调者，B和C是事务的参与者。

事务的提交分成两个阶段

第一个阶段是**「投票阶段」**

- 1.协调者首先将命令**「写入日志」**
- \2. **「发一个prepare命令」**给B和C节点这两个参与者
- 3.B和C收到消息后，根据自己的实际情况，**「判断自己的实际情况是否可以提交」**
- 4.将处理结果**「记录到日志」**系统
- 5.将结果**「返回」**给协调者

![img](https://ask.qcloudimg.com/http-save/8352478/26a16oglz0.png)

第二个阶段是**「决定阶段」**

当A节点收到B和C参与者所有的确认消息后

- 「判断」

  所有协调者

  「是否都可以提交」

  - 如果可以则**「写入日志」**并且发起commit命令
  - 有一个不可以则**「写入日志」**并且发起abort命令

- 参与者收到协调者发起的命令，**「执行命令」**

- 将执行命令及结果**「写入日志」**

- **「返回结果」**给协调者

#### **可能会存在哪些问题？**

- **「单点故障」**：一旦事务管理器出现故障，整个系统不可用
- **「数据不一致」**：在阶段二，如果事务管理器只发送了部分 commit 消息，此时网络发生异常，那么只有部分参与者接收到 commit 消息，也就是说只有部分参与者提交了事务，使得系统数据不一致。
- **「响应时间较长」**：整个消息链路是串行的，要等待响应结果，不适合高并发的场景
- **「不确定性」**：当事务管理器发送 commit 之后，并且此时只有一个参与者收到了 commit，那么当该参与者与事务管理器同时宕机之后，重新选举的事务管理器无法确定该条消息是否提交成功。

### **三阶段提交（3PC）**

三阶段提交又称3PC，相对于2PC来说增加了CanCommit阶段和超时机制。如果段时间内没有收到协调者的commit请求，那么就会自动进行commit，解决了2PC单点故障的问题。

但是性能问题和不一致问题仍然没有根本解决。下面我们还是一起看下三阶段流程的是什么样的？

- 第一阶段：

  「CanCommit阶段」

  这个阶段所做的事很简单，就是协调者询问事务参与者，你是否有能力完成此次事务。

  - 如果都返回yes，则进入第二阶段
  - 有一个返回no或等待响应超时，则中断事务，并向所有参与者发送abort请求

- 第二阶段：**「PreCommit阶段」**此时协调者会向所有的参与者发送PreCommit请求，参与者收到后开始执行事务操作，并将Undo和Redo信息记录到事务日志中。参与者执行完事务操作后（此时属于未提交事务的状态），就会向协调者反馈“Ack”表示我已经准备好提交了，并等待协调者的下一步指令。

- 第三阶段：**「DoCommit阶段」**在阶段二中如果所有的参与者节点都可以进行PreCommit提交，那么协调者就会从“预提交状态”转变为“提交状态”。然后向所有的参与者节点发送"doCommit"请求，参与者节点在收到提交请求后就会各自执行事务提交操作，并向协调者节点反馈“Ack”消息，协调者收到所有参与者的Ack消息后完成事务。相反，如果有一个参与者节点未完成PreCommit的反馈或者反馈超时，那么协调者都会向所有的参与者节点发送abort请求，从而中断事务。

### **补偿事务（TCC）**

TCC其实就是采用的补偿机制，其核心思想是：**「针对每个操作，都要注册一个与其对应的确认和补偿（撤销）操作」**。它分为三个阶段：

**「Try,Confirm,Cancel」**

- Try阶段主要是对

  「业务系统做检测及资源预留」

  ，其主要分为两个阶段

  - Confirm 阶段主要是对**「业务系统做确认提交」**，Try阶段执行成功并开始执行 Confirm阶段时，默认 Confirm阶段是不会出错的。即：只要Try成功，Confirm一定成功。
  - Cancel 阶段主要是在业务执行错误，需要回滚的状态下执行的业务取消，**「预留资源释放」**。

比如下一个订单减一个库存：

![img](https://ask.qcloudimg.com/http-save/8352478/s6k4jk82zh.png)

执行流程：

- Try阶段：订单系统将当前订单状态设置为支付中，库存系统校验当前剩余库存数量是否大于1，然后将可用库存数量设置为库存剩余数量-1，
  - 如果Try阶段**「执行成功」**，执行Confirm阶段，将订单状态修改为支付成功，库存剩余数量修改为可用库存数量
  - 如果Try阶段**「执行失败」**，执行Cancel阶段，将订单状态修改为支付失败，可用库存数量修改为库存剩余数量

TCC 事务机制相比于上面介绍的2PC，解决了其几个缺点：

- 1.**「解决了协调者单点」**，由主业务方发起并完成这个业务活动。业务活动管理器也变成多点，引入集群。
- 2.**「同步阻塞」**：引入超时，超时后进行补偿，并且不会锁定整个资源，将资源转换为业务逻辑形式，粒度变小。
- 3.**「数据一致性」**，有了补偿机制之后，由业务活动管理器控制一致性

总之，TCC 就是通过代码人为实现了两阶段提交，不同的业务场景所写的代码都不一样，并且很大程度的**「增加」**了业务代码的**「复杂度」**，因此，这种模式并不能很好地被复用。

### **本地消息表**

![img](https://ask.qcloudimg.com/http-save/8352478/3qzdr10vbm.png)

执行流程：

- 消息生产方，需要额外建一个消息表，并

  「记录消息发送状态」

  。消息表和业务数据要在一个事务里提交，也就是说他们要在一个数据库里面。然后消息会经过MQ发送到消息的消费方。

  - 如果消息发送失败，会进行重试发送。

- 消息消费方，需要

  「处理」

  这个

  「消息」

  ，并完成自己的业务逻辑。

  - 如果是**「业务上面的失败」**，可以给生产方**「发送一个业务补偿消息」**，通知生产方进行回滚等操作。
  - 此时如果本地事务处理成功，表明已经处理成功了
  - 如果处理失败，那么就会重试执行。

- 生产方和消费方定时扫描本地消息表，把还没处理完成的消息或者失败的消息再发送一遍。

### **消息事务**

消息事务的原理是将两个事务**「通过消息**[**中间件**](https://cloud.tencent.com/product/tdmq?from_column=20065&from=20065)**进行异步解耦」**，和上述的本地消息表有点类似，但是是通过消息中间件的机制去做的，其本质就是'将本地消息表封装到了消息中间件中'。

执行流程：

- 发送prepare消息到消息中间件
- 发送成功后，执行本地事务
  - 如果事务执行成功，则commit，消息中间件将消息下发至消费端
  - 如果事务执行失败，则回滚，消息中间件将这条prepare消息删除
- 消费端接收到消息进行消费，如果消费失败，则不断重试

这种方案也是实现了**「最终一致性」**，对比本地消息表实现方案，不需要再建消息表，**「不再依赖本地数据库事务」**了，所以这种方案更适用于高并发的场景。目前市面上实现该方案的**「只有阿里的 RocketMQ」**。

### **最大努力通知**

最大努力通知的方案实现比较简单，适用于一些最终一致性要求较低的业务。

执行流程：

- 系统 A 本地事务执行完之后，发送个消息到 MQ；
- 这里会有个专门消费 MQ 的服务，这个服务会消费 MQ 并调用系统 B 的接口；
- 要是系统 B 执行成功就 ok 了；要是系统 B 执行失败了，那么最大努力通知服务就定时尝试重新调用系统 B, 反复 N 次，最后还是不行就放弃。

### **Sagas 事务模型**

Saga事务模型又叫做长时间运行的事务

其核心思想是**「将长事务拆分为多个本地短事务」**，由Saga事务协调器协调，如果正常结束那就正常完成，如果**「某个步骤失败，则根据相反顺序一次调用补偿操作」**。

Seata框架中一个分布式事务包含3种角色：

**「Transaction Coordinator (TC)」**：事务协调器，维护全局事务的运行状态，负责协调并驱动全局事务的提交或回滚。**「Transaction Manager (TM)」**：控制全局事务的边界，负责开启一个全局事务，并最终发起全局提交或全局回滚的决议。**「Resource Manager (RM)」**：控制分支事务，负责分支注册、状态汇报，并接收事务协调器的指令，驱动分支（本地）事务的提交和回滚。

seata框架**「为每一个RM维护了一张UNDO_LOG表」**，其中保存了每一次本地事务的回滚数据。

具体流程：1.首先TM 向 TC 申请**「开启一个全局事务」**，全局事务**「创建」**成功并生成一个**「全局唯一的 XID」**。

2.XID 在微服务调用链路的上下文中传播。

3.RM 开始执行这个分支事务，RM首先解析这条SQL语句，**「生成对应的UNDO_LOG记录」**。下面是一条UNDO_LOG中的记录，UNDO_LOG表中记录了分支ID，全局事务ID，以及事务执行的redo和undo数据以供二阶段恢复。

4.RM在同一个本地事务中**「执行业务SQL和UNDO_LOG数据的插入」**。在提交这个本地事务前，RM会向TC**「申请关于这条记录的全局锁」**。

如果申请不到，则说明有其他事务也在对这条记录进行操作，因此它会在一段时间内重试，重试失败则回滚本地事务，并向TC汇报本地事务执行失败。

6.RM在事务提交前，**「申请到了相关记录的全局锁」**，然后直接提交本地事务，并向TC**「汇报本地事务执行成功」**。此时全局锁并没有释放，全局锁的释放取决于二阶段是提交命令还是回滚命令。

7.TC根据所有的分支事务执行结果，向RM**「下发提交或回滚」**命令。

- RM如果**「收到TC的提交命令」**，首先**「立即释放」**相关记录的全局**「锁」**，然后把提交请求放入一个异步任务的队列中，马上返回提交成功的结果给 TC。异步队列中的提交请求真正执行时，只是删除相应 UNDO LOG 记录而已。

- RM如果

  「收到TC的回滚命令」

  ，则会开启一个本地事务，通过 XID 和 Branch ID 查找到相应的 UNDO LOG 记录。将 UNDO LOG 中的后镜与当前数据进行比较，

  - 如果不同，说明数据被当前全局事务之外的动作做了修改。这种情况，需要根据配置策略来做处理。
  - 如果相同，根据 UNDO LOG 中的前镜像和业务 SQL 的相关信息生成并执行回滚的语句并执行，然后提交本地事务达到回滚的目的，最后释放相关记录的全局锁。

**总结**

本文介绍了分布式事务的一些基础理论，并对常用的分布式事务方案进行了讲解。

分布式事务本身就是一个技术难题，业务中具体使用哪种方案还是需要不同的业务特点自行选择，但是我们也会发现，分布式事务会大大的提高流程的复杂度，会带来很多额外的开销工作，**「代码量上去了，业务复杂了，性能下跌了」**。

所以，当我们真实开发的过程中，能不使用分布式事务就不使用。

# Spring  Interview

## 基础

### 1.Spring 是什么？特性？有哪些模块？

![Spring Logo](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/spring-165c27b4-2ea0-409a-8fa5-389c105db0fa.png)Spring Logo

一句话概括：**Spring 是一个轻量级、非入侵式的控制反转 (IoC) 和面向切面 (AOP) 的框架。**

2003 年，一个音乐家 Rod Johnson 决定发展一个轻量级的 Java 开发框架，`Spring`作为 Java 战场的龙骑兵渐渐崛起，并淘汰了`EJB`这个传统的重装骑兵。

![Spring重要版本](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/spring-5d9efb93-03a5-400c-8429-3be7c5eeddfb.png)Spring重要版本

到了现在，企业级开发的标配基本就是 **Spring5** + **Spring Boot 2** + **JDK 8**

> Spring 有哪些特性呢？

Spring 有很多优点：

![Spring特性](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/spring-a0f0ef9d-3289-41ea-94c2-34b7e37ef854.png)Spring特性

1. **IOC** 和 **DI** 的支持

Spring 的核心就是一个大的工厂容器，可以维护所有对象的创建和依赖关系，Spring 工厂用于生成 Bean，并且管理 Bean 的生命周期，实现**高内聚低耦合**的设计理念。

1. AOP 编程的支持

Spring 提供了**面向切面编程**，可以方便的实现对程序进行权限拦截、运行监控等切面功能。

1. 声明式事务的支持

支持通过配置就来完成对事务的管理，而不需要通过硬编码的方式，以前重复的一些事务提交、回滚的 JDBC 代码，都可以不用自己写了。

1. 快捷测试的支持

Spring 对 Junit 提供支持，可以通过**注解**快捷地测试 Spring 程序。

1. 快速集成功能

方便集成各种优秀框架，Spring 不排斥各种优秀的开源框架，其内部提供了对各种优秀框架（如：Struts、Hibernate、MyBatis、Quartz 等）的直接支持。

1. 复杂 API 模板封装

Spring 对 JavaEE 开发中非常难用的一些 API（JDBC、JavaMail、远程调用等）都提供了模板化的封装，这些封装 API 的提供使得应用难度大大降低。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/spring.html#_2-spring-有哪些模块呢)2.Spring 有哪些模块呢？

Spring 框架是分模块存在，除了最核心的`Spring Core Container`是必要模块之外，其他模块都是`可选`，大约有 20 多个模块。

![Spring模块划分](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/spring-bb7c13ea-3174-4b32-84b8-821849ddc377.png)Spring模块划分

最主要的七大模块：

1. **Spring Core**：Spring 核心，它是框架最基础的部分，提供 IOC 和依赖注入 DI 特性。
2. **Spring Context**：Spring 上下文容器，它是 BeanFactory 功能加强的一个子接口。
3. **Spring Web**：它提供 Web 应用开发的支持。
4. **Spring MVC**：它针对 Web 应用中 MVC 思想的实现。
5. **Spring DAO**：提供对 JDBC 抽象层，简化了 JDBC 编码，同时，编码更具有健壮性。
6. **Spring ORM**：它支持用于流行的 ORM 框架的整合，比如：Spring + Hibernate、Spring + iBatis、Spring + JDO 的整合等。
7. **Spring AOP**：即面向切面编程，它提供了与 AOP 联盟兼容的编程实现。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/spring.html#_3-spring-有哪些常用注解呢)3.Spring 有哪些常用注解呢？

Spring 有很多模块，甚至广义的 SpringBoot、SpringCloud 也算是 Spring 的一部分，我们来分模块，按功能来看一下一些常用的注解：

![Spring常用注解](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/spring-8d0a1518-a425-4887-9735-45321095d927.png)Spring常用注解

**Web**:

- @Controller：组合注解（组合了@Component 注解），应用在 MVC 层（控制层）。
- @RestController：该注解为一个组合注解，相当于@Controller 和@ResponseBody 的组合，注解在类上，意味着，该 Controller 的所有方法都默认加上了@ResponseBody。
- @RequestMapping：用于映射 Web 请求，包括访问路径和参数。如果是 Restful 风格接口，还可以根据请求类型使用不同的注解：
  - @GetMapping
  - @PostMapping
  - @PutMapping
  - @DeleteMapping
- @ResponseBody：支持将返回值放在 response 内，而不是一个页面，通常用户返回 json 数据。
- @RequestBody：允许 request 的参数在 request 体中，而不是在直接连接在地址后面。
- @PathVariable：用于接收路径参数，比如 `@RequestMapping(“/hello/{name}”)`申明的路径，将注解放在参数中前，即可获取该值，通常作为 Restful 的接口实现方法。
- @RestController：该注解为一个组合注解，相当于@Controller 和@ResponseBody 的组合，注解在类上，意味着，该 Controller 的所有方法都默认加上了@ResponseBody。

**容器**:

- @Component：表示一个带注释的类是一个“组件”，成为 Spring 管理的 Bean。当使用基于注解的配置和类路径扫描时，这些类被视为自动检测的候选对象。同时@Component 还是一个元注解。
- @Service：组合注解（组合了@Component 注解），应用在 service 层（业务逻辑层）。
- @Repository：组合注解（组合了@Component 注解），应用在 dao 层（数据访问层）。
- @Autowired：Spring 提供的工具（由 Spring 的依赖注入工具（BeanPostProcessor、BeanFactoryPostProcessor）自动注入）。
- @Qualifier：该注解通常跟 @Autowired 一起使用，当想对注入的过程做更多的控制，@Qualifier 可帮助配置，比如两个以上相同类型的 Bean 时 Spring 无法抉择，用到此注解
- @Configuration：声明当前类是一个配置类（相当于一个 Spring 配置的 xml 文件）
- @Value：可用在字段，构造器参数跟方法参数，指定一个默认值，支持 `#{} 跟 \${}` 两个方式。一般将 SpringbBoot 中的 application.properties 配置的属性值赋值给变量。
- @Bean：注解在方法上，声明当前方法的返回值为一个 Bean。返回的 Bean 对应的类中可以定义 init()方法和 destroy()方法，然后在`@Bean(initMethod=”init”,destroyMethod=”destroy”)`定义，在构造之后执行 init，在销毁之前执行 destroy。
- @Scope:定义我们采用什么模式去创建 Bean（方法上，得有@Bean） 其设置类型包括：Singleton 、Prototype、Request 、 Session、GlobalSession。

**AOP**:

- @Aspect:声明一个切面（类上） 使用@After、@Before、@Around 定义建言（advice），可直接将拦截规则（切点）作为参数。
  - `@After` ：在方法执行之后执行（方法上）。
  - `@Before`： 在方法执行之前执行（方法上）。
  - `@Around`： 在方法执行之前与之后执行（方法上）。
  - `@PointCut`： 声明切点 在 java 配置类中使用@EnableAspectJAutoProxy 注解开启 Spring 对 AspectJ 代理的支持（类上）。

**事务：**

- @Transactional：在要开启事务的方法上使用@Transactional 注解，即可声明式开启事务。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/spring.html#_4-spring-中应用了哪些设计模式呢)4.Spring 中应用了哪些设计模式呢？

Spring 框架中广泛使用了不同类型的设计模式，下面我们来看看到底有哪些设计模式?

![Spring中用到的设计模式](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/spring-ee1c5cee-8462-4bae-93ea-ec936cc77640.png)Spring中用到的设计模式

1. **工厂模式** : Spring 容器本质是一个大工厂，使用工厂模式通过 BeanFactory、ApplicationContext 创建 bean 对象。
2. **代理模式** : Spring AOP 功能功能就是通过代理模式来实现的，分为动态代理和静态代理。
3. **单例模式** : Spring 中的 Bean 默认都是单例的，这样有利于容器对 Bean 的管理。
4. **模板模式** : Spring 中 JdbcTemplate、RestTemplate 等以 Template 结尾的对数据库、网络等等进行操作的模板类，就使用到了模板模式。
5. **观察者模式**: Spring 事件驱动模型就是观察者模式很经典的一个应用。
6. **适配器模式** :Spring AOP 的增强或通知 (Advice) 使用到了适配器模式、Spring MVC 中也是用到了适配器模式适配 Controller。
7. **策略模式**：Spring 中有一个 Resource 接口，它的不同实现类，会根据不同的策略去访问资源。

GitHub 上标星 7600+ 的开源知识库《[二哥的 Java 进阶之路open in new window](https://github.com/itwanger/toBeBetterJavaer)》第一版 PDF 终于来了！包括Java基础语法、数组&字符串、OOP、集合框架、Java IO、异常处理、Java 新特性、网络编程、NIO、并发编程、JVM等等，共计 32 万余字，可以说是通俗易懂、风趣幽默……详情戳：[太赞了，GitHub 上标星 7600+ 的 Java 教程open in new window](https://tobebetterjavaer.com/overview/)

微信搜 **沉默王二** 或扫描下方二维码关注二哥的原创公众号沉默王二，回复 **222** 即可免费领取。

![img](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/gongzhonghao.png)

## [#](https://tobebetterjavaer.com/sidebar/sanfene/spring.html#ioc)IOC

### [#](https://tobebetterjavaer.com/sidebar/sanfene/spring.html#_5-说一说什么是-ioc-什么是-di)5.说一说什么是 IOC？什么是 DI?

Java 是面向对象的编程语言，一个个实例对象相互合作组成了业务逻辑，原来，我们都是在代码里创建对象和对象的依赖。

所谓的**IOC**（控制反转）：就是由容器来负责控制对象的生命周期和对象间的关系。以前是我们想要什么，就自己创建什么，现在是我们需要什么，容器就给我们送来什么。

![引入IOC之前和引入IOC之后](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/spring-619da277-c15e-4dd7-9f2b-dbd809a9aaa0.png)引入IOC之前和引入IOC之后

也就是说，控制对象生命周期的不再是引用它的对象，而是容器。对具体对象，以前是它控制其它对象，现在所有对象都被容器控制，所以这就叫**控制反转**。

![控制反转示意图](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/spring-440f5d0e-f4db-462c-97fb-d54407a354d5.png)控制反转示意图

**DI（依赖注入）**：指的是容器在实例化对象的时候把它依赖的类注入给它。有的说法 IOC 和 DI 是一回事，有的说法是 IOC 是思想，DI 是 IOC 的实现。

> **为什么要使用 IOC 呢？**

最主要的是两个字**解耦**，硬编码会造成对象间的过度耦合，使用 IOC 之后，我们可以不用关心对象间的依赖，专心开发应用就行。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/spring.html#_6-能简单说一下-spring-ioc-的实现机制吗)6.能简单说一下 Spring IOC 的实现机制吗？

PS:这道题老三在面试中被问到过，问法是“**你有自己实现过简单的 Spring 吗？**”

Spring 的 IOC 本质就是一个大工厂，我们想想一个工厂是怎么运行的呢？

![工厂运行](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/spring-7678c40f-a48d-4bd5-80f8-e902ad688e11.png)工厂运行

- **生产产品**：一个工厂最核心的功能就是生产产品。在 Spring 里，不用 Bean 自己来实例化，而是交给 Spring，应该怎么实现呢？——答案毫无疑问，**反射**。

  那么这个厂子的生产管理是怎么做的？你应该也知道——**工厂模式**。

- **库存产品**：工厂一般都是有库房的，用来库存产品，毕竟生产的产品不能立马就拉走。Spring 我们都知道是一个容器，这个容器里存的就是对象，不能每次来取对象，都得现场来反射创建对象，得把创建出的对象存起来。

- **订单处理**：还有最重要的一点，工厂根据什么来提供产品呢？订单。这些订单可能五花八门，有线上签签的、有到工厂签的、还有工厂销售上门签的……最后经过处理，指导工厂的出货。

  在 Spring 里，也有这样的订单，它就是我们 bean 的定义和依赖关系，可以是 xml 形式，也可以是我们最熟悉的注解形式。

我们简单地实现一个 mini 版的 Spring IOC：

![mini版本Spring IOC](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/spring-1d55c63d-2d12-43b1-9f43-428f5f4a1413.png)mini版本Spring IOC

**Bean 定义：**

Bean 通过一个配置文件定义，把它解析成一个类型。

- beans.properties

  偷懒，这里直接用了最方便解析的 properties，这里直接用一个`<key,value>`类型的配置来代表 Bean 的定义，其中 key 是 beanName，value 是 class

  

  ```java
  userDao:cn.fighter3.bean.UserDao
  ```

- BeanDefinition.java

  bean 定义类，配置文件中 bean 定义对应的实体

  

  ```java
  public class BeanDefinition {
  
      private String beanName;
  
      private Class beanClass;
       //省略getter、setter
   }
  ```

- ResourceLoader.java

  资源加载器，用来完成配置文件中配置的加载

  

  ```java
  public class ResourceLoader {
  
      public static Map<String, BeanDefinition> getResource() {
          Map<String, BeanDefinition> beanDefinitionMap = new HashMap<>(16);
          Properties properties = new Properties();
          try {
              InputStream inputStream = ResourceLoader.class.getResourceAsStream("/beans.properties");
              properties.load(inputStream);
              Iterator<String> it = properties.stringPropertyNames().iterator();
              while (it.hasNext()) {
                  String key = it.next();
                  String className = properties.getProperty(key);
                  BeanDefinition beanDefinition = new BeanDefinition();
                  beanDefinition.setBeanName(key);
                  Class clazz = Class.forName(className);
                  beanDefinition.setBeanClass(clazz);
                  beanDefinitionMap.put(key, beanDefinition);
              }
              inputStream.close();
          } catch (IOException | ClassNotFoundException e) {
              e.printStackTrace();
          }
          return beanDefinitionMap;
      }
  
  }
  ```

- BeanRegister.java

  对象注册器，这里用于单例 bean 的缓存，我们大幅简化，默认所有 bean 都是单例的。可以看到所谓单例注册，也很简单，不过是往 HashMap 里存对象。

  

  ```java
  public class BeanRegister {
  
      //单例Bean缓存
      private Map<String, Object> singletonMap = new HashMap<>(32);
  
      /**
       * 获取单例Bean
       *
       * @param beanName bean名称
       * @return
       */
      public Object getSingletonBean(String beanName) {
          return singletonMap.get(beanName);
      }
  
      /**
       * 注册单例bean
       *
       * @param beanName
       * @param bean
       */
      public void registerSingletonBean(String beanName, Object bean) {
          if (singletonMap.containsKey(beanName)) {
              return;
          }
          singletonMap.put(beanName, bean);
      }
  
  }
  ```

- **BeanFactory.java**

![BeanFactory](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/spring-c6b3b707-cf53-4c7c-a6f9-8560950806fc.png)BeanFactory

- 对象工厂，我们最**核心**的一个类，在它初始化的时候，创建了 bean 注册器，完成了资源的加载。

- 获取 bean 的时候，先从单例缓存中取，如果没有取到，就创建并注册一个 bean

  

  ```java
  public class BeanFactory {
  
      private Map<String, BeanDefinition> beanDefinitionMap = new HashMap<>();
  
      private BeanRegister beanRegister;
  
      public BeanFactory() {
          //创建bean注册器
          beanRegister = new BeanRegister();
          //加载资源
          this.beanDefinitionMap = new ResourceLoader().getResource();
      }
  
      /**
       * 获取bean
       *
       * @param beanName bean名称
       * @return
       */
      public Object getBean(String beanName) {
          //从bean缓存中取
          Object bean = beanRegister.getSingletonBean(beanName);
          if (bean != null) {
              return bean;
          }
          //根据bean定义，创建bean
          return createBean(beanDefinitionMap.get(beanName));
      }
  
      /**
       * 创建Bean
       *
       * @param beanDefinition bean定义
       * @return
       */
      private Object createBean(BeanDefinition beanDefinition) {
          try {
              Object bean = beanDefinition.getBeanClass().newInstance();
              //缓存bean
              beanRegister.registerSingletonBean(beanDefinition.getBeanName(), bean);
              return bean;
          } catch (InstantiationException | IllegalAccessException e) {
              e.printStackTrace();
          }
          return null;
      }
  }
  ```

- 测试

  - UserDao.java

    我们的 Bean 类，很简单

    

    ```java
    public class UserDao {
    
        public void queryUserInfo(){
            System.out.println("A good man.");
        }
    }
    ```

  - 单元测试

    

    ```java
    public class ApiTest {
        @Test
        public void test_BeanFactory() {
            //1.创建bean工厂(同时完成了加载资源、创建注册单例bean注册器的操作)
            BeanFactory beanFactory = new BeanFactory();
    
            //2.第一次获取bean（通过反射创建bean，缓存bean）
            UserDao userDao1 = (UserDao) beanFactory.getBean("userDao");
            userDao1.queryUserInfo();
    
            //3.第二次获取bean（从缓存中获取bean）
            UserDao userDao2 = (UserDao) beanFactory.getBean("userDao");
            userDao2.queryUserInfo();
        }
    }
    ```

  - 运行结果

    

    ```java
    A good man.
    A good man.
    ```

至此，我们一个乞丐+破船版的 Spring 就完成了，代码也比较完整，有条件的可以跑一下。

PS:因为时间+篇幅的限制，这个 demo 比较简陋，没有面向接口、没有解耦、边界检查、异常处理……健壮性、扩展性都有很大的不足，感兴趣可以学习参考[15]。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/spring.html#_7-说说-beanfactory-和-applicantcontext)7.说说 BeanFactory 和 ApplicantContext?

可以这么形容，BeanFactory 是 Spring 的“心脏”，ApplicantContext 是完整的“身躯”。

![BeanFactory和ApplicantContext的比喻](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/spring-66328446-f89f-4b7a-8d9f-0e1145dd9b2f.png)BeanFactory和ApplicantContext的比喻

- BeanFactory（Bean 工厂）是 Spring 框架的基础设施，面向 Spring 本身。
- ApplicantContext（应用上下文）建立在 BeanFactoty 基础上，面向使用 Spring 框架的开发者。

###### [#](https://tobebetterjavaer.com/sidebar/sanfene/spring.html#beanfactory-接口)BeanFactory 接口

BeanFactory 是类的通用工厂，可以创建并管理各种类的对象。

Spring 为 BeanFactory 提供了很多种实现，最常用的是 XmlBeanFactory，但在 Spring 3.2 中已被废弃，建议使用 XmlBeanDefinitionReader、DefaultListableBeanFactory。

![Spring5 BeanFactory继承体系](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/spring-6e6d4b69-f36c-41e6-b8ba-9277be147c9b.png)Spring5 BeanFactory继承体系

BeanFactory 接口位于类结构树的顶端，它最主要的方法就是 getBean(String var1)，这个方法从容器中返回特定名称的 Bean。

BeanFactory 的功能通过其它的接口得到了不断的扩展，比如 AbstractAutowireCapableBeanFactory 定义了将容器中的 Bean 按照某种规则（比如按名字匹配、按类型匹配等）进行自动装配的方法。

这里看一个 XMLBeanFactory（已过期） 获取 bean 的例子：



```java
public class HelloWorldApp{
   public static void main(String[] args) {
      BeanFactory factory = new XmlBeanFactory (new ClassPathResource("beans.xml"));
      HelloWorld obj = (HelloWorld) factory.getBean("helloWorld");
      obj.getMessage();
   }
}
```

###### [#](https://tobebetterjavaer.com/sidebar/sanfene/spring.html#applicationcontext-接口)ApplicationContext 接口

ApplicationContext 由 BeanFactory 派生而来，提供了更多面向实际应用的功能。可以这么说，使用 BeanFactory 就是手动档，使用 ApplicationContext 就是自动档。

![Spring5 ApplicationContext部分体系类图](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/spring-e201c9a3-f23c-4768-b844-ac7e0ba4bcec.png)Spring5 ApplicationContext部分体系类图

ApplicationContext 继承了 HierachicalBeanFactory 和 ListableBeanFactory 接口，在此基础上，还通过其他的接口扩展了 BeanFactory 的功能，包括：

- Bean instantiation/wiring
- Bean 的实例化/串联
- 自动的 BeanPostProcessor 注册
- 自动的 BeanFactoryPostProcessor 注册
- 方便的 MessageSource 访问（i18n）
- ApplicationEvent 的发布与 BeanFactory 懒加载的方式不同，它是预加载，所以，每一个 bean 都在 ApplicationContext 启动之后实例化

这是 ApplicationContext 的使用例子：



```java
public class HelloWorldApp{
   public static void main(String[] args) {
      ApplicationContext context=new ClassPathXmlApplicationContext("beans.xml");
      HelloWorld obj = (HelloWorld) context.getBean("helloWorld");
      obj.getMessage();
   }
}
```

ApplicationContext 包含 BeanFactory 的所有特性，通常推荐使用前者。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/spring.html#_8-你知道-spring-容器启动阶段会干什么吗)8.你知道 Spring 容器启动阶段会干什么吗？

PS：这道题老三面试被问到过

Spring 的 IOC 容器工作的过程，其实可以划分为两个阶段：**容器启动阶段**和**Bean 实例化阶段**。

其中容器启动阶段主要做的工作是加载和解析配置文件，保存到对应的 Bean 定义中。

![容器启动和Bean实例化阶段](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/spring-8f8103f7-2a51-4858-856e-96a4ac400d76.png)容器启动和Bean实例化阶段

容器启动开始，首先会通过某种途径加载 Congiguration MetaData，在大部分情况下，容器需要依赖某些工具类（BeanDefinitionReader）对加载的 Congiguration MetaData 进行解析和分析，并将分析后的信息组为相应的 BeanDefinition。

![xml配置信息映射注册过程](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/spring-dfb3d8c4-ba8d-4a2c-aef2-4ad425f7180c.png)xml配置信息映射注册过程

最后把这些保存了 Bean 定义必要信息的 BeanDefinition，注册到相应的 BeanDefinitionRegistry，这样容器启动就完成了。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/spring.html#_9-能说一下-spring-bean-生命周期吗)9.能说一下 Spring Bean 生命周期吗？

可以看看：[Spring Bean 生命周期，好像人的一生。。open in new window](https://mp.weixin.qq.com/s/zb6eA3Se0gQoqL8PylCPLw)

在 Spring 中，基本容器 BeanFactory 和扩展容器 ApplicationContext 的实例化时机不太一样，BeanFactory 采用的是延迟初始化的方式，也就是只有在第一次 getBean()的时候，才会实例化 Bean；ApplicationContext 启动之后会实例化所有的 Bean 定义。

Spring IOC 中 Bean 的生命周期大致分为四个阶段：**实例化**（Instantiation）、**属性赋值**（Populate）、**初始化**（Initialization）、**销毁**（Destruction）。

![Bean生命周期四个阶段](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/spring-595fce5b-36cb-4dcb-b08c-8205a1e98d8a.png)Bean生命周期四个阶段

我们再来看一个稍微详细一些的过程：

- **实例化**：第 1 步，实例化一个 Bean 对象
- **属性赋值**：第 2 步，为 Bean 设置相关属性和依赖
- **初始化**：初始化的阶段的步骤比较多，5、6 步是真正的初始化，第 3、4 步为在初始化前执行，第 7 步在初始化后执行，初始化完成之后，Bean 就可以被使用了
- **销毁**：第 8~10 步，第 8 步其实也可以算到销毁阶段，但不是真正意义上的销毁，而是先在使用前注册了销毁的相关调用接口，为了后面第 9、10 步真正销毁 Bean 时再执行相应的方法

![SpringBean生命周期](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/spring-942a927a-86e4-4a01-8f52-9addd89642ff.png)SpringBean生命周期

简单总结一下，Bean 生命周期里初始化的过程相对步骤会多一些，比如前置、后置的处理。

最后通过一个实例来看一下具体的细节： ![Bean一生实例](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/spring-a3b7714e-38f2-433d-97c6-acb1d20f2887.png)

- 定义一个`PersonBean`类，实现`DisposableBean`,`InitializingBean`, `BeanFactoryAware`, `BeanNameAware`这 4 个接口，同时还有自定义的`init-method`和`destroy-method`。



```java
public class PersonBean implements InitializingBean, BeanFactoryAware, BeanNameAware, DisposableBean {

    /**
     * 身份证号
     */
    private Integer no;

    /**
     * 姓名
     */
    private String name;

    public PersonBean() {
        System.out.println("1.调用构造方法：我出生了！");
    }

    public Integer getNo() {
        return no;
    }

    public void setNo(Integer no) {
        this.no = no;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
        System.out.println("2.设置属性：我的名字叫"+name);
    }

    @Override
    public void setBeanName(String s) {
        System.out.println("3.调用BeanNameAware#setBeanName方法:我要上学了，起了个学名");
    }

    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        System.out.println("4.调用BeanFactoryAware#setBeanFactory方法：选好学校了");
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("6.InitializingBean#afterPropertiesSet方法：入学登记");
    }

    public void init() {
        System.out.println("7.自定义init方法：努力上学ing");
    }

    @Override
    public void destroy() throws Exception {
        System.out.println("9.DisposableBean#destroy方法：平淡的一生落幕了");
    }

    public void destroyMethod() {
        System.out.println("10.自定义destroy方法:睡了，别想叫醒我");
    }

    public void work(){
        System.out.println("Bean使用中：工作，只有对社会没有用的人才放假。。");
    }

}
```

- 定义一个`MyBeanPostProcessor`实现`BeanPostProcessor`接口。



```java
public class MyBeanPostProcessor implements BeanPostProcessor {

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("5.BeanPostProcessor.postProcessBeforeInitialization方法：到学校报名啦");
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("8.BeanPostProcessor#postProcessAfterInitialization方法：终于毕业，拿到毕业证啦！");
        return bean;
    }
}
```

- 配置文件，指定`init-method`和`destroy-method`属性



```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean name="myBeanPostProcessor" class="cn.fighter3.spring.life.MyBeanPostProcessor" />
    <bean name="personBean" class="cn.fighter3.spring.life.PersonBean"
          init-method="init" destroy-method="destroyMethod">
        <property name="idNo" value= "80669865"/>
        <property name="name" value="张铁钢" />
    </bean>

</beans>
```

- 测试



```java
public class Main {

    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("spring-config.xml");
        PersonBean personBean = (PersonBean) context.getBean("personBean");
        personBean.work();
        ((ClassPathXmlApplicationContext) context).destroy();
    }
}
```

- 运行结果：



```java
1.调用构造方法：我出生了！
2.设置属性：我的名字叫张铁钢
3.调用BeanNameAware#setBeanName方法:我要上学了，起了个学名
4.调用BeanFactoryAware#setBeanFactory方法：选好学校了
5.BeanPostProcessor#postProcessBeforeInitialization方法：到学校报名啦
6.InitializingBean#afterPropertiesSet方法：入学登记
7.自定义init方法：努力上学ing
8.BeanPostProcessor#postProcessAfterInitialization方法：终于毕业，拿到毕业证啦！
Bean使用中：工作，只有对社会没有用的人才放假。。
9.DisposableBean#destroy方法：平淡的一生落幕了
10.自定义destroy方法:睡了，别想叫醒我
```

关于源码，Bean 创建过程可以查看`AbstractBeanFactory#doGetBean`方法，在这个方法里可以看到 Bean 的实例化，赋值、初始化的过程，至于最终的销毁，可以看看`ConfigurableApplicationContext#close()`。

![Bean生命周期源码追踪](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/spring-d2da20a3-08d0-4648-b9a3-2fff8512b159.png)Bean生命周期源码追踪

### [#](https://tobebetterjavaer.com/sidebar/sanfene/spring.html#_10-bean-定义和依赖定义有哪些方式)10.Bean 定义和依赖定义有哪些方式？

有三种方式：**直接编码方式**、**配置文件方式**、**注解方式**。

![Bean依赖配置方式](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/spring-89f0f50d-a9e4-4dec-b267-cb1a526cb340.png)Bean依赖配置方式

- 直接编码方式：我们一般接触不到直接编码的方式，但其实其它的方式最终都要通过直接编码来实现。
- 配置文件方式：通过 xml、propreties 类型的配置文件，配置相应的依赖关系，Spring 读取配置文件，完成依赖关系的注入。
- 注解方式：注解方式应该是我们用的最多的一种方式了，在相应的地方使用注解修饰，Spring 会扫描注解，完成依赖关系的注入。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/spring.html#_11-有哪些依赖注入的方法)11.有哪些依赖注入的方法？

Spring 支持**构造方法注入**、**属性注入**、**工厂方法注入**,其中工厂方法注入，又可以分为**静态工厂方法注入**和**非静态工厂方法注入**。

![Spring依赖注入方法](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/spring-491f8444-54ba-4628-b8eb-8418a2197096.png)Spring依赖注入方法

- **构造方法注入**

  通过调用类的构造方法，将接口实现类通过构造方法变量传入

  

  ```java
   public CatDaoImpl(String message){
     this. message = message;
   }
  ```

  

  ```java
  <bean id="CatDaoImpl" class="com.CatDaoImpl">
    <constructor-arg value=" message "></constructor-arg>
  </bean>
  ```

- **属性注入**

  通过 Setter 方法完成调用类所需依赖的注入

  

  ```java
   public class Id {
      private int id;
  
      public int getId() { return id; }
  
      public void setId(int id) { this.id = id; }
  }
  ```

  

  ```java
  <bean id="id" class="com.id ">
    <property name="id" value="123"></property>
  </bean>
  ```

- **工厂方法注入**

  - **静态工厂注入**

    静态工厂顾名思义，就是通过调用静态工厂的方法来获取自己需要的对象，为了让 Spring 管理所有对象，我们不能直接通过"工程类.静态方法()"来获取对象，而是依然通过 Spring 注入的形式获取：

    

    ```java
    public class DaoFactory { //静态工厂
    
       public static final FactoryDao getStaticFactoryDaoImpl(){
          return new StaticFacotryDaoImpl();
       }
    }
    
    public class SpringAction {
    
     //注入对象
     private FactoryDao staticFactoryDao;
    
     //注入对象的 set 方法
     public void setStaticFactoryDao(FactoryDao staticFactoryDao) {
         this.staticFactoryDao = staticFactoryDao;
     }
    
    }
    ```

    

    ```java
    //factory-method="getStaticFactoryDaoImpl"指定调用哪个工厂方法
     <bean name="springAction" class=" SpringAction" >
       <!--使用静态工厂的方法注入对象,对应下面的配置文件-->
       <property name="staticFactoryDao" ref="staticFactoryDao"></property>
     </bean>
    
     <!--此处获取对象的方式是从工厂类中获取静态方法-->
    <bean name="staticFactoryDao" class="DaoFactory"
      factory-method="getStaticFactoryDaoImpl"></bean>
    ```

  - **非静态工厂注入**

    非静态工厂，也叫实例工厂，意思是工厂方法不是静态的，所以我们需要首先 new 一个工厂实例，再调用普通的实例方法。

    

    ```java
    //非静态工厂
    public class DaoFactory {
       public FactoryDao getFactoryDaoImpl(){
         return new FactoryDaoImpl();
       }
     }
    
    public class SpringAction {
      //注入对象
      private FactoryDao factoryDao;
    
      public void setFactoryDao(FactoryDao factoryDao) {
        this.factoryDao = factoryDao;
      }
    }
    ```

    

    ```java
     <bean name="springAction" class="SpringAction">
       <!--使用非静态工厂的方法注入对象,对应下面的配置文件-->
       <property name="factoryDao" ref="factoryDao"></property>
     </bean>
    
     <!--此处获取对象的方式是从工厂类中获取实例方法-->
     <bean name="daoFactory" class="com.DaoFactory"></bean>
    
    <bean name="factoryDao" factory-bean="daoFactory" factory-method="getFactoryDaoImpl"></bean>
    ```

### [#](https://tobebetterjavaer.com/sidebar/sanfene/spring.html#_12-spring-有哪些自动装配的方式)12.Spring 有哪些自动装配的方式？

> **什么是自动装配？**

Spring IOC 容器知道所有 Bean 的配置信息，此外，通过 Java 反射机制还可以获知实现类的结构信息，如构造方法的结构、属性等信息。掌握所有 Bean 的这些信息后，Spring IOC 容器就可以按照某种规则对容器中的 Bean 进行自动装配，而无须通过显式的方式进行依赖配置。

Spring 提供的这种方式，可以按照某些规则进行 Bean 的自动装配，`<bean>`元素提供了一个指定自动装配类型的属性：`autowire="<自动装配类型>"`

> **Spring 提供了哪几种自动装配类型？**

Spring 提供了 4 种自动装配类型：

![Spring四种自动装配类型](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/spring-034120d9-88c7-490b-af07-7d48f3b6b7bc.png)Spring四种自动装配类型

- **byName**：根据名称进行自动匹配，假设 Boss 又一个名为 car 的属性，如果容器中刚好有一个名为 car 的 bean，Spring 就会自动将其装配给 Boss 的 car 属性
- **byType**：根据类型进行自动匹配，假设 Boss 有一个 Car 类型的属性，如果容器中刚好有一个 Car 类型的 Bean，Spring 就会自动将其装配给 Boss 这个属性
- **constructor**：与 byType 类似， 只不过它是针对构造函数注入而言的。如果 Boss 有一个构造函数，构造函数包含一个 Car 类型的入参，如果容器中有一个 Car 类型的 Bean，则 Spring 将自动把这个 Bean 作为 Boss 构造函数的入参；如果容器中没有找到和构造函数入参匹配类型的 Bean，则 Spring 将抛出异常。
- **autodetect**：根据 Bean 的自省机制决定采用 byType 还是 constructor 进行自动装配，如果 Bean 提供了默认的构造函数，则采用 byType，否则采用 constructor。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/spring.html#_13-spring-中的-bean-的作用域有哪些)13.Spring 中的 Bean 的作用域有哪些?

Spring 的 Bean 主要支持五种作用域：

![Spring Bean支持作用域](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/spring-08a9cb31-5a4f-4224-94cd-0c0f643a57ea.png)Spring Bean支持作用域

- **singleton** : 在 Spring 容器仅存在一个 Bean 实例，Bean 以单实例的方式存在，是 Bean 默认的作用域。
- **prototype** : 每次从容器重调用 Bean 时，都会返回一个新的实例。

以下三个作用域于只在 Web 应用中适用：

- **request** : 每一次 HTTP 请求都会产生一个新的 Bean，该 Bean 仅在当前 HTTP Request 内有效。
- **session** : 同一个 HTTP Session 共享一个 Bean，不同的 HTTP Session 使用不同的 Bean。
- **globalSession**：同一个全局 Session 共享一个 Bean，只用于基于 Protlet 的 Web 应用，Spring5 中已经不存在了。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/spring.html#_14-spring-中的单例-bean-会存在线程安全问题吗)14.Spring 中的单例 Bean 会存在线程安全问题吗？

首先结论在这：Spring 中的单例 Bean**不是线程安全的**。

因为单例 Bean，是全局只有一个 Bean，所有线程共享。如果说单例 Bean，是一个无状态的，也就是线程中的操作不会对 Bean 中的成员变量执行**查询**以外的操作，那么这个单例 Bean 是线程安全的。比如 Spring mvc 的 Controller、Service、Dao 等，这些 Bean 大多是无状态的，只关注于方法本身。

假如这个 Bean 是有状态的，也就是会对 Bean 中的成员变量进行写操作，那么可能就存在线程安全的问题。

![Spring单例Bean线程安全问题](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/spring-35dacef4-1a9e-45e1-b3f2-5a91227eb244.png)Spring单例Bean线程安全问题

> **单例 Bean 线程安全问题怎么解决呢？**

常见的有这么些解决办法：

1. 将 Bean 定义为多例

   这样每一个线程请求过来都会创建一个新的 Bean，但是这样容器就不好管理 Bean，不能这么办。

2. 在 Bean 对象中尽量避免定义可变的成员变量

   削足适履了属于是，也不能这么干。

3. 将 Bean 中的成员变量保存在 ThreadLocal 中 ⭐

   我们知道 ThredLoca 能保证多线程下变量的隔离，可以在类中定义一个 ThreadLocal 成员变量，将需要的可变成员变量保存在 ThreadLocal 里，这是推荐的一种方式。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/spring.html#_15-说说循环依赖)15.说说循环依赖?

> **什么是循环依赖？**

![Spring循环依赖](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/spring-f8fea53f-56fa-4cca-9199-ec7f648da625.png)Spring循环依赖

Spring 循环依赖：简单说就是自己依赖自己，或者和别的 Bean 相互依赖。

![鸡和蛋](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/spring-0035fd25-2972-4642-a8ec-ee44a566a5bd.png)鸡和蛋

只有单例的 Bean 才存在循环依赖的情况，**原型**(Prototype)情况下，Spring 会直接抛出异常。原因很简单，AB 循环依赖，A 实例化的时候，发现依赖 B，创建 B 实例，创建 B 的时候发现需要 A，创建 A1 实例……无限套娃，直接把系统干垮。

> **Spring 可以解决哪些情况的循环依赖？**

Spring 不支持基于构造器注入的循环依赖，但是假如 AB 循环依赖，如果一个是构造器注入，一个是 setter 注入呢？

看看几种情形：

![循环依赖的几种情形](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/spring-37bb576d-b4af-42ed-91f4-d846ceb012b6.png)循环依赖的几种情形

第四种可以而第五种不可以的原因是 Spring 在创建 Bean 时默认会根据自然排序进行创建，所以 A 会先于 B 进行创建。

所以简单总结，当循环依赖的实例都采用 setter 方法注入的时候，Spring 可以支持，都采用构造器注入的时候，不支持，构造器注入和 setter 注入同时存在的时候，看天。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/spring.html#_16-那-spring-怎么解决循环依赖的呢)16.那 Spring 怎么解决循环依赖的呢？

> PS：其实正确答案是开发人员做好设计，别让 Bean 循环依赖，但是没办法，面试官不想听这个。

我们都知道，单例 Bean 初始化完成，要经历三步：

![Bean初始化步骤](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/spring-867066f1-49d1-4e57-94f9-4c66a3a8797e.png)Bean初始化步骤

注入就发生在第二步，**属性赋值**，结合这个过程，Spring 通过**三级缓存**解决了循环依赖：

1. 一级缓存 : `Map<String,Object>` **singletonObjects**，单例池，用于保存实例化、属性赋值（注入）、初始化完成的 bean 实例
2. 二级缓存 : `Map<String,Object>` **earlySingletonObjects**，早期曝光对象，用于保存实例化完成的 bean 实例
3. 三级缓存 : `Map<String,ObjectFactory<?>>` **singletonFactories**，早期曝光对象工厂，用于保存 bean 创建工厂，以便于后面扩展有机会创建代理对象。

![三级缓存](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/spring-01d92863-a2cb-4f61-8d8d-30ecf0279b28.png)三级缓存

我们来看一下三级缓存解决循环依赖的过程：

当 A、B 两个类发生循环依赖时： ![循环依赖](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/spring-cfc09f84-f8e1-4702-80b6-d115843e81fe.png)

A 实例的初始化过程：

1. 创建 A 实例，实例化的时候把 A 对象⼯⼚放⼊三级缓存，表示 A 开始实例化了，虽然我这个对象还不完整，但是先曝光出来让大家知道

![1](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/spring-1a8bdc29-ff43-4ff4-9b61-3eedd9da59b3.png)1

1. A 注⼊属性时，发现依赖 B，此时 B 还没有被创建出来，所以去实例化 B
2. 同样，B 注⼊属性时发现依赖 A，它就会从缓存里找 A 对象。依次从⼀级到三级缓存查询 A，从三级缓存通过对象⼯⼚拿到 A，发现 A 虽然不太完善，但是存在，把 A 放⼊⼆级缓存，同时删除三级缓存中的 A，此时，B 已经实例化并且初始化完成，把 B 放入⼀级缓存。

![2](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/spring-bf2507bf-96aa-4b88-a58b-7ec41d11bc70.png)2

1. 接着 A 继续属性赋值，顺利从⼀级缓存拿到实例化且初始化完成的 B 对象，A 对象创建也完成，删除⼆级缓存中的 A，同时把 A 放⼊⼀级缓存
2. 最后，⼀级缓存中保存着实例化、初始化都完成的 A、B 对象

![5](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/spring-022f7cb9-2c83-4fe9-b252-b02bd0fb2435.png)5

所以，我们就知道为什么 Spring 能解决 setter 注入的循环依赖了，因为实例化和属性赋值是分开的，所以里面有操作的空间。如果都是构造器注入的化，那么都得在实例化这一步完成注入，所以自然是无法支持了。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/spring.html#_17-为什么要三级缓存-二级不行吗)17.为什么要三级缓存？⼆级不⾏吗？

不行，主要是为了**⽣成代理对象**。如果是没有代理的情况下，使用二级缓存解决循环依赖也是 OK 的。但是如果存在代理，三级没有问题，二级就不行了。

因为三级缓存中放的是⽣成具体对象的匿名内部类，获取 Object 的时候，它可以⽣成代理对象，也可以返回普通对象。使⽤三级缓存主要是为了保证不管什么时候使⽤的都是⼀个对象。

假设只有⼆级缓存的情况，往⼆级缓存中放的显示⼀个普通的 Bean 对象，Bean 初始化过程中，通过 BeanPostProcessor 去⽣成代理对象之后，覆盖掉⼆级缓存中的普通 Bean 对象，那么可能就导致取到的 Bean 对象不一致了。

![二级缓存不行的原因](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/spring-6ece8a46-25b1-459b-8cfa-19fc696dd7d6.png)二级缓存不行的原因

### [#](https://tobebetterjavaer.com/sidebar/sanfene/spring.html#_18-autowired-的实现原理)18.@Autowired 的实现原理？

实现@Autowired 的关键是：**AutowiredAnnotationBeanPostProcessor**

在 Bean 的初始化阶段，会通过 Bean 后置处理器来进行一些前置和后置的处理。

实现@Autowired 的功能，也是通过后置处理器来完成的。这个后置处理器就是 AutowiredAnnotationBeanPostProcessor。

- Spring 在创建 bean 的过程中，最终会调用到 doCreateBean()方法，在 doCreateBean()方法中会调用 populateBean()方法，来为 bean 进行属性填充，完成自动装配等工作。
- 在 populateBean()方法中一共调用了两次后置处理器，第一次是为了判断是否需要属性填充，如果不需要进行属性填充，那么就会直接进行 return，如果需要进行属性填充，那么方法就会继续向下执行，后面会进行第二次后置处理器的调用，这个时候，就会调用到 AutowiredAnnotationBeanPostProcessor 的 postProcessPropertyValues()方法，在该方法中就会进行@Autowired 注解的解析，然后实现自动装配。



```java
/**
* 属性赋值
**/
protected void populateBean(String beanName, RootBeanDefinition mbd, @Nullable BeanWrapper bw) {
          //…………
          if (hasInstAwareBpps) {
              if (pvs == null) {
                  pvs = mbd.getPropertyValues();
              }

              PropertyValues pvsToUse;
              for(Iterator var9 = this.getBeanPostProcessorCache().instantiationAware.iterator(); var9.hasNext(); pvs = pvsToUse) {
                  InstantiationAwareBeanPostProcessor bp = (InstantiationAwareBeanPostProcessor)var9.next();
                  pvsToUse = bp.postProcessProperties((PropertyValues)pvs, bw.getWrappedInstance(), beanName);
                  if (pvsToUse == null) {
                      if (filteredPds == null) {
                          filteredPds = this.filterPropertyDescriptorsForDependencyCheck(bw, mbd.allowCaching);
                      }
                      //执行后处理器，填充属性，完成自动装配
                      //调用InstantiationAwareBeanPostProcessor的postProcessPropertyValues()方法
                      pvsToUse = bp.postProcessPropertyValues((PropertyValues)pvs, filteredPds, bw.getWrappedInstance(), beanName);
                      if (pvsToUse == null) {
                          return;
                      }
                  }
              }
          }
         //…………
  }
```

- postProcessorPropertyValues()方法的源码如下，在该方法中，会先调用 findAutowiringMetadata()方法解析出 bean 中带有@Autowired 注解、@Inject 和@Value 注解的属性和方法。然后调用 metadata.inject()方法，进行属性填充。



```java
  public PropertyValues postProcessProperties(PropertyValues pvs, Object bean, String beanName) {
      //@Autowired注解、@Inject和@Value注解的属性和方法
      InjectionMetadata metadata = this.findAutowiringMetadata(beanName, bean.getClass(), pvs);

      try {
          //属性填充
          metadata.inject(bean, beanName, pvs);
          return pvs;
      } catch (BeanCreationException var6) {
          throw var6;
      } catch (Throwable var7) {
          throw new BeanCreationException(beanName, "Injection of autowired dependencies failed", var7);
      }
  }
```

GitHub 上标星 7600+ 的开源知识库《[二哥的 Java 进阶之路open in new window](https://github.com/itwanger/toBeBetterJavaer)》第一版 PDF 终于来了！包括Java基础语法、数组&字符串、OOP、集合框架、Java IO、异常处理、Java 新特性、网络编程、NIO、并发编程、JVM等等，共计 32 万余字，可以说是通俗易懂、风趣幽默……详情戳：[太赞了，GitHub 上标星 7600+ 的 Java 教程open in new window](https://tobebetterjavaer.com/overview/)

微信搜 **沉默王二** 或扫描下方二维码关注二哥的原创公众号沉默王二，回复 **222** 即可免费领取。

![img](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/gongzhonghao.png)

## [#](https://tobebetterjavaer.com/sidebar/sanfene/spring.html#aop)AOP

### [#](https://tobebetterjavaer.com/sidebar/sanfene/spring.html#_19-说说什么是-aop)19.说说什么是 AOP？

AOP：面向切面编程。简单说，就是把一些业务逻辑中的相同的代码抽取到一个独立的模块中，让业务逻辑更加清爽。

![横向抽取](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/spring-09dbcda4-7c1b-42d6-8520-1a5fc84abbde.png)横向抽取

具体来说，假如我现在要 crud 写一堆业务，可是如何业务代码前后前后进行打印日志和参数的校验呢？

我们可以把`日志记录`和`数据校验`可重用的功能模块分离出来，然后在程序的执行的合适的地方动态地植入这些代码并执行。这样就简化了代码的书写。

![AOP应用示例](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/spring-4754b4c0-0356-4077-a2f9-55e246cf8ba0.png)AOP应用示例

业务逻辑代码中没有参和通用逻辑的代码，业务模块更简洁，只包含核心业务代码。实现了业务逻辑和通用逻辑的代码分离，便于维护和升级，降低了业务逻辑和通用逻辑的耦合性。

AOP 可以将遍布应用各处的功能分离出来形成可重用的组件。在编译期间、装载期间或运行期间实现在不修改源代码的情况下给程序动态添加功能。从而实现对业务逻辑的隔离，提高代码的模块化能力。

![Java语言执行过程](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/spring-2b9859e5-019c-4e87-a449-990d3deae135.png)Java语言执行过程

AOP 的核心其实就是**动态代理**，如果是实现了接口的话就会使用 JDK 动态代理，否则使用 CGLIB 代理，主要应用于处理一些具有横切性质的系统级服务，如日志收集、事务管理、安全检查、缓存、对象池管理等。

> **AOP 有哪些核心概念？**

- **切面**（Aspect）：类是对物体特征的抽象，切面就是对横切关注点的抽象

- **连接点**（Joinpoint）：被拦截到的点，因为 Spring 只支持方法类型的连接点，所以在 Spring 中连接点指的就是被拦截到的方法，实际上连接点还可以是字段或者构造器

- **切点**（Pointcut）：对连接点进行拦截的定位

- **通知**（Advice）：所谓通知指的就是指拦截到连接点之后要执行的代码，也可以称作**增强**

- **目标对象** （Target）：代理的目标对象

- **织入**（Weabing）：织入是将增强添加到目标类的具体连接点上的过程。

  - 编译期织入：切面在目标类编译时被织入

  - 类加载期织入：切面在目标类加载到 JVM 时被织入。需要特殊的类加载器，它可以在目标类被引入应用之前增强该目标类的字节码。

  - 运行期织入：切面在应用运行的某个时刻被织入。一般情况下，在织入切面时，AOP 容器会为目标对象动态地创建一个代理对象。SpringAOP 就是以这种方式织入切面。

    Spring 采用运行期织入，而 AspectJ 采用编译期织入和类加载器织入。

- **引介**（introduction）：引介是一种特殊的增强，可以动态地为类添加一些属性和方法

> **AOP 有哪些环绕方式？**

AOP 一般有 **5 种**环绕方式：

- 前置通知 (@Before)
- 返回通知 (@AfterReturning)
- 异常通知 (@AfterThrowing)
- 后置通知 (@After)
- 环绕通知 (@Around)

![环绕方式](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/spring-320fa34f-6620-419c-b17a-4f516a83caeb.png)环绕方式

多个切面的情况下，可以通过 @Order 指定先后顺序，数字越小，优先级越高。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/spring.html#_20-说说你平时有用到-aop-吗)20.说说你平时有用到 AOP 吗？

PS：这道题老三的同事面试候选人的时候问到了，候选人说了一堆 AOP 原理，同事就势来一句，你能现场写一下 AOP 的应用吗？结果——场面一度很尴尬。虽然我对面试写这种百度就能出来的东西持保留意见，但是还是加上了这一问，毕竟招人最后都是要撸代码的。

这里给出一个小例子，SpringBoot 项目中，利用 AOP 打印接口的入参和出参日志，以及执行时间，还是比较快捷的。

- 引入依赖：引入 AOP 依赖

  

  ```java
          <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-aop</artifactId>
          </dependency>
  ```

- 自定义注解：自定义一个注解作为切点

  

  ```java
  @Retention(RetentionPolicy.RUNTIME)
  @Target({ElementType.METHOD})
  @Documented
  public @interface WebLog {
  }
  ```

- 配置 AOP 切面：

  - @Aspect：标识切面
  - @Pointcut：设置切点，这里以自定义注解为切点，定义切点有很多其它种方式，自定义注解是比较常用的一种。
  - @Before：在切点之前织入，打印了一些入参信息
  - @Around：环绕切点，打印返回参数和接口执行时间

  

  ```java
  @Aspect
  @Component
  public class WebLogAspect {
  
      private final static Logger logger         = LoggerFactory.getLogger(WebLogAspect.class);
  
      /**
       * 以自定义 @WebLog 注解为切点
       **/
      @Pointcut("@annotation(cn.fighter3.spring.aop_demo.WebLog)")
      public void webLog() {}
  
      /**
       * 在切点之前织入
       */
      @Before("webLog()")
      public void doBefore(JoinPoint joinPoint) throws Throwable {
          // 开始打印请求日志
          ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
          HttpServletRequest request = attributes.getRequest();
          // 打印请求相关参数
          logger.info("========================================== Start ==========================================");
          // 打印请求 url
          logger.info("URL            : {}", request.getRequestURL().toString());
          // 打印 Http method
          logger.info("HTTP Method    : {}", request.getMethod());
          // 打印调用 controller 的全路径以及执行方法
          logger.info("Class Method   : {}.{}", joinPoint.getSignature().getDeclaringTypeName(), joinPoint.getSignature().getName());
          // 打印请求的 IP
          logger.info("IP             : {}", request.getRemoteAddr());
          // 打印请求入参
          logger.info("Request Args   : {}",new ObjectMapper().writeValueAsString(joinPoint.getArgs()));
      }
  
      /**
       * 在切点之后织入
       * @throws Throwable
       */
      @After("webLog()")
      public void doAfter() throws Throwable {
          // 结束后打个分隔线，方便查看
          logger.info("=========================================== End ===========================================");
      }
  
      /**
       * 环绕
       */
      @Around("webLog()")
      public Object doAround(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
          //开始时间
          long startTime = System.currentTimeMillis();
          Object result = proceedingJoinPoint.proceed();
          // 打印出参
          logger.info("Response Args  : {}", new ObjectMapper().writeValueAsString(result));
          // 执行耗时
          logger.info("Time-Consuming : {} ms", System.currentTimeMillis() - startTime);
          return result;
      }
  
  }
  ```

- 使用：只需要在接口上加上自定义注解

  

  ```java
      @GetMapping("/hello")
      @WebLog(desc = "这是一个欢迎接口")
      public String hello(String name){
          return "Hello "+name;
      }
  ```

- 执行结果：可以看到日志打印了入参、出参和执行时间 ![执行结果](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/spring-9c14f774-44b9-41b3-a8c0-f2a54385f6ff.png)

### [#](https://tobebetterjavaer.com/sidebar/sanfene/spring.html#_21-说说-jdk-动态代理和-cglib-代理)21.说说 JDK 动态代理和 CGLIB 代理 ？

Spring 的 AOP 是通过[动态代理open in new window](https://mp.weixin.qq.com/s/aZtfwik0weJN5JzYc-JxYg)来实现的，动态代理主要有两种方式 JDK 动态代理和 Cglib 动态代理，这两种动态代理的使用和原理有些不同。

**JDK 动态代理**

1. **Interface**：对于 JDK 动态代理，目标类需要实现一个 Interface。
2. **InvocationHandler**：InvocationHandler 是一个接口，可以通过实现这个接口，定义横切逻辑，再通过反射机制（invoke）调用目标类的代码，在次过程，可能包装逻辑，对目标方法进行前置后置处理。
3. **Proxy**：Proxy 利用 InvocationHandler 动态创建一个符合目标类实现的接口的实例，生成目标类的代理对象。

**CgLib 动态代理**

1. 使用 JDK 创建代理有一大限制，它只能为接口创建代理实例，而 CgLib 动态代理就没有这个限制。
2. CgLib 动态代理是使用字节码处理框架 **ASM**，其原理是通过字节码技术为一个类创建子类，并在子类中采用方法拦截的技术拦截所有父类方法的调用，顺势织入横切逻辑。
3. **CgLib** 创建的动态代理对象性能比 JDK 创建的动态代理对象的性能高不少，但是 CGLib 在创建代理对象时所花费的时间却比 JDK 多得多，所以对于单例的对象，因为无需频繁创建对象，用 CGLib 合适，反之，使用 JDK 方式要更为合适一些。同时，由于 CGLib 由于是采用动态创建子类的方法，对于 final 方法，无法进行代理。

我们来看一个常见的小场景，客服中转，解决用户问题：

![用户向客服提问题](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/spring-c5c4b247-62dd-43a2-a043-da51c58f77c8.png)用户向客服提问题

**JDK 动态代理实现：**

![JDK动态代理类图](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/spring-65b14a3f-2653-463e-af77-a8875d3d635c.png)JDK动态代理类图

- 接口

  

  ```java
  public interface ISolver {
      void solve();
  }
  ```

- 目标类:需要实现对应接口

  

  ```java
  public class Solver implements ISolver {
      @Override
      public void solve() {
          System.out.println("疯狂掉头发解决问题……");
      }
  }
  ```

- 态代理工厂:ProxyFactory，直接用反射方式生成一个目标对象的代理对象，这里用了一个匿名内部类方式重写 InvocationHandler 方法，实现接口重写也差不多

  

  ```java
  public class ProxyFactory {
  
      // 维护一个目标对象
      private Object target;
  
      public ProxyFactory(Object target) {
          this.target = target;
      }
  
      // 为目标对象生成代理对象
      public Object getProxyInstance() {
          return Proxy.newProxyInstance(target.getClass().getClassLoader(), target.getClass().getInterfaces(),
                  new InvocationHandler() {
                      @Override
                      public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                          System.out.println("请问有什么可以帮到您？");
  
                          // 调用目标对象方法
                          Object returnValue = method.invoke(target, args);
  
                          System.out.println("问题已经解决啦！");
                          return null;
                      }
                  });
      }
  }
  ```

- 客户端：Client，生成一个代理对象实例，通过代理对象调用目标对象方法

  

  ```java
  public class Client {
      public static void main(String[] args) {
          //目标对象:程序员
          ISolver developer = new Solver();
          //代理：客服小姐姐
          ISolver csProxy = (ISolver) new ProxyFactory(developer).getProxyInstance();
          //目标方法：解决问题
          csProxy.solve();
      }
  }
  ```

**Cglib 动态代理实现：**

![Cglib动态代理类图](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/spring-74da87af-20d1-4a5b-a212-3837a15f0bab.png)Cglib动态代理类图

- 目标类：Solver，这里目标类不用再实现接口。

  

  ```java
  public class Solver {
  
      public void solve() {
          System.out.println("疯狂掉头发解决问题……");
      }
  }
  ```

- 动态代理工厂：

  

  ```java
  public class ProxyFactory implements MethodInterceptor {
  
     //维护一个目标对象
      private Object target;
  
      public ProxyFactory(Object target) {
          this.target = target;
      }
  
      //为目标对象生成代理对象
      public Object getProxyInstance() {
          //工具类
          Enhancer en = new Enhancer();
          //设置父类
          en.setSuperclass(target.getClass());
          //设置回调函数
          en.setCallback(this);
          //创建子类对象代理
          return en.create();
      }
  
      @Override
      public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
          System.out.println("请问有什么可以帮到您？");
          // 执行目标对象的方法
          Object returnValue = method.invoke(target, args);
          System.out.println("问题已经解决啦！");
          return null;
      }
  
  }
  ```

- 客户端：Client

  

  ```java
  public class Client {
      public static void main(String[] args) {
          //目标对象:程序员
          Solver developer = new Solver();
          //代理：客服小姐姐
          Solver csProxy = (Solver) new ProxyFactory(developer).getProxyInstance();
          //目标方法：解决问题
          csProxy.solve();
      }
  }
  ```

### [#](https://tobebetterjavaer.com/sidebar/sanfene/spring.html#_22-说说-spring-aop-和-aspectj-aop-区别)22.说说 Spring AOP 和 AspectJ AOP 区别?

**Spring AOP**

Spring AOP 属于`运行时增强`，主要具有如下特点：

1. 基于动态代理来实现，默认如果使用接口的，用 JDK 提供的动态代理实现，如果是方法则使用 CGLIB 实现
2. Spring AOP 需要依赖 IOC 容器来管理，并且只能作用于 Spring 容器，使用纯 Java 代码实现
3. 在性能上，由于 Spring AOP 是基于**动态代理**来实现的，在容器启动时需要生成代理实例，在方法调用上也会增加栈的深度，使得 Spring AOP 的性能不如 AspectJ 的那么好。
4. Spring AOP 致力于解决企业级开发中最普遍的 AOP(方法织入)。

**AspectJ**

AspectJ 是一个易用的功能强大的 AOP 框架，属于`编译时增强`， 可以单独使用，也可以整合到其它框架中，是 AOP 编程的完全解决方案。AspectJ 需要用到单独的编译器 ajc。

AspectJ 属于**静态织入**，通过修改代码来实现，在实际运行之前就完成了织入，所以说它生成的类是没有额外运行时开销的，一般有如下几个织入的时机：

1. 编译期织入（Compile-time weaving）：如类 A 使用 AspectJ 添加了一个属性，类 B 引用了它，这个场景就需要编译期的时候就进行织入，否则没法编译类 B。
2. 编译后织入（Post-compile weaving）：也就是已经生成了 .class 文件，或已经打成 jar 包了，这种情况我们需要增强处理的话，就要用到编译后织入。
3. 类加载后织入（Load-time weaving）：指的是在加载类的时候进行织入，要实现这个时期的织入，有几种常见的方法

整体对比如下：

![Spring AOP和AspectJ对比](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/spring-d1dbe9d9-c55f-4293-8622-d9759064d613.png)Spring AOP和AspectJ对比

GitHub 上标星 7600+ 的开源知识库《[二哥的 Java 进阶之路open in new window](https://github.com/itwanger/toBeBetterJavaer)》第一版 PDF 终于来了！包括Java基础语法、数组&字符串、OOP、集合框架、Java IO、异常处理、Java 新特性、网络编程、NIO、并发编程、JVM等等，共计 32 万余字，可以说是通俗易懂、风趣幽默……详情戳：[太赞了，GitHub 上标星 7600+ 的 Java 教程open in new window](https://tobebetterjavaer.com/overview/)

微信搜 **沉默王二** 或扫描下方二维码关注二哥的原创公众号沉默王二，回复 **222** 即可免费领取。

![img](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/gongzhonghao.png)

## [#](https://tobebetterjavaer.com/sidebar/sanfene/spring.html#事务)事务

Spring 事务的本质其实就是数据库对事务的支持，没有数据库的事务支持，Spring 是无法提供事务功能的。Spring 只提供统一事务管理接口，具体实现都是由各数据库自己实现，数据库事务的提交和回滚是通过数据库自己的事务机制实现。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/spring.html#_23-spring-事务的种类)23.Spring 事务的种类？

Spring 支持`编程式事务`管理和`声明式`事务管理两种方式：

![Spring事务分类](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/spring-d3ee77fa-926d-4c39-91f8-a8b1544a9134.png)Spring事务分类

1. 编程式事务

编程式事务管理使用 TransactionTemplate，需要显式执行事务。

1. 声明式事务
2. 声明式事务管理建立在 AOP 之上的。其本质是通过 AOP 功能，对方法前后进行拦截，将事务处理的功能编织到拦截的方法中，也就是在目标方法开始之前启动一个事务，在执行完目标方法之后根据执行情况提交或者回滚事务
3. 优点是不需要在业务逻辑代码中掺杂事务管理的代码，只需在配置文件中做相关的事务规则声明或通过 @Transactional 注解的方式，便可以将事务规则应用到业务逻辑中，减少业务代码的污染。唯一不足地方是，最细粒度只能作用到方法级别，无法做到像编程式事务那样可以作用到代码块级别。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/spring.html#_24-spring-的事务隔离级别)24.Spring 的事务隔离级别？

Spring 的接口 TransactionDefinition 中定义了表示隔离级别的常量，当然其实主要还是对应数据库的事务隔离级别：

1. ISOLATION_DEFAULT：使用后端数据库默认的隔离界别，MySQL 默认可重复读，Oracle 默认读已提交。
2. ISOLATION_READ_UNCOMMITTED：读未提交
3. ISOLATION_READ_COMMITTED：读已提交
4. ISOLATION_REPEATABLE_READ：可重复读
5. ISOLATION_SERIALIZABLE：串行化

### [#](https://tobebetterjavaer.com/sidebar/sanfene/spring.html#_25-spring-的事务传播机制)25.Spring 的事务传播机制？

Spring 事务的传播机制说的是，当多个事务同时存在的时候——一般指的是多个事务方法相互调用时，Spring 如何处理这些事务的行为。

事务传播机制是使用简单的 ThreadLocal 实现的，所以，如果调用的方法是在新线程调用的，事务传播实际上是会失效的。

![7种事务传播机制](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/spring-a6e2a8dc-9771-4d8b-9d91-76ddee98af1a.png)7种事务传播机制

Spring 默认的事务传播行为是 PROPAFATION_REQUIRED，它适合绝大多数情况，如果多个 ServiceX#methodX()都工作在事务环境下（均被 Spring 事务增强），且程序中存在调用链 `Service1#method1()->Service2#method2()->Service3#method3()`，那么这 3 个服务类的三个方法通过 Spring 的事务传播机制都工作在同一个事务中。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/spring.html#_26-声明式事务实现原理了解吗)26.声明式事务实现原理了解吗？

就是通过 AOP/动态代理。

- **在 Bean 初始化阶段创建代理对象**：Spring 容器在初始化每个单例 bean 的时候，会遍历容器中的所有 BeanPostProcessor 实现类，并执行其 postProcessAfterInitialization 方法，在执行 AbstractAutoProxyCreator 类的 postProcessAfterInitialization 方法时会遍历容器中所有的切面，查找与当前实例化 bean 匹配的切面，这里会获取事务属性切面，查找@Transactional 注解及其属性值，然后根据得到的切面创建一个代理对象，默认是使用 JDK 动态代理创建代理，如果目标类是接口，则使用 JDK 动态代理，否则使用 Cglib。
- **在执行目标方法时进行事务增强操作**：当通过代理对象调用 Bean 方法的时候，会触发对应的 AOP 增强拦截器，声明式事务是一种环绕增强，对应接口为`MethodInterceptor`，事务增强对该接口的实现为`TransactionInterceptor`，类图如下：

![图片来源网易技术专栏](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/spring-97493c7f-c596-4e98-a6a8-dab254d6d1ab.png)图片来源网易技术专栏

事务拦截器`TransactionInterceptor`在`invoke`方法中，通过调用父类`TransactionAspectSupport`的`invokeWithinTransaction`方法进行事务处理，包括开启事务、事务提交、异常回滚。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/spring.html#_27-声明式事务在哪些情况下会失效)27.声明式事务在哪些情况下会失效？

![声明式事务的几种失效的情况](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/spring-381e4ec9-a235-4cfa-9b4d-518095a7502a.png)声明式事务的几种失效的情况

**1、@Transactional 应用在非 public 修饰的方法上**

如果 Transactional 注解应用在非 public 修饰的方法上，Transactional 将会失效。

是因为在 Spring AOP 代理时，TransactionInterceptor （事务拦截器）在目标方法执行前后进行拦截，DynamicAdvisedInterceptor（CglibAopProxy 的内部类）的 intercept 方法 或 JdkDynamicAopProxy 的 invoke 方法会间接调用 AbstractFallbackTransactionAttributeSource 的 **computeTransactionAttribute**方法，获取 Transactional 注解的事务配置信息。



```java
protected TransactionAttribute computeTransactionAttribute(Method method,
    Class<?> targetClass) {
        // Don't allow no-public methods as required.
        if (allowPublicMethodsOnly() && !Modifier.isPublic(method.getModifiers())) {
        return null;
}
```

此方法会检查目标方法的修饰符是否为 public，不是 public 则不会获取@Transactional 的属性配置信息。

**2、@Transactional 注解属性 propagation 设置错误**

- TransactionDefinition.PROPAGATION_SUPPORTS：如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。
- TransactionDefinition.PROPAGATION_NOT_SUPPORTED：以非事务方式运行，如果当前存在事务，则把当前事务挂起。
- TransactionDefinition.PROPAGATION_NEVER：以非事务方式运行，如果当前存在事务，则抛出异常。

**3、@Transactional 注解属性 rollbackFor 设置错误**

rollbackFor 可以指定能够触发事务回滚的异常类型。Spring 默认抛出了未检查 unchecked 异常（继承自 RuntimeException 的异常）或者 Error 才回滚事务，其他异常不会触发回滚事务。

![Spring默认支持的异常回滚](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/spring-04053b02-3264-4d7f-b868-560a0333f08d.png)Spring默认支持的异常回滚



```java
// 希望自定义的异常可以进行回滚
@Transactional(propagation= Propagation.REQUIRED,rollbackFor= MyException.class
```

若在目标方法中抛出的异常是 rollbackFor 指定的异常的子类，事务同样会回滚。

**4、同一个类中方法调用，导致@Transactional 失效**

开发中避免不了会对同一个类里面的方法调用，比如有一个类 Test，它的一个方法 A，A 再调用本类的方法 B（不论方法 B 是用 public 还是 private 修饰），但方法 A 没有声明注解事务，而 B 方法有。则外部调用方法 A 之后，方法 B 的事务是不会起作用的。这也是经常犯错误的一个地方。

那为啥会出现这种情况？其实这还是由于使用 Spring AOP 代理造成的，因为只有当事务方法被当前类以外的代码调用时，才会由 Spring 生成的代理对象来管理。



```java
 //@Transactional
     @GetMapping("/test")
     private Integer A() throws Exception {
         CityInfoDict cityInfoDict = new CityInfoDict();
         cityInfoDict.setCityName("2");
         /**
          * B 插入字段为 3的数据
          */
         this.insertB();
        /**
         * A 插入字段为 2的数据
         */
        int insert = cityInfoDictMapper.insert(cityInfoDict);
        return insert;
    }

    @Transactional()
    public Integer insertB() throws Exception {
        CityInfoDict cityInfoDict = new CityInfoDict();
        cityInfoDict.setCityName("3");
        cityInfoDict.setParentCityId(3);

        return cityInfoDictMapper.insert(cityInfoDict);
    }
```

这种情况是最常见的一种@Transactional 注解失效场景



```java
@Transactional
private Integer A() throws Exception {
    int insert = 0;
    try {
        CityInfoDict cityInfoDict = new CityInfoDict();
        cityInfoDict.setCityName("2");
        cityInfoDict.setParentCityId(2);
        /**
         * A 插入字段为 2的数据
         */
        insert = cityInfoDictMapper.insert(cityInfoDict);
        /**
         * B 插入字段为 3的数据
        */
        b.insertB();
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

如果 B 方法内部抛了异常，而 A 方法此时 try catch 了 B 方法的异常，那这个事务就不能正常回滚了，会抛出异常：



```java
org.springframework.transaction.UnexpectedRollbackException: Transaction rolled back because it has been marked as rollback-only
```

GitHub 上标星 7600+ 的开源知识库《[二哥的 Java 进阶之路open in new window](https://github.com/itwanger/toBeBetterJavaer)》第一版 PDF 终于来了！包括Java基础语法、数组&字符串、OOP、集合框架、Java IO、异常处理、Java 新特性、网络编程、NIO、并发编程、JVM等等，共计 32 万余字，可以说是通俗易懂、风趣幽默……详情戳：[太赞了，GitHub 上标星 7600+ 的 Java 教程open in new window](https://tobebetterjavaer.com/overview/)

微信搜 **沉默王二** 或扫描下方二维码关注二哥的原创公众号沉默王二，回复 **222** 即可免费领取。

![img](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/gongzhonghao.png)

## [#](https://tobebetterjavaer.com/sidebar/sanfene/spring.html#mvc)MVC

### [#](https://tobebetterjavaer.com/sidebar/sanfene/spring.html#_28-spring-mvc-的核心组件)28.Spring MVC 的核心组件？

1. **DispatcherServlet**：前置控制器，是整个流程控制的**核心**，控制其他组件的执行，进行统一调度，降低组件之间的耦合性，相当于总指挥。
2. **Handler**：处理器，完成具体的业务逻辑，相当于 Servlet 或 Action。
3. **HandlerMapping**：DispatcherServlet 接收到请求之后，通过 HandlerMapping 将不同的请求映射到不同的 Handler。
4. **HandlerInterceptor**：处理器拦截器，是一个接口，如果需要完成一些拦截处理，可以实现该接口。
5. **HandlerExecutionChain**：处理器执行链，包括两部分内容：Handler 和 HandlerInterceptor（系统会有一个默认的 HandlerInterceptor，如果需要额外设置拦截，可以添加拦截器）。
6. **HandlerAdapter**：处理器适配器，Handler 执行业务方法之前，需要进行一系列的操作，包括表单数据的验证、数据类型的转换、将表单数据封装到 JavaBean 等，这些操作都是由 HandlerApater 来完成，开发者只需将注意力集中业务逻辑的处理上，DispatcherServlet 通过 HandlerAdapter 执行不同的 Handler。
7. **ModelAndView**：装载了模型数据和视图信息，作为 Handler 的处理结果，返回给 DispatcherServlet。
8. **ViewResolver**：视图解析器，DispatcheServlet 通过它将逻辑视图解析为物理视图，最终将渲染结果响应给客户端。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/spring.html#_29-spring-mvc-的工作流程)29.Spring MVC 的工作流程？

![Spring MVC的工作流程](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/spring-e29a122b-db07-48b8-8289-7251032e87a1.png)Spring MVC的工作流程

1. 客户端向服务端发送一次请求，这个请求会先到前端控制器 DispatcherServlet(也叫中央控制器)。
2. DispatcherServlet 接收到请求后会调用 HandlerMapping 处理器映射器。由此得知，该请求该由哪个 Controller 来处理（并未调用 Controller，只是得知）
3. DispatcherServlet 调用 HandlerAdapter 处理器适配器，告诉处理器适配器应该要去执行哪个 Controller
4. HandlerAdapter 处理器适配器去执行 Controller 并得到 ModelAndView(数据和视图)，并层层返回给 DispatcherServlet
5. DispatcherServlet 将 ModelAndView 交给 ViewReslover 视图解析器解析，然后返回真正的视图。
6. DispatcherServlet 将模型数据填充到视图中
7. DispatcherServlet 将结果响应给客户端

**Spring MVC** 虽然整体流程复杂，但是实际开发中很简单，大部分的组件不需要开发人员创建和管理，只需要通过配置文件的方式完成配置即可，真正需要开发人员进行处理的只有 **Handler（Controller）** 、**View** 、**Model**。

当然我们现在大部分的开发都是前后端分离，Restful 风格接口，后端只需要返回 Json 数据就行了。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/spring.html#_30-springmvc-restful-风格的接口的流程是什么样的呢)30.SpringMVC Restful 风格的接口的流程是什么样的呢？

PS:这是一道全新的八股，毕竟 ModelAndView 这种方式应该没人用了吧？现在都是前后端分离接口，八股也该更新换代了。

我们都知道 Restful 接口，响应格式是 json，这就用到了一个常用注解：**@ResponseBody**



```java
    @GetMapping("/user")
    @ResponseBody
    public User user(){
        return new User(1,"张三");
    }
```

加入了这个注解后，整体的流程上和使用 ModelAndView 大体上相同，但是细节上有一些不同：

![Spring MVC Restful请求响应示意图](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/spring-2da963a0-5da9-4b3a-aafd-fd8dbc7e1807.png)Spring MVC Restful请求响应示意图

1. 客户端向服务端发送一次请求，这个请求会先到前端控制器 DispatcherServlet

2. DispatcherServlet 接收到请求后会调用 HandlerMapping 处理器映射器。由此得知，该请求该由哪个 Controller 来处理

3. DispatcherServlet 调用 HandlerAdapter 处理器适配器，告诉处理器适配器应该要去执行哪个 Controller

4. Controller 被封装成了 ServletInvocableHandlerMethod，HandlerAdapter 处理器适配器去执行 invokeAndHandle 方法，完成对 Controller 的请求处理

5. HandlerAdapter 执行完对 Controller 的请求，会调用 HandlerMethodReturnValueHandler 去处理返回值，主要的过程：

   5.1. 调用 RequestResponseBodyMethodProcessor，创建 ServletServerHttpResponse（Spring 对原生 ServerHttpResponse 的封装）实例

   5.2.使用 HttpMessageConverter 的 write 方法，将返回值写入 ServletServerHttpResponse 的 OutputStream 输出流中

   5.3.在写入的过程中，会使用 JsonGenerator（默认使用 Jackson 框架）对返回值进行 Json 序列化

6. 执行完请求后，返回的 ModealAndView 为 null，ServletServerHttpResponse 里也已经写入了响应，所以不用关心 View 的处理

GitHub 上标星 7600+ 的开源知识库《[二哥的 Java 进阶之路open in new window](https://github.com/itwanger/toBeBetterJavaer)》第一版 PDF 终于来了！包括Java基础语法、数组&字符串、OOP、集合框架、Java IO、异常处理、Java 新特性、网络编程、NIO、并发编程、JVM等等，共计 32 万余字，可以说是通俗易懂、风趣幽默……详情戳：[太赞了，GitHub 上标星 7600+ 的 Java 教程open in new window](https://tobebetterjavaer.com/overview/)

微信搜 **沉默王二** 或扫描下方二维码关注二哥的原创公众号沉默王二，回复 **222** 即可免费领取。

![img](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/gongzhonghao.png)

## [#](https://tobebetterjavaer.com/sidebar/sanfene/spring.html#spring-boot)Spring Boot

### [#](https://tobebetterjavaer.com/sidebar/sanfene/spring.html#_31-介绍一下-springboot-有哪些优点)31.介绍一下 SpringBoot，有哪些优点？

Spring Boot 基于 Spring 开发，Spirng Boot 本身并不提供 Spring 框架的核心特性以及扩展功能，只是用于快速、敏捷地开发新一代基于 Spring 框架的应用程序。它并不是用来替代 Spring 的解决方案，而是和 Spring 框架紧密结合用于提升 Spring 开发者体验的工具。

![SpringBoot图标](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/spring-d9164ee6-5c86-4313-8fd9-efb9acfa5f0b.png)SpringBoot图标

Spring Boot 以`约定大于配置`核心思想开展工作，相比 Spring 具有如下优势：

1. Spring Boot 可以快速创建独立的 Spring 应用程序。
2. Spring Boot 内嵌了如 Tomcat，Jetty 和 Undertow 这样的容器，也就是说可以直接跑起来，用不着再做部署工作了。
3. Spring Boot 无需再像 Spring 一样使用一堆繁琐的 xml 文件配置。
4. Spring Boot 可以自动配置(核心)Spring。SpringBoot 将原有的 XML 配置改为 Java 配置，将 bean 注入改为使用注解注入的方式(@Autowire)，并将多个 xml、properties 配置浓缩在一个 appliaction.yml 配置文件中。
5. Spring Boot 提供了一些现有的功能，如量度工具，表单数据验证以及一些外部配置这样的一些第三方功能。
6. Spring Boot 可以快速整合常用依赖（开发库，例如 spring-webmvc、jackson-json、validation-api 和 tomcat 等），提供的 POM 可以简化 Maven 的配置。当我们引入核心依赖时，SpringBoot 会自引入其他依赖。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/spring.html#_32-springboot-自动配置原理了解吗)32.SpringBoot 自动配置原理了解吗？

SpringBoot 开启自动配置的注解是`@EnableAutoConfiguration` ，启动类上的注解`@SpringBootApplication`是一个复合注解，包含了@EnableAutoConfiguration：

![SpringBoot自动配置原理](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/spring-df77ee15-2ff0-4ec7-8e65-e4ebb8ba88f1.png)SpringBoot自动配置原理

- `EnableAutoConfiguration` 只是一个简单的注解，自动装配核心功能的实现实际是通过 `AutoConfigurationImportSelector`类

  

  ```java
  @AutoConfigurationPackage //将main同级的包下的所有组件注册到容器中
  @Import({AutoConfigurationImportSelector.class}) //加载自动装配类 xxxAutoconfiguration
  public @interface EnableAutoConfiguration {
      String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";
  
      Class<?>[] exclude() default {};
  
      String[] excludeName() default {};
  }
  ```

- `AutoConfigurationImportSelector`实现了`ImportSelector`接口，这个接口的作用就是收集需要导入的配置类，配合`@Import(）`就可以将相应的类导入到 Spring 容器中

- 获取注入类的方法是 selectImports()，它实际调用的是`getAutoConfigurationEntry`，这个方法是获取自动装配类的关键，主要流程可以分为这么几步：

  1. 获取注解的属性，用于后面的排除
  2. **获取所有需要自动装配的配置类的路径**：这一步是最关键的，从 META-INF/spring.factories 获取自动配置类的路径
  3. 去掉重复的配置类和需要排除的重复类，把需要自动加载的配置类的路径存储起来



```java
    protected AutoConfigurationImportSelector.AutoConfigurationEntry getAutoConfigurationEntry(AnnotationMetadata annotationMetadata) {
        if (!this.isEnabled(annotationMetadata)) {
            return EMPTY_ENTRY;
        } else {
            //1.获取到注解的属性
            AnnotationAttributes attributes = this.getAttributes(annotationMetadata);
            //2.获取需要自动装配的所有配置类，读取META-INF/spring.factories，获取自动配置类路径
            List<String> configurations = this.getCandidateConfigurations(annotationMetadata, attributes);
            //3.1.移除重复的配置
            configurations = this.removeDuplicates(configurations);
            //3.2.处理需要排除的配置
            Set<String> exclusions = this.getExclusions(annotationMetadata, attributes);
            this.checkExcludedClasses(configurations, exclusions);
            configurations.removeAll(exclusions);
            configurations = this.getConfigurationClassFilter().filter(configurations);
            this.fireAutoConfigurationImportEvents(configurations, exclusions);
            return new AutoConfigurationImportSelector.AutoConfigurationEntry(configurations, exclusions);
        }
    }
```

### [#](https://tobebetterjavaer.com/sidebar/sanfene/spring.html#_33-如何自定义一个-springboot-srarter)33.如何自定义一个 SpringBoot Srarter?

知道了自动配置原理，创建一个自定义 SpringBoot Starter 也很简单。

1. 创建一个项目，命名为 demo-spring-boot-starter，引入 SpringBoot 相关依赖



```java
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <optional>true</optional>
        </dependency>
```

1. 编写配置文件

   这里定义了属性配置的前缀

   

   ```java
   @ConfigurationProperties(prefix = "hello")
   public class HelloProperties {
   
       private String name;
   
       //省略getter、setter
   }
   ```

2. 自动装配

   创建自动配置类 HelloPropertiesConfigure

   

   ```java
   @Configuration
   @EnableConfigurationProperties(HelloProperties.class)
   public class HelloPropertiesConfigure {
   }
   ```

3. 配置自动类

   在`/resources/META-INF/spring.factories`文件中添加自动配置类路径

   

   ```java
   org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
     cn.fighter3.demo.starter.configure.HelloPropertiesConfigure
   ```

4. 测试

   - 创建一个工程，引入自定义 starter 依赖

     

     ```java
             <dependency>
                 <groupId>cn.fighter3</groupId>
                 <artifactId>demo-spring-boot-starter</artifactId>
                 <version>0.0.1-SNAPSHOT</version>
             </dependency>
     ```

   - 在配置文件里添加配置

     

     ```text
     hello.name=张三
     ```

   - 测试类

     

     ```java
     @RunWith(SpringRunner.class)
     @SpringBootTest
     public class HelloTest {
         @Autowired
         HelloProperties helloProperties;
     
         @Test
         public void hello(){
             System.out.println("你好，"+helloProperties.getName());
         }
     }
     ```

- 运行结果

![运行结果](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/spring-3ff3cc21-6a56-434b-a89e-d2b55d558bd6.png)运行结果

至此，随手写的一个自定义 SpringBoot-Starter 就完成了，虽然比较简单，但是完成了主要的自动装配的能力。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/spring.html#_34-springboot-启动原理)34.Springboot 启动原理？

SpringApplication 这个类主要做了以下四件事情：

1. 推断应用的类型是普通的项目还是 Web 项目
2. 查找并加载所有可用初始化器 ， 设置到 initializers 属性中
3. 找出所有的应用程序监听器，设置到 listeners 属性中
4. 推断并设置 main 方法的定义类，找到运行的主类

SpringBoot 启动大致流程如下 ：

![SpringBoot 启动大致流程-图片来源网络](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/spring-68744556-a1ba-4e1f-a092-1582875f0da6.png)SpringBoot 启动大致流程-图片来源网络

## [#](https://tobebetterjavaer.com/sidebar/sanfene/spring.html#spring-cloud)Spring Cloud

### [#](https://tobebetterjavaer.com/sidebar/sanfene/spring.html#_35-对-springcloud-了解多少)35.对 SpringCloud 了解多少？

SpringCloud 是 Spring 官方推出的微服务治理框架。

![Spring Cloud Netfilx核心组件-来源参考[2]](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/spring-92ab53d5-f303-4fc5-bd26-e62cefe374b3.png)Spring Cloud Netfilx核心组件-来源参考[2]

> **什么是微服务？**

1. 2014 年 **Martin Fowler** 提出的一种新的架构形式。微服务架构是一种**架构模式**，提倡将单一应用程序划分成一组小的服务，服务之间相互协调，互相配合，为用户提供最终价值。每个服务运行在其独立的进程中，服务与服务之间采用轻量级的通信机制(如 HTTP 或 Dubbo)互相协作，每个服务都围绕着具体的业务进行构建，并且能够被独立的部署到生产环境中，另外，应尽量避免统一的，集中式的服务管理机制，对具体的一个服务而言，应根据业务上下文，选择合适的语言、工具(如 Maven)对其进行构建。
2. 微服务化的核心就是将传统的一站式应用，根据业务拆分成一个一个的服务，彻底地去耦合，每一个微服务提供单个业务功能的服务，一个服务做一件事情，从技术角度看就是一种小而独立的处理过程，类似进程的概念，能够自行单独启动或销毁，拥有自己独立的数据库。

> **微服务架构主要要解决哪些问题？**

1. 服务很多，客户端怎么访问，如何提供对外网关?
2. 这么多服务，服务之间如何通信? HTTP 还是 RPC?
3. 这么多服务，如何治理? 服务的注册和发现。
4. 服务挂了怎么办？熔断机制。

> **有哪些主流微服务框架？**

1. Spring Cloud Netflix
2. Spring Cloud Alibaba
3. SpringBoot + Dubbo + ZooKeeper

> **SpringCloud 有哪些核心组件？**

![SpringCloud](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/spring-2b988a72-0739-4fed-b271-eaf12589444f.png)SpringCloud

PS:微服务后面有机会再扩展，其实面试一般都是结合项目去问。

------

*没有什么使我停留——除了目的，纵然岸旁有玫瑰、有绿荫、有宁静的港湾，我是不系之舟*。



# 分布式  Interview

## 基础

![img](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/mysql-1992b6dd-1c1d-4b8b-b98a-8407e8c51ff9.jpg)

作为 SQL Boy，基础部分不会有人不会吧？面试也不怎么问，基础掌握不错的小伙伴可以**跳过**这一部分。当然，可能会现场写一些 SQL 语句，SQ 语句可以通过牛客、LeetCode、LintCode 之类的网站来练习。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#_1-什么是内连接、外连接、交叉连接、笛卡尔积呢)1. 什么是内连接、外连接、交叉连接、笛卡尔积呢？

- 内连接（inner join）：取得两张表中满足存在连接匹配关系的记录。
- 外连接（outer join）：不只取得两张表中满足存在连接匹配关系的记录，还包括某张表（或两张表）中不满足匹配关系的记录。
- 交叉连接（cross join）：显示两张表所有记录一一对应，没有匹配关系进行筛选，它是笛卡尔积在 SQL 中的实现，如果 A 表有 m 行，B 表有 n 行，那么 A 和 B 交叉连接的结果就有 m*n 行。
- 笛卡尔积：是数学中的一个概念，例如集合 A={a,b}，集合 B={1,2,3}，那么 A✖️B=`{<a,o>,<a,1>,<a,2>,<b,0>,<b,1>,<b,2>,}`。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#_2-那-mysql-的内连接、左连接、右连接有有什么区别)2. 那 MySQL 的内连接、左连接、右连接有有什么区别？

MySQL 的连接主要分为内连接和外连接，外连接常用的有左连接、右连接。

![img](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/mysql-fcdaad5f-c50e-4834-9f9a-0b676cc6be83.jpg)

MySQL-joins-来源菜鸟教程

- inner join 内连接，在两张表进行连接查询时，只保留两张表中完全匹配的结果集
- left join 在两张表进行连接查询时，会返回左表所有的行，即使在右表中没有匹配的记录。
- right join 在两张表进行连接查询时，会返回右表所有的行，即使在左表中没有匹配的记录。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#_3-说一下数据库的三大范式)3.说一下数据库的三大范式？

![数据库三范式](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/mysql-16e74a6b-a42a-464e-9b10-0252ee7ecc6e.jpg)数据库三范式

- 第一范式：数据表中的每一列（每个字段）都不可以再拆分。例如用户表，用户地址还可以拆分成国家、省份、市，这样才是符合第一范式的。
- 第二范式：在第一范式的基础上，非主键列完全依赖于主键，而不能是依赖于主键的一部分。例如订单表里，存储了商品信息（商品价格、商品类型），那就需要把商品 ID 和订单 ID 作为联合主键，才满足第二范式。
- 第三范式：在满足第二范式的基础上，表中的非主键只依赖于主键，而不依赖于其他非主键。例如订单表，就不能存储用户信息（姓名、地址）。

![你设计遵守范式吗？](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/mysql-5351d57d-1cea-49c8-b6bb-b7d49abb4427.jpg)你设计遵守范式吗？

三大范式的作用是为了控制数据库的冗余，是对空间的节省，实际上，一般互联网公司的设计都是反范式的，通过冗余一些数据，避免跨表跨库，利用空间换时间，提高性能。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#_4-varchar-与-char-的区别)4.varchar 与 char 的区别？

![varchar](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/mysql-40f42d59-a295-4543-8a03-43925da4d6d9.jpg)varchar

**char**：

- char 表示定长字符串，长度是固定的；
- 如果插入数据的长度小于 char 的固定长度时，则用空格填充；
- 因为长度固定，所以存取速度要比 varchar 快很多，甚至能快 50%，但正因为其长度固定，所以会占据多余的空间，是空间换时间的做法；
- 对于 char 来说，最多能存放的字符个数为 255，和编码无关

**varchar**：

- varchar 表示可变长字符串，长度是可变的；
- 插入的数据是多长，就按照多长来存储；
- varchar 在存取方面与 char 相反，它存取慢，因为长度不固定，但正因如此，不占据多余的空间，是时间换空间的做法；
- 对于 varchar 来说，最多能存放的字符个数为 65532

日常的设计，对于长度相对固定的字符串，可以使用 char，对于长度不确定的，使用 varchar 更合适一些。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#_5-blob-和-text-有什么区别)5.blob 和 text 有什么区别？

- blob 用于存储二进制数据，而 text 用于存储大字符串。
- blob 没有字符集，text 有一个字符集，并且根据字符集的校对规则对值进行排序和比较

### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#_6-datetime-和-timestamp-的异同)6.DATETIME 和 TIMESTAMP 的异同？

**相同点**：

1. 两个数据类型存储时间的表现格式一致。均为 `YYYY-MM-DD HH:MM:SS`
2. 两个数据类型都包含「日期」和「时间」部分。
3. 两个数据类型都可以存储微秒的小数秒（秒后 6 位小数秒）

**区别**：

![img](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/mysql-d94e5e1c-2614-4b8b-acdb-efb333032854.jpg)

DATETIME 和 TIMESTAMP 的区别

1. **日期范围**：DATETIME 的日期范围是 `1000-01-01 00:00:00.000000` 到 `9999-12-31 23:59:59.999999`；TIMESTAMP 的时间范围是`1970-01-01 00:00:01.000000` UTC `到 ``2038-01-09 03:14:07.999999` UTC
2. **存储空间**：DATETIME 的存储空间为 8 字节；TIMESTAMP 的存储空间为 4 字节
3. **时区相关**：DATETIME 存储时间与时区无关；TIMESTAMP 存储时间与时区有关，显示的值也依赖于时区
4. **默认值**：DATETIME 的默认值为 null；TIMESTAMP 的字段默认不为空(not null)，默认值为当前时间(CURRENT_TIMESTAMP)

### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#_7-mysql-中-in-和-exists-的区别)7.MySQL 中 in 和 exists 的区别？

MySQL 中的 in 语句是把外表和内表作 hash 连接，而 exists 语句是对外表作 loop 循环，每次 loop 循环再对内表进行查询。我们可能认为 exists 比 in 语句的效率要高，这种说法其实是不准确的，要区分情景：

1. 如果查询的两个表大小相当，那么用 in 和 exists 差别不大。
2. 如果两个表中一个较小，一个是大表，则子查询表大的用 exists，子查询表小的用 in。
3. not in 和 not exists：如果查询语句使用了 not in，那么内外表都进行全表扫描，没有用到索引；而 not extsts 的子查询依然能用到表上的索引。所以无论那个表大，用 not exists 都比 not in 要快。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#_8-mysql-里记录货币用什么字段类型比较好)8.MySQL 里记录货币用什么字段类型比较好？

货币在数据库中 MySQL 常用 Decimal 和 Numric 类型表示，这两种类型被 MySQL 实现为同样的类型。他们被用于保存与货币有关的数据。

例如 salary DECIMAL(9,2)，9(precision)代表将被用于存储值的总的小数位数，而 2(scale)代表将被用于存储小数点后的位数。存储在 salary 列中的值的范围是从-9999999.99 到 9999999.99。

DECIMAL 和 NUMERIC 值作为字符串存储，而不是作为二进制浮点数，以便保存那些值的小数精度。

之所以不使用 float 或者 double 的原因：因为 float 和 double 是以二进制存储的，所以有一定的误差。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#_9-mysql-怎么存储-emoji😊)9.MySQL 怎么存储 emoji😊?

MySQL 可以直接使用字符串存储 emoji。

但是需要注意的，utf8 编码是不行的，MySQL 中的 utf8 是阉割版的 utf8，它最多只用 3 个字节存储字符，所以存储不了表情。那该怎么办？

需要使用 utf8mb4 编码。



```text
alter table blogs modify content text CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci not null;
```

### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#_10-drop、delete-与-truncate-的区别)10.drop、delete 与 truncate 的区别？

三者都表示删除，但是三者有一些差别：

| delete   | truncate                                 | drop                           |
| -------- | ---------------------------------------- | ------------------------------ |
| 类型     | 属于 DML                                 | 属于 DDL                       |
| 回滚     | 可回滚                                   | 不可回滚                       |
| 删除内容 | 表结构还在，删除表的全部或者一部分数据行 | 表结构还在，删除表中的所有数据 |
| 删除速度 | 删除速度慢，需要逐行删除                 | 删除速度快                     |

因此，在不再需要一张表的时候，用 drop；在想删除部分数据行时候，用 delete；在保留表而删除所有数据的时候用 truncate。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#_11-union-与-union-all-的区别)11.UNION 与 UNION ALL 的区别？

- 如果使用 UNION，会在表链接后筛选掉重复的记录行
- 如果使用 UNION ALL，不会合并重复的记录行
- 从效率上说，UNION ALL 要比 UNION 快很多，如果合并没有刻意要删除重复行，那么就使用 UNION All

### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#_12-count-1-、count-与-count-列名-的区别)12.count(1)、count(*) 与 count(列名) 的区别？

![三种计数方式](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/mysql-2c754ee2-20c4-4c03-9db0-22c7c9eb7f01.jpg)三种计数方式

**执行效果**：

- count(*)包括了所有的列，相当于行数，在统计结果的时候，不会忽略列值为 NULL
- count(1)包括了忽略所有列，用 1 代表代码行，在统计结果的时候，不会忽略列值为 NULL
- count(列名)只包括列名那一列，在统计结果的时候，会忽略列值为空（这里的空不是只空字符串或者 0，而是表示 null）的计数，即某个字段值为 NULL 时，不统计。

**执行速度**：

- 列名为主键，count(列名)会比 count(1)快
- 列名不为主键，count(1)会比 count(列名)快
- 如果表多个列并且没有主键，则 count（1） 的执行效率优于 count（*）
- 如果有主键，则 select count（主键）的执行效率是最优的
- 如果表只有一个字段，则 select count（*）最优。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#_13-一条-sql-查询语句的执行顺序)13.一条 SQL 查询语句的执行顺序？

![查询语句执行顺序](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/mysql-47ddea92-cf8f-49c4-ab2e-69a829ff1be2.jpg)查询语句执行顺序

1. **FROM**：对 FROM 子句中的左表<left_table>和右表<right_table>执行笛卡儿积（Cartesianproduct），产生虚拟表 VT1
2. **ON**：对虚拟表 VT1 应用 ON 筛选，只有那些符合<join_condition>的行才被插入虚拟表 VT2 中
3. **JOIN**：如果指定了 OUTER JOIN（如 LEFT OUTER JOIN、RIGHT OUTER JOIN），那么保留表中未匹配的行作为外部行添加到虚拟表 VT2 中，产生虚拟表 VT3。如果 FROM 子句包含两个以上表，则对上一个连接生成的结果表 VT3 和下一个表重复执行步骤 1）～步骤 3），直到处理完所有的表为止
4. **WHERE**：对虚拟表 VT3 应用 WHERE 过滤条件，只有符合<where_condition>的记录才被插入虚拟表 VT4 中
5. **GROUP BY**：根据 GROUP BY 子句中的列，对 VT4 中的记录进行分组操作，产生 VT5
6. **CUBE|ROLLUP**：对表 VT5 进行 CUBE 或 ROLLUP 操作，产生表 VT6
7. **HAVING**：对虚拟表 VT6 应用 HAVING 过滤器，只有符合<having_condition>的记录才被插入虚拟表 VT7 中。
8. **SELECT**：第二次执行 SELECT 操作，选择指定的列，插入到虚拟表 VT8 中
9. **DISTINCT**：去除重复数据，产生虚拟表 VT9
10. **ORDER BY**：将虚拟表 VT9 中的记录按照<order_by_list>进行排序操作，产生虚拟表 VT10。11）
11. **LIMIT**：取出指定行的记录，产生虚拟表 VT11，并返回给查询用户

## [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#数据库架构)数据库架构

### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#_14-说说-mysql-的基础架构)14.说说 MySQL 的基础架构?

![img](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/mysql-77626fdb-d2b0-4256-a483-d1c60e68d8ec.jpg)

MySQL 逻辑架构图主要分三层：

- 客户端：最上层的服务并不是 MySQL 所独有的，大多数基于网络的客户端/服务器的工具或者服务都有类似的架构。比如连接处理、授权认证、安全等等。
- Server 层：大多数 MySQL 的核心服务功能都在这一层，包括查询解析、分析、优化、缓存以及所有的内置函数（例如，日期、时间、数学和加密函数），所有跨存储引擎的功能都在这一层实现：存储过程、触发器、视图等。
- 存储引擎层：第三层包含了存储引擎。存储引擎负责 MySQL 中数据的存储和提取。Server 层通过 API 与存储引擎进行通信。这些接口屏蔽了不同存储引擎之间的差异，使得这些差异对上层的查询过程透明。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#_15-一条-sql-查询语句在-mysql-中如何执行的)15.一条 SQL 查询语句在 MySQL 中如何执行的？

- 先检查该语句`是否有权限`，如果没有权限，直接返回错误信息，如果有权限会先查询缓存 (MySQL8.0 版本以前)。
- 如果没有缓存，分析器进行`语法分析`，提取 sql 语句中 select 等关键元素，然后判断 sql 语句是否有语法错误，比如关键词是否正确等等。
- 语法解析之后，MySQL 的服务器会对查询的语句进行优化，确定执行的方案。
- 完成查询优化后，按照生成的执行计划`调用数据库引擎接口`，返回执行结果。

GitHub 上标星 7600+ 的开源知识库《[二哥的 Java 进阶之路open in new window](https://github.com/itwanger/toBeBetterJavaer)》第一版 PDF 终于来了！包括Java基础语法、数组&字符串、OOP、集合框架、Java IO、异常处理、Java 新特性、网络编程、NIO、并发编程、JVM等等，共计 32 万余字，可以说是通俗易懂、风趣幽默……详情戳：[太赞了，GitHub 上标星 7600+ 的 Java 教程open in new window](https://tobebetterjavaer.com/overview/)

微信搜 **沉默王二** 或扫描下方二维码关注二哥的原创公众号沉默王二，回复 **222** 即可免费领取。

![img](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/gongzhonghao.png)

## [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#存储引擎)存储引擎

### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#_16-mysql-有哪些常见存储引擎)16.MySQL 有哪些常见存储引擎？

![主要存储引擎](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/mysql-db586190-6d1f-49ef-a5b5-496c13e7050d.jpg)主要存储引擎

主要存储引擎以及功能如下：

| 功能         | MylSAM | MEMORY | InnoDB |
| ------------ | ------ | ------ | ------ |
| 存储限制     | 256TB  | RAM    | 64TB   |
| 支持事务     | No     | No     | Yes    |
| 支持全文索引 | Yes    | No     | Yes    |
| 支持树索引   | Yes    | Yes    | Yes    |
| 支持哈希索引 | No     | Yes    | Yes    |
| 支持数据缓存 | No     | N/A    | Yes    |
| 支持外键     | No     | No     | Yes    |

MySQL5.5 之前，默认存储引擎是 MylSAM，5.5 之后变成了 InnoDB。

> InnoDB 支持的哈希索引是自适应的，InnoDB 会根据表的使用情况自动为表生成哈希索引，不能人为干预是否在一张表中生成哈希索引。

> MySQL 5.6 开始 InnoDB 支持全文索引。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#_17-那存储引擎应该怎么选择)17.那存储引擎应该怎么选择？

大致上可以这么选择：

- 大多数情况下，使用默认的 InnoDB 就够了。如果要提供提交、回滚和恢复的事务安全（ACID 兼容）能力，并要求实现并发控制，InnoDB 就是比较靠前的选择了。
- 如果数据表主要用来插入和查询记录，则 MyISAM 引擎提供较高的处理效率。
- 如果只是临时存放数据，数据量不大，并且不需要较高的数据安全性，可以选择将数据保存在内存的 MEMORY 引擎中，MySQL 中使用该引擎作为临时表，存放查询的中间结果。

使用哪一种引擎可以根据需要灵活选择，因为存储引擎是基于表的，所以一个数据库中多个表可以使用不同的引擎以满足各种性能和实际需求。使用合适的存储引擎将会提高整个数据库的性能。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#_18-innodb-和-mylsam-主要有什么区别)18.InnoDB 和 MylSAM 主要有什么区别？

PS:MySQL8.0 都开始慢慢流行了，如果不是面试，MylSAM 其实可以不用怎么了解。

![InnoDB 和 MylSAM 主要有什么区别](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/mysql-b7aa040e-a3a7-4133-8c43-baccc3c8d012.jpg)InnoDB 和 MylSAM 主要有什么区别

**1.  存储结构**：每个 MyISAM 在磁盘上存储成三个文件；InnoDB 所有的表都保存在同一个数据文件中（也可能是多个文件，或者是独立的表空间文件），InnoDB 表的大小只受限于操作系统文件的大小，一般为 2GB。

**2. 事务支持**：MyISAM 不提供事务支持；InnoDB 提供事务支持事务，具有事务(commit)、回滚(rollback)和崩溃修复能力(crash recovery capabilities)的事务安全特性。

**3  最小锁粒度**：MyISAM 只支持表级锁，更新时会锁住整张表，导致其它查询和更新都会被阻塞 InnoDB 支持行级锁。

**4. 索引类型**：MyISAM 的索引为非聚簇索引，数据结构是 B 树；InnoDB 的索引是聚簇索引，数据结构是 B+树。

**5.  主键必需**：MyISAM 允许没有任何索引和主键的表存在；InnoDB 如果没有设定主键或者非空唯一索引，**就会自动生成一个 6 字节的主键(用户不可见)**，数据是主索引的一部分，附加索引保存的是主索引的值。

**6. 表的具体行数**：MyISAM 保存了表的总行数，如果 select count(*) from table;会直接取出出该值; InnoDB 没有保存表的总行数，如果使用 select count(*) from table；就会遍历整个表;但是在加了 wehre 条件后，MyISAM 和 InnoDB 处理的方式都一样。

**7.  外键支持**：MyISAM 不支持外键；InnoDB 支持外键。

GitHub 上标星 7600+ 的开源知识库《[二哥的 Java 进阶之路open in new window](https://github.com/itwanger/toBeBetterJavaer)》第一版 PDF 终于来了！包括Java基础语法、数组&字符串、OOP、集合框架、Java IO、异常处理、Java 新特性、网络编程、NIO、并发编程、JVM等等，共计 32 万余字，可以说是通俗易懂、风趣幽默……详情戳：[太赞了，GitHub 上标星 7600+ 的 Java 教程open in new window](https://tobebetterjavaer.com/overview/)

微信搜 **沉默王二** 或扫描下方二维码关注二哥的原创公众号沉默王二，回复 **222** 即可免费领取。

![img](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/gongzhonghao.png)

## [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#日志)日志

### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#_19-mysql-日志文件有哪些-分别介绍下作用)19.MySQL 日志文件有哪些？分别介绍下作用？

![MySQL 主要日志](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/mysql-c0ef6e68-bb33-48fc-b3a2-b9cdadd8e403.jpg)MySQL 主要日志

MySQL 日志文件有很多，包括 ：

- **错误日志**（error log）：错误日志文件对 MySQL 的启动、运行、关闭过程进行了记录，能帮助定位 MySQL 问题。
- **慢查询日志**（slow query log）：慢查询日志是用来记录执行时间超过 long_query_time 这个变量定义的时长的查询语句。通过慢查询日志，可以查找出哪些查询语句的执行效率很低，以便进行优化。
- **一般查询日志**（general log）：一般查询日志记录了所有对 MySQL 数据库请求的信息，无论请求是否正确执行。
- **二进制日志**（bin log）：关于二进制日志，它记录了数据库所有执行的 DDL 和 DML 语句（除了数据查询语句 select、show 等），以事件形式记录并保存在二进制文件中。

还有两个 InnoDB 存储引擎特有的日志文件：

- **重做日志**（redo log）：重做日志至关重要，因为它们记录了对于 InnoDB 存储引擎的事务日志。
- **回滚日志**（undo log）：回滚日志同样也是 InnoDB 引擎提供的日志，顾名思义，回滚日志的作用就是对数据进行回滚。当事务对数据库进行修改，InnoDB 引擎不仅会记录 redo log，还会生成对应的 undo log 日志；如果事务执行失败或调用了 rollback，导致事务需要回滚，就可以利用 undo log 中的信息将数据回滚到修改之前的样子。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#_20-binlog-和-redo-log-有什么区别)20.binlog 和 redo log 有什么区别？

- bin log 会记录所有与数据库有关的日志记录，包括 InnoDB、MyISAM 等存储引擎的日志，而 redo log 只记 InnoDB 存储引擎的日志。
- 记录的内容不同，bin log 记录的是关于一个事务的具体操作内容，即该日志是逻辑日志。而 redo log 记录的是关于每个页（Page）的更改的物理情况。
- 写入的时间不同，bin log 仅在事务提交前进行提交，也就是只写磁盘一次。而在事务进行的过程中，却不断有 redo ertry 被写入 redo log 中。
- 写入的方式也不相同，redo log 是循环写入和擦除，bin log 是追加写入，不会覆盖已经写的文件。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#_21-一条更新语句怎么执行的了解吗)21.一条更新语句怎么执行的了解吗？

更新语句的执行是 Server 层和引擎层配合完成，数据除了要写入表中，还要记录相应的日志。

![update 执行](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/mysql-812fb038-39de-4204-ac9f-93d8b7448a18.jpg)update 执行

1. 执行器先找引擎获取 ID=2 这一行。ID 是主键，存储引擎检索数据，找到这一行。如果 ID=2 这一行所在的数据页本来就在内存中，就直接返回给执行器；否则，需要先从磁盘读入内存，然后再返回。
2. 执行器拿到引擎给的行数据，把这个值加上 1，比如原来是 N，现在就是 N+1，得到新的一行数据，再调用引擎接口写入这行新数据。
3. 引擎将这行新数据更新到内存中，同时将这个更新操作记录到 redo log 里面，此时 redo log 处于 prepare 状态。然后告知执行器执行完成了，随时可以提交事务。
4. 执行器生成这个操作的 binlog，并把 binlog 写入磁盘。
5. 执行器调用引擎的提交事务接口，引擎把刚刚写入的 redo log 改成提交（commit）状态，更新完成。

从上图可以看出，MySQL 在执行更新语句的时候，在服务层进行语句的解析和执行，在引擎层进行数据的提取和存储；同时在服务层对 binlog 进行写入，在 InnoDB 内进行 redo log 的写入。

不仅如此，在对 redo log 写入时有两个阶段的提交，一是 binlog 写入之前`prepare`状态的写入，二是 binlog 写入之后`commit`状态的写入。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#_22-那为什么要两阶段提交呢)22.那为什么要两阶段提交呢？

为什么要两阶段提交呢？直接提交不行吗？

我们可以假设不采用两阶段提交的方式，而是采用“单阶段”进行提交，即要么先写入 redo log，后写入 binlog；要么先写入 binlog，后写入 redo log。这两种方式的提交都会导致原先数据库的状态和被恢复后的数据库的状态不一致。

**先写入 redo log，后写入 binlog：**

在写完 redo log 之后，数据此时具有`crash-safe`能力，因此系统崩溃，数据会恢复成事务开始之前的状态。但是，若在 redo log 写完时候，binlog 写入之前，系统发生了宕机。此时 binlog 没有对上面的更新语句进行保存，导致当使用 binlog 进行数据库的备份或者恢复时，就少了上述的更新语句。从而使得`id=2`这一行的数据没有被更新。

![先写 redo log，后写 bin log 的问题](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/mysql-75d5226b-cab9-461a-89a9-befcb2dfb996.jpg)先写 redo log，后写 bin log 的问题

**先写入 binlog，后写入 redo log：**

写完 binlog 之后，所有的语句都被保存，所以通过 binlog 复制或恢复出来的数据库中 id=2 这一行的数据会被更新为 a=1。但是如果在 redo log 写入之前，系统崩溃，那么 redo log 中记录的这个事务会无效，导致实际数据库中`id=2`这一行的数据并没有更新。

![先写 bin log，后写 redo log 的问题](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/mysql-869c309b-9b93-46e1-8414-b35128e287a5.jpg)先写 bin log，后写 redo log 的问题

简单说，redo log 和 binlog 都可以用于表示事务的提交状态，而两阶段提交就是让这两个状态保持逻辑上的一致。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#_23-redo-log-怎么刷入磁盘的知道吗)23.redo log 怎么刷入磁盘的知道吗？

redo log 的写入不是直接落到磁盘，而是在内存中设置了一片称之为`redo log buffer`的连续内存空间，也就是`redo 日志缓冲区`。

![redo log 缓冲](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/mysql-e1f59341-0695-45db-b759-30db73314e39.jpg)redo log 缓冲

> **什么时候会刷入磁盘？**

在如下的一些情况中，log buffer 的数据会刷入磁盘：

- log buffer 空间不足时

log buffer 的大小是有限的，如果不停的往这个有限大小的 log buffer 里塞入日志，很快它就会被填满。如果当前写入 log buffer 的 redo 日志量已经占满了 log buffer 总容量的大约**一半**左右，就需要把这些日志刷新到磁盘上。

- 事务提交时

在事务提交时，为了保证持久性，会把 log buffer 中的日志全部刷到磁盘。注意，这时候，除了本事务的，可能还会刷入其它事务的日志。

- 后台线程输入

有一个后台线程，大约每秒都会刷新一次`log buffer`中的`redo log`到磁盘。

- 正常关闭服务器时
- **触发 checkpoint 规则**

重做日志缓存、重做日志文件都是以**块（block）\**的方式进行保存的，称之为\**重做日志块（redo log block）**,块的大小是固定的 512 字节。我们的 redo log 它是固定大小的，可以看作是一个逻辑上的 **log group**，由一定数量的**log block** 组成。

![redo log 分块和写入](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/mysql-8d944e76-89ba-4fa6-9066-64ff4f55b532.jpg)redo log 分块和写入

它的写入方式是从头到尾开始写，写到末尾又回到开头循环写。

其中有两个标记位置：

`write pos`是当前记录的位置，一边写一边后移，写到第 3 号文件末尾后就回到 0 号文件开头。`checkpoint`是当前要擦除的位置，也是往后推移并且循环的，擦除记录前要把记录更新到磁盘。

![write pos 和 checkpoint](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/mysql-31a14149-b261-45d9-bd3b-6afaec16e136.jpg)write pos 和 checkpoint

当`write_pos`追上`checkpoint`时，表示 redo log 日志已经写满。这时候就不能接着往里写数据了，需要执行`checkpoint`规则腾出可写空间。

所谓的**checkpoint 规则**，就是 checkpoint 触发后，将 buffer 中日志页都刷到磁盘。

GitHub 上标星 7600+ 的开源知识库《[二哥的 Java 进阶之路open in new window](https://github.com/itwanger/toBeBetterJavaer)》第一版 PDF 终于来了！包括Java基础语法、数组&字符串、OOP、集合框架、Java IO、异常处理、Java 新特性、网络编程、NIO、并发编程、JVM等等，共计 32 万余字，可以说是通俗易懂、风趣幽默……详情戳：[太赞了，GitHub 上标星 7600+ 的 Java 教程open in new window](https://tobebetterjavaer.com/overview/)

微信搜 **沉默王二** 或扫描下方二维码关注二哥的原创公众号沉默王二，回复 **222** 即可免费领取。

![img](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/gongzhonghao.png)

## [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#sql-优化)SQL 优化

### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#_24-慢-sql-如何定位呢)24.慢 SQL 如何定位呢？

慢 SQL 的监控主要通过两个途径：

![发现慢 SQL](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/mysql-c0c43f82-3930-44f0-9abc-b33b08c02d2d.jpg)发现慢 SQL

- **慢查询日志**：开启 MySQL 的慢查询日志，再通过一些工具比如 mysqldumpslow 去分析对应的慢查询日志，当然现在一般的云厂商都提供了可视化的平台。
- **服务监控**：可以在业务的基建中加入对慢 SQL 的监控，常见的方案有字节码插桩、连接池扩展、ORM 框架过程，对服务运行中的慢 SQL 进行监控和告警。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#_25-有哪些方式优化慢-sql)25.有哪些方式优化慢 SQL？

慢 SQL 的优化，主要从两个方面考虑，SQL 语句本身的优化，以及数据库设计的优化。

![SQL 优化](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/mysql-e65e2428-a8f7-4381-9e15-c3e18b4c4d9c.jpg)SQL 优化

#### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#避免不必要的列)避免不必要的列

这个是老生常谈，但还是经常会出的情况，SQL 查询的时候，应该只查询需要的列，而不要包含额外的列，像`slect *` 这种写法应该尽量避免。

#### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#分页优化)分页优化

在数据量比较大，分页比较深的情况下，需要考虑分页的优化。

例如：



```text
select * from table where type = 2 and level = 9 order by id asc limit 190289,10;
```

优化方案：

- **延迟关联**

先通过 where 条件提取出主键，在将该表与原数据表关联，通过主键 id 提取数据行，而不是通过原来的二级索引提取数据行

例如：



```text
select a.* from table a, 
 (select id from table where type = 2 and level = 9 order by id asc limit 190289,10 ) b
 where a.id = b.id
```

- **书签方式**

书签方式就是找到 limit 第一个参数对应的主键值，根据这个主键值再去过滤并 limit

例如：



```text
  select * from table where id >
  (select * from table where type = 2 and level = 9 order by id asc limit 190
```

#### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#索引优化)索引优化

合理地设计和使用索引，是优化慢 SQL 的利器。

**利用覆盖索引**

InnoDB 使用非主键索引查询数据时会回表，但是如果索引的叶节点中已经包含要查询的字段，那它没有必要再回表查询了，这就叫覆盖索引

例如对于如下查询：



```text
select name from test where city='上海'
```

我们将被查询的字段建立到联合索引中，这样查询结果就可以直接从索引中获取



```text
alter table test add index idx_city_name (city, name);
```

**低版本避免使用 or 查询**

在 MySQL 5.0 之前的版本要尽量避免使用 or 查询，可以使用 union 或者子查询来替代，因为早期的 MySQL 版本使用 or 查询可能会导致索引失效，高版本引入了索引合并，解决了这个问题。

**避免使用 != 或者 <> 操作符**

SQL 中，不等于操作符会导致查询引擎放弃查询索引，引起全表扫描，即使比较的字段上有索引

解决方法：通过把不等于操作符改成 or，可以使用索引，避免全表扫描

例如，把`column<>’aaa’，改成column>’aaa’ or column<’aaa’`，就可以使用索引了

**适当使用前缀索引**

适当地使用前缀所云，可以降低索引的空间占用，提高索引的查询效率。

比如，邮箱的后缀都是固定的“`@xxx.com`”，那么类似这种后面几位为固定值的字段就非常适合定义为前缀索引



```text
alter table test add index index2(email(6));
```

PS:需要注意的是，前缀索引也存在缺点，MySQL 无法利用前缀索引做 order by 和 group by 操作，也无法作为覆盖索引

**避免列上函数运算**

要避免在列字段上进行算术运算或其他表达式运算，否则可能会导致存储引擎无法正确使用索引，从而影响了查询的效率



```text
select * from test where id + 1 = 50;
select * from test where month(updateTime) = 7;
```

**正确使用联合索引**

使用联合索引的时候，注意最左匹配原则。

#### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#join-优化)JOIN 优化

**优化子查询**

尽量使用 Join 语句来替代子查询，因为子查询是嵌套查询，而嵌套查询会新创建一张临时表，而临时表的创建与销毁会占用一定的系统资源以及花费一定的时间，同时对于返回结果集比较大的子查询，其对查询性能的影响更大

**小表驱动大表**

关联查询的时候要拿小表去驱动大表，因为关联的时候，MySQL 内部会遍历驱动表，再去连接被驱动表。

比如 left join，左表就是驱动表，A 表小于 B 表，建立连接的次数就少，查询速度就被加快了。



```text
 select name from A left join B ;
```

**适当增加冗余字段**

增加冗余字段可以减少大量的连表查询，因为多张表的连表查询性能很低，所有可以适当的增加冗余字段，以减少多张表的关联查询，这是以空间换时间的优化策略

**避免使用 JOIN 关联太多的表**

《阿里巴巴 Java 开发手册》规定不要 join 超过三张表，第一 join 太多降低查询的速度，第二 join 的 buffer 会占用更多的内存。

如果不可避免要 join 多张表，可以考虑使用数据异构的方式异构到 ES 中查询。

#### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#排序优化)排序优化

**利用索引扫描做排序**

MySQL 有两种方式生成有序结果：其一是对结果集进行排序的操作，其二是按照索引顺序扫描得出的结果自然是有序的

但是如果索引不能覆盖查询所需列，就不得不每扫描一条记录回表查询一次，这个读操作是随机 IO，通常会比顺序全表扫描还慢

因此，在设计索引时，尽可能使用同一个索引既满足排序又用于查找行

例如：



```text
--建立索引（date,staff_id,customer_id）
select staff_id, customer_id from test where date = '2010-01-01' order by staff_id,customer_id;
```

只有当索引的列顺序和 ORDER BY 子句的顺序完全一致，并且所有列的排序方向都一样时，才能够使用索引来对结果做排序

#### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#union-优化)UNION 优化

**条件下推**

MySQL 处理 union 的策略是先创建临时表，然后将各个查询结果填充到临时表中最后再来做查询，很多优化策略在 union 查询中都会失效，因为它无法利用索引

最好手工将 where、limit 等子句下推到 union 的各个子查询中，以便优化器可以充分利用这些条件进行优化

此外，除非确实需要服务器去重，一定要使用 union all，如果不加 all 关键字，MySQL 会给临时表加上 distinct 选项，这会导致对整个临时表做唯一性检查，代价很高。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#_26-怎么看执行计划-explain-如何理解其中各个字段的含义)26.怎么看执行计划（explain），如何理解其中各个字段的含义？

explain 是 sql 优化的利器，除了优化慢 sql，平时的 sql 编写，也应该先 explain，查看一下执行计划，看看是否还有优化的空间。

直接在 select 语句之前增加`explain` 关键字，就会返回执行计划的信息。

![img](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/mysql-77711553-bb7b-4580-968a-4a973e3a31ca.jpg)

![img](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/mysql-e234658f-5672-4a8d-9a75-872b305a171d.jpg)

1. **id** 列：MySQL 会为每个 select 语句分配一个唯一的 id 值
2. **select_type** 列，查询的类型，根据关联、union、子查询等等分类，常见的查询类型有 SIMPLE、PRIMARY。
3. **table** 列：表示 explain 的一行正在访问哪个表。
4. **type** 列：最重要的列之一。表示关联类型或访问类型，即 MySQL 决定如何查找表中的行。

性能从最优到最差分别为：system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range > index > ALL

- system

`system`：当表仅有一行记录时(系统表)，数据量很少，往往不需要进行磁盘 IO，速度非常快

- const

`const`：表示查询时命中 `primary key` 主键或者 `unique` 唯一索引，或者被连接的部分是一个常量(`const`)值。这类扫描效率极高，返回数据量少，速度非常快。

- eq_ref

`eq_ref`：查询时命中主键`primary key` 或者 `unique key`索引， `type` 就是 `eq_ref`。

- ref_or_null

`ref_or_null`：这种连接类型类似于 ref，区别在于 `MySQL`会额外搜索包含`NULL`值的行。

- index_merge

`index_merge`：使用了索引合并优化方法，查询使用了两个以上的索引。

- unique_subquery

`unique_subquery`：替换下面的 `IN`子查询，子查询返回不重复的集合。

- index_subquery

`index_subquery`：区别于`unique_subquery`，用于非唯一索引，可以返回重复值。

- range

`range`：使用索引选择行，仅检索给定范围内的行。简单点说就是针对一个有索引的字段，给定范围检索数据。在`where`语句中使用 `bettween...and`、`<`、`>`、`<=`、`in` 等条件查询 `type` 都是 `range`。

- index

`index`：`Index` 与`ALL` 其实都是读全表，区别在于`index`是遍历索引树读取，而`ALL`是从硬盘中读取。

- ALL

就不用多说了，全表扫描。

1. **possible_keys** 列：显示查询可能使用哪些索引来查找，使用索引优化 sql 的时候比较重要。
2. **key** 列：这一列显示 mysql 实际采用哪个索引来优化对该表的访问，判断索引是否失效的时候常用。
3. **key_len** 列：显示了 MySQL 使用
4. **ref** 列：ref 列展示的就是与索引列作等值匹配的值，常见的有：const（常量），func，NULL，字段名。
5. **rows** 列：这也是一个重要的字段，MySQL 查询优化器根据统计信息，估算 SQL 要查到结果集需要扫描读取的数据行数，这个值非常直观显示 SQL 的效率好坏，原则上 rows 越少越好。
6. **Extra** 列：显示不适合在其它列的额外信息，虽然叫额外，但是也有一些重要的信息：

- Using index：表示 MySQL 将使用覆盖索引，以避免回表
- Using where：表示会在存储引擎检索之后再进行过滤
- Using temporary ：表示对查询结果排序时会使用一个临时表。

GitHub 上标星 7600+ 的开源知识库《[二哥的 Java 进阶之路open in new window](https://github.com/itwanger/toBeBetterJavaer)》第一版 PDF 终于来了！包括Java基础语法、数组&字符串、OOP、集合框架、Java IO、异常处理、Java 新特性、网络编程、NIO、并发编程、JVM等等，共计 32 万余字，可以说是通俗易懂、风趣幽默……详情戳：[太赞了，GitHub 上标星 7600+ 的 Java 教程open in new window](https://tobebetterjavaer.com/overview/)

微信搜 **沉默王二** 或扫描下方二维码关注二哥的原创公众号沉默王二，回复 **222** 即可免费领取。

![img](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/gongzhonghao.png)

## [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#索引)索引

索引可以说是 MySQL 面试中的重中之重，一定要彻底拿下。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#_27-能简单说一下索引的分类吗)27.能简单说一下索引的分类吗？

从三个不同维度对索引分类：

![索引分类](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/mysql-f650c0b8-9ebe-4e17-ac7e-b8eb121756a1.jpg)索引分类

例如从基本使用使用的角度来讲：

- 主键索引: InnoDB 主键是默认的索引，数据列不允许重复，不允许为 NULL，一个表只能有一个主键。
- 唯一索引: 数据列不允许重复，允许为 NULL 值，一个表允许多个列创建唯一索引。
- 普通索引: 基本的索引类型，没有唯一性的限制，允许为 NULL 值。
- 组合索引：多列值组成一个索引，用于组合搜索，效率大于索引合并

### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#_28-为什么使用索引会加快查询)28.为什么使用索引会加快查询？

传统的查询方法，是按照表的顺序遍历的，不论查询几条数据，MySQL 需要将表的数据从头到尾遍历一遍。

在我们添加完索引之后，MySQL 一般通过 BTREE 算法生成一个索引文件，在查询数据库时，找到索引文件进行遍历，在比较小的索引数据里查找，然后映射到对应的数据，能大幅提升查找的效率。

和我们通过书的目录，去查找对应的内容，一样的道理。

![索引加快查询远离](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/mysql-6b9c9901-9bf3-46ed-a5c4-c1b781965c1e.jpg)索引加快查询远离

### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#_29-创建索引有哪些注意点)29.创建索引有哪些注意点？

索引虽然是 sql 性能优化的利器，但是索引的维护也是需要成本的，所以创建索引，也要注意：

1. 索引应该建在查询应用频繁的字段

在用于 where 判断、 order 排序和 join 的(on)字段上创建索引。

1. 索引的个数应该适量

索引需要占用空间；更新时候也需要维护。

1. 区分度低的字段，例如性别，不要建索引。

离散度太低的字段，扫描的行数降低的有限。

1. 频繁更新的值，不要作为主键或者索引

维护索引文件需要成本；还会导致页分裂，IO 次数增多。

1. 组合索引把散列性高(区分度高)的值放在前面

为了满足最左前缀匹配原则

1. 创建组合索引，而不是修改单列索引。

组合索引代替多个单列索引（对于单列索引，MySQL 基本只能使用一个索引，所以经常使用多个条件查询时更适合使用组合索引）

1. 过长的字段，使用前缀索引。当字段值比较长的时候，建立索引会消耗很多的空间，搜索起来也会很慢。我们可以通过截取字段的前面一部分内容建立索引，这个就叫前缀索引。
2. 不建议用无序的值(例如身份证、UUID )作为索引

当主键具有不确定性，会造成叶子节点频繁分裂，出现磁盘存储的碎片化

### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#_30-索引哪些情况下会失效呢)30.索引哪些情况下会失效呢？

- 查询条件包含 or，可能导致索引失效
- 如果字段类型是字符串，where 时一定用引号括起来，否则会因为隐式类型转换，索引失效
- like 通配符可能导致索引失效。
- 联合索引，查询时的条件列不是联合索引中的第一个列，索引失效。
- 在索引列上使用 mysql 的内置函数，索引失效。
- 对索引列运算（如，+、-、*、/），索引失效。
- 索引字段上使用（！= 或者 < >，not in）时，可能会导致索引失效。
- 索引字段上使用 is null， is not null，可能导致索引失效。
- 左连接查询或者右连接查询查询关联的字段编码格式不一样，可能导致索引失效。
- MySQL 优化器估计使用全表扫描要比使用索引快,则不使用索引。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#_31-索引不适合哪些场景呢)31.索引不适合哪些场景呢？

- 数据量比较少的表不适合加索引
- 更新比较频繁的字段也不适合加索引
- 离散低的字段不适合加索引（如性别）

### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#_32-索引是不是建的越多越好呢)32.索引是不是建的越多越好呢？

当然不是。

- **索引会占据磁盘空间**
- **索引虽然会提高查询效率，但是会降低更新表的效率**。比如每次对表进行增删改操作，MySQL 不仅要保存数据，还有保存或者更新对应的索引文件。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#_33-mysql-索引用的什么数据结构了解吗)33.MySQL 索引用的什么数据结构了解吗？

MySQL 的默认存储引擎是 InnoDB，它采用的是 B+树结构的索引。

- B+树：只有叶子节点才会存储数据，非叶子节点只存储键值。叶子节点之间使用双向指针连接，最底层的叶子节点形成了一个双向有序链表。

![B+树索引](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/mysql-0a47bec7-1cf2-47d7-94e2-3cd82ce806c7.jpg)B+树索引

在这张图里，有两个重点：

- 最外面的方块，的块我们称之为一个磁盘块，可以看到每个磁盘块包含几个数据项（粉色所示）和指针（黄色/灰色所示），如根节点磁盘包含数据项 17 和 35，包含指针 P1、P2、P3，P1 表示小于 17 的磁盘块，P2 表示在 17 和 35 之间的磁盘块，P3 表示大于 35 的磁盘块。真实的数据存在于叶子节点即 3、4、5……、65。非叶子节点只不存储真实的数据，只存储指引搜索方向的数据项，如 17、35 并不真实存在于数据表中。
- 叶子节点之间使用双向指针连接，最底层的叶子节点形成了一个双向有序链表，可以进行范围查询。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#_34-那一棵-b-树能存储多少条数据呢)34.那一棵 B+树能存储多少条数据呢？

![B+树存储数据条数](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/mysql-16f3523d-20b0-4376-908d-ac40b329768f.jpg)B+树存储数据条数

假设索引字段是 bigint 类型，长度为 8 字节。指针大小在 InnoDB 源码中设置为 6 字节，这样一共 14 字节。非叶子节点(一页)可以存储 16384/14=1170 个这样的 单元(键值+指针)，代表有 1170 个指针。

树深度为 2 的时候，有 1170^2 个叶子节点，可以存储的数据为 1170*1170*16=**21902400**。

在查找数据时一次页的查找代表一次 IO，也就是说，一张 2000 万左右的表，查询数据最多需要访问 3 次磁盘。

所以在 InnoDB 中 B+ 树深度一般为 1-3 层，它就能满足千万级的数据存储。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#_35-为什么要用-b-树-而不用普通二叉树)35.为什么要用 B+ 树，而不用普通二叉树？

可以从几个维度去看这个问题，查询是否够快，效率是否稳定，存储数据多少，以及查找磁盘次数。

> **为什么不用普通二叉树？**

普通二叉树存在退化的情况，如果它退化成链表，相当于全表扫描。平衡二叉树相比于二叉查找树来说，查找效率更稳定，总体的查找速度也更快。

> **为什么不用平衡二叉树呢？**

读取数据的时候，是从磁盘读到内存。如果树这种数据结构作为索引，那每查找一次数据就需要从磁盘中读取一个节点，也就是一个磁盘块，但是平衡二叉树可是每个节点只存储一个键值和数据的，如果是 B+ 树，可以存储更多的节点数据，树的高度也会降低，因此读取磁盘的次数就降下来啦，查询效率就快。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#_36-为什么用-b-树而不用-b-树呢)36.为什么用 B+ 树而不用 B 树呢？

B+相比较 B 树，有这些优势：

- 它是 B Tree 的变种，B Tree 能解决的问题，它都能解决。

B Tree 解决的两大问题：每个节点存储更多关键字；路数更多

- 扫库、扫表能力更强

如果我们要对表进行全表扫描，只需要遍历叶子节点就可以 了，不需要遍历整棵 B+Tree 拿到所有的数据。

- B+Tree 的磁盘读写能力相对于 B Tree 来说更强，IO 次数更少

根节点和枝节点不保存数据区， 所以一个节点可以保存更多的关键字，一次磁盘加载的关键字更多，IO 次数更少。

- 排序能力更强

因为叶子节点上有下一个数据区的指针，数据形成了链表。

- 效率更加稳定

B+Tree 永远是在叶子节点拿到数据，所以 IO 次数是稳定的。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#_37-hash-索引和-b-树索引区别是什么)37.Hash 索引和 B+ 树索引区别是什么？

- B+ 树可以进行范围查询，Hash 索引不能。
- B+ 树支持联合索引的最左侧原则，Hash 索引不支持。
- B+ 树支持 order by 排序，Hash 索引不支持。
- Hash 索引在等值查询上比 B+ 树效率更高。
- B+ 树使用 like 进行模糊查询的时候，like 后面（比如 % 开头）的话可以起到优化的作用，Hash 索引根本无法进行模糊查询。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#_38-聚簇索引与非聚簇索引的区别)38.聚簇索引与非聚簇索引的区别？

首先理解聚簇索引不是一种新的索引，而是而是一种**数据存储方式**。聚簇表示数据行和相邻的键值紧凑地存储在一起。我们熟悉的两种存储引擎——MyISAM 采用的是非聚簇索引，InnoDB 采用的是聚簇索引。

可以这么说：

- 索引的数据结构是树，聚簇索引的索引和数据存储在一棵树上，树的叶子节点就是数据，非聚簇索引索引和数据不在一棵树上。

![聚簇索引和非聚簇索引](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/mysql-692cced2-615a-4b70-a933-69771d53e809.jpg)聚簇索引和非聚簇索引

- 一个表中只能拥有一个聚簇索引，而非聚簇索引一个表可以存在多个。
- 聚簇索引，索引中键值的逻辑顺序决定了表中相应行的物理顺序；索引，索引中索引的逻辑顺序与磁盘上行的物理存储顺序不同。
- 聚簇索引：物理存储按照索引排序；非聚集索引：物理存储不按照索引排序；

### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#_39-回表了解吗)39.回表了解吗？

在 InnoDB 存储引擎里，利用辅助索引查询，先通过辅助索引找到主键索引的键值，再通过主键值查出主键索引里面没有符合要求的数据，它比基于主键索引的查询多扫描了一棵索引树，这个过程就叫回表。

例如:`select \* from user where name = ‘张三’;`

![InnoDB 回表](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/mysql-7d69e289-dc05-47e1-9308-20a8278ebf2e.jpg)InnoDB 回表

### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#_40-覆盖索引了解吗)40.覆盖索引了解吗？

在辅助索引里面，不管是单列索引还是联合索引，如果 select 的数据列只用辅助索引中就能够取得，不用去查主键索引，这时候使用的索引就叫做覆盖索引，避免了回表。

比如，`select name from user where name = ‘张三’;`

![覆盖索引](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/mysql-69e33c61-34bc-4f4b-912b-ca7beb9d5c7c.jpg)覆盖索引

### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#_41-什么是最左前缀原则-最左匹配原则)41.什么是最左前缀原则/最左匹配原则？

注意：最左前缀原则、最左匹配原则、最左前缀匹配原则这三个都是一个概念。

**最左匹配原则**：在 InnoDB 的联合索引中，查询的时候只有匹配了前一个/左边的值之后，才能匹配下一个。

根据最左匹配原则，我们创建了一个组合索引，如 (a1,a2,a3)，相当于创建了（a1）、(a1,a2)和 (a1,a2,a3) 三个索引。

为什么不从最左开始查，就无法匹配呢？

比如有一个 user 表，我们给 name 和 age 建立了一个组合索引。



```text
ALTER TABLE user add INDEX comidx_name_phone (name,age);
```

组合索引在 B+Tree 中是复合的数据结构，它是按照从左到右的顺序来建立搜索树的 (name 在左边，age 在右边)。

![组合索引](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/mysql-e348203c-f00a-42a4-a745-b219d98ea435.jpg)组合索引

从这张图可以看出来，name 是有序的，age 是无序的。当 name 相等的时候， age 才是有序的。

这个时候我们使用 `where name= ‘张三‘ and age = ‘20 ‘`去查询数据的时候， B+Tree 会优先比较 name 来确定下一步应该搜索的方向，往左还是往右。如果 name 相同的时候再比较 age。但是如果查询条件没有 name，就不知道下一步应该查哪个 节点，因为建立搜索树的时候 name 是第一个比较因子，所以就没用上索引。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#_42-什么是索引下推优化)42.什么是索引下推优化？

索引条件下推优化`（Index Condition Pushdown (ICP) ）`是 MySQL5.6 添加的，用于优化数据查询。

- 不使用索引条件下推优化时存储引擎通过索引检索到数据，然后返回给 MySQL Server，MySQL Server 进行过滤条件的判断。
- 当使用索引条件下推优化时，如果存在某些被索引的列的判断条件时，MySQL Server 将这一部分判断条件**下推**给存储引擎，然后由存储引擎通过判断索引是否符合 MySQL Server 传递的条件，只有当索引符合条件时才会将数据检索出来返回给 MySQL 服务器。

例如一张表，建了一个联合索引（name, age），查询语句：`select * from t_user where name like '张%' and age=10;`，由于`name`使用了范围查询，根据最左匹配原则：

不使用 ICP，引擎层查找到`name like '张%'`的数据，再由 Server 层去过滤`age=10`这个条件，这样一来，就回表了两次，浪费了联合索引的另外一个字段`age`。

![没有使用 ICP](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/mysql-c58f59e0-850b-4dfd-8129-2dfc51cf4768.jpg)没有使用 ICP

但是，使用了索引下推优化，把 where 的条件放到了引擎层执行，直接根据`name like '张%' and age=10`的条件进行过滤，减少了回表的次数。

![使用 ICP](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/mysql-a8525cf3-2d16-49a9-a7da-a19762ed16df.jpg)使用 ICP

索引条件下推优化可以减少存储引擎查询基础表的次数，也可以减少 MySQL 服务器从存储引擎接收数据的次数。

GitHub 上标星 7600+ 的开源知识库《[二哥的 Java 进阶之路open in new window](https://github.com/itwanger/toBeBetterJavaer)》第一版 PDF 终于来了！包括Java基础语法、数组&字符串、OOP、集合框架、Java IO、异常处理、Java 新特性、网络编程、NIO、并发编程、JVM等等，共计 32 万余字，可以说是通俗易懂、风趣幽默……详情戳：[太赞了，GitHub 上标星 7600+ 的 Java 教程open in new window](https://tobebetterjavaer.com/overview/)

微信搜 **沉默王二** 或扫描下方二维码关注二哥的原创公众号沉默王二，回复 **222** 即可免费领取。

![img](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/gongzhonghao.png)

## [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#锁)锁

### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#_43-mysql-中有哪几种锁-列举一下)43.MySQL 中有哪几种锁，列举一下？

![MySQL 中的锁](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/mysql-a07e4525-ccc1-4287-aec5-ebf3f277857c.jpg)MySQL 中的锁

如果按锁粒度划分，有以下 3 种：

- 表锁：开销小，加锁快；锁定力度大，发生锁冲突概率高，并发度最低;不会出现死锁。
- 行锁：开销大，加锁慢；会出现死锁；锁定粒度小，发生锁冲突的概率低，并发度高。
- 页锁：开销和加锁速度介于表锁和行锁之间；会出现死锁；锁定粒度介于表锁和行锁之间，并发度一般

如果按照兼容性，有两种，

- 共享锁（S Lock）,也叫读锁（read lock），相互不阻塞。
- 排他锁（X Lock），也叫写锁（write lock），排它锁是阻塞的，在一定时间内，只有一个请求能执行写入，并阻止其它锁读取正在写入的数据。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#_44-说说-innodb-里的行锁实现)44.说说 InnoDB 里的行锁实现?

我们拿这么一个用户表来表示行级锁，其中插入了 4 行数据，主键值分别是 1,6,8,12，现在简化它的聚簇索引结构，只保留数据记录。

![简化的主键索引](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/mysql-013afdbe-889b-4ed0-ae68-c8c9882570d9.jpg)简化的主键索引

InnoDB 的行锁的主要实现如下：

- **Record Lock 记录锁**

记录锁就是直接锁定某行记录。当我们使用唯一性的索引(包括唯一索引和聚簇索引)进行等值查询且精准匹配到一条记录时，此时就会直接将这条记录锁定。例如`select * from t where id =6 for update;`就会将`id=6`的记录锁定。

![记录锁](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/mysql-8989ac27-e442-4c14-81ad-6bc133d78bfd.jpg)记录锁

- **Gap Lock 间隙锁**

间隙锁(Gap Locks) 的间隙指的是两个记录之间逻辑上尚未填入数据的部分,是一个**左开右开空间**。

![间隙锁](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/mysql-d60f3a42-4b0f-4612-b7ad-65191fecb852.jpg)间隙锁

间隙锁就是锁定某些间隙区间的。当我们使用用等值查询或者范围查询，并且没有命中任何一个`record`，此时就会将对应的间隙区间锁定。例如`select * from t where id =3 for update;`或者`select * from t where id > 1 and id < 6 for update;`就会将(1,6)区间锁定。

- **Next-key Lock 临键锁**

临键指的是间隙加上它右边的记录组成的**左开右闭区间**。比如上述的(1,6]、(6,8]等。

![临键锁](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/mysql-ae8a21cc-8b52-467d-9173-4e01b24e04b9.jpg)临键锁

临键锁就是记录锁(Record Locks)和间隙锁(Gap Locks)的结合，即除了锁住记录本身，还要再锁住索引之间的间隙。当我们使用范围查询，并且命中了部分`record`记录，此时锁住的就是临键区间。注意，临键锁锁住的区间会包含最后一个 record 的右边的临键区间。例如`select * from t where id > 5 and id <= 7 for update;`会锁住(4,7]、(7,+∞)。mysql 默认行锁类型就是`临键锁(Next-Key Locks)`。当使用唯一性索引，等值查询匹配到一条记录的时候，临键锁(Next-Key Locks)会退化成记录锁；没有匹配到任何记录的时候，退化成间隙锁。

> `间隙锁(Gap Locks)`和`临键锁(Next-Key Locks)`都是用来解决幻读问题的，在`已提交读（READ COMMITTED）`隔离级别下，`间隙锁(Gap Locks)`和`临键锁(Next-Key Locks)`都会失效！

上面是行锁的三种实现算法，除此之外，在行上还存在插入意向锁。

- **Insert Intention Lock 插入意向锁**

一个事务在插入一条记录时需要判断一下插入位置是不是被别的事务加了意向锁 ，如果有的话，插入操作需要等待，直到拥有 gap 锁 的那个事务提交。但是事务在等待的时候也需要在内存中生成一个 锁结构 ，表明有事务想在某个 间隙 中插入新记录，但是现在在等待。这种类型的锁命名为 Insert Intention Locks ，也就是插入意向锁 。

假如我们有个 T1 事务，给(1,6)区间加上了意向锁，现在有个 T2 事务，要插入一个数据，id 为 4，它会获取一个（1,6）区间的插入意向锁，又有有个 T3 事务，想要插入一个数据，id 为 3，它也会获取一个（1,6）区间的插入意向锁，但是，这两个插入意向锁锁不会互斥。

![插入意向锁](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/mysql-751425cb-daba-4da1-bab6-f843254cad3d.jpg)插入意向锁

### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#_45-意向锁是什么知道吗)45.意向锁是什么知道吗？

意向锁是一个表级锁，不要和插入意向锁搞混。

意向锁的出现是为了支持 InnoDB 的多粒度锁，它解决的是表锁和行锁共存的问题。

当我们需要给一个表加表锁的时候，我们需要根据去判断表中有没有数据行被锁定，以确定是否能加成功。

假如没有意向锁，那么我们就得遍历表中所有数据行来判断有没有行锁；

有了意向锁这个表级锁之后，则我们直接判断一次就知道表中是否有数据行被锁定了。

有了意向锁之后，要执行的事务 A 在申请行锁（写锁）之前，数据库会自动先给事务 A 申请表的意向排他锁。当事务 B 去申请表的互斥锁时就会失败，因为表上有意向排他锁之后事务 B 申请表的互斥锁时会被阻塞。

![意向锁](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/mysql-31f7f49c-1e5a-4d42-b8b3-e022b3ba82ae.jpg)意向锁

### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#_46-mysql-的乐观锁和悲观锁了解吗)46.MySQL 的乐观锁和悲观锁了解吗？

- **悲观锁**（Pessimistic Concurrency Control）：

悲观锁认为被它保护的数据是极其不安全的，每时每刻都有可能被改动，一个事务拿到悲观锁后，其他任何事务都不能对该数据进行修改，只能等待锁被释放才可以执行。

数据库中的行锁，表锁，读锁，写锁均为悲观锁。

- **乐观锁（Optimistic Concurrency Control）**

乐观锁认为数据的变动不会太频繁。

乐观锁通常是通过在表中增加一个版本(version)或时间戳(timestamp)来实现，其中，版本最为常用。

事务在从数据库中取数据时，会将该数据的版本也取出来(v1)，当事务对数据变动完毕想要将其更新到表中时，会将之前取出的版本 v1 与数据中最新的版本 v2 相对比，如果 v1=v2，那么说明在数据变动期间，没有其他事务对数据进行修改，此时，就允许事务对表中的数据进行修改，并且修改时 version 会加 1，以此来表明数据已被变动。

如果，v1 不等于 v2，那么说明数据变动期间，数据被其他事务改动了，此时不允许数据更新到表中，一般的处理办法是通知用户让其重新操作。不同于悲观锁，乐观锁通常是由开发者实现的。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#_47-mysql-遇到过死锁问题吗-你是如何解决的)47.MySQL 遇到过死锁问题吗，你是如何解决的？

排查死锁的一般步骤是这样的：

（1）查看死锁日志 show engine innodb status;

（2）找出死锁 sql

（3）分析 sql 加锁情况

（4）模拟死锁案发

（5）分析死锁日志

（6）分析死锁结果

当然，这只是一个简单的流程说明，实际上生产中的死锁千奇百怪，排查和解决起来没那么简单。

GitHub 上标星 7600+ 的开源知识库《[二哥的 Java 进阶之路open in new window](https://github.com/itwanger/toBeBetterJavaer)》第一版 PDF 终于来了！包括Java基础语法、数组&字符串、OOP、集合框架、Java IO、异常处理、Java 新特性、网络编程、NIO、并发编程、JVM等等，共计 32 万余字，可以说是通俗易懂、风趣幽默……详情戳：[太赞了，GitHub 上标星 7600+ 的 Java 教程open in new window](https://tobebetterjavaer.com/overview/)

微信搜 **沉默王二** 或扫描下方二维码关注二哥的原创公众号沉默王二，回复 **222** 即可免费领取。

![img](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/gongzhonghao.png)

## [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#事务)事务

### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#_48-mysql-事务的四大特性说一下)48.MySQL 事务的四大特性说一下？

![事务四大特性](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/mysql-eaafb8b8-fbe6-42c0-9cc2-f2e04631b56c.jpg)事务四大特性

- 原子性：事务作为一个整体被执行，包含在其中的对数据库的操作要么全部被执行，要么都不执行。
- 一致性：指在事务开始之前和事务结束以后，数据不会被破坏，假如 A 账户给 B 账户转 10 块钱，不管成功与否，A 和 B 的总金额是不变的。
- 隔离性：多个事务并发访问时，事务之间是相互隔离的，即一个事务不影响其它事务运行效果。简言之，就是事务之间是进水不犯河水的。
- 持久性：表示事务完成以后，该事务对数据库所作的操作更改，将持久地保存在数据库之中。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#_49-那-acid-靠什么保证的呢)49.那 ACID 靠什么保证的呢？

- 事务的**隔离性**是通过数据库锁的机制实现的。
- 事务的**一致性**由 undo log 来保证：undo log 是逻辑日志，记录了事务的 insert、update、deltete 操作，回滚的时候做相反的 delete、update、insert 操作来恢复数据。
- 事务的**原子性**和**持久性**由 redo log 来保证：redolog 被称作重做日志，是物理日志，事务提交的时候，必须先将事务的所有日志写入 redo log 持久化，到事务的提交操作才算完成。

![ACID 靠什么保证](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/mysql-61393d95-38d1-4c38-a7a9-4b9580a1f95d.jpg)ACID 靠什么保证

### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#_50-事务的隔离级别有哪些-mysql-的默认隔离级别是什么)50.事务的隔离级别有哪些？MySQL 的默认隔离级别是什么？

![img](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/mysql-99942529-4a91-420b-9ce2-4149e747f64d.jpg)

事务的四个隔离级别

- 读未提交（Read Uncommitted）
- 读已提交（Read Committed）
- 可重复读（Repeatable Read）
- 串行化（Serializable）

MySQL 默认的事务隔离级别是可重复读 (Repeatable Read)。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#_51-什么是幻读-脏读-不可重复读呢)51.什么是幻读，脏读，不可重复读呢？

- 事务 A、B 交替执行，事务 A 读取到事务 B 未提交的数据，这就是**脏读**。
- 在一个事务范围内，两个相同的查询，读取同一条记录，却返回了不同的数据，这就是**不可重复读**。
- 事务 A 查询一个范围的结果集，另一个并发事务 B 往这个范围中插入 / 删除了数据，并静悄悄地提交，然后事务 A 再次查询相同的范围，两次读取得到的结果集不一样了，这就是**幻读**。

不同的隔离级别，在并发事务下可能会发生的问题：

| 隔离级别                   | 脏读 | 不可重复读 | 幻读 |
| -------------------------- | ---- | ---------- | ---- |
| Read Uncommited 读取未提交 | 是   | 是         | 是   |
| Read Commited 读取已提交   | 否   | 是         | 是   |
| Repeatable Read 可重复读   | 否   | 否         | 是   |
| Serialzable 可串行化       | 否   | 否         | 否   |

### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#_52-事务的各个隔离级别都是如何实现的)52.事务的各个隔离级别都是如何实现的？

**读未提交**

读未提交，就不用多说了，采取的是读不加锁原理。

- 事务读不加锁，不阻塞其他事务的读和写
- 事务写阻塞其他事务写，但不阻塞其他事务读；

**读取已提交&可重复读**

读取已提交和可重复读级别利用了`ReadView`和`MVCC`，也就是每个事务只能读取它能看到的版本（ReadView）。

- READ COMMITTED：每次读取数据前都生成一个 ReadView
- REPEATABLE READ ：在第一次读取数据时生成一个 ReadView

**串行化**

串行化的实现采用的是读写都加锁的原理。

串行化的情况下，对于同一行事务，`写`会加`写锁`，`读`会加`读锁`。当出现读写锁冲突的时候，后访问的事务必须等前一个事务执行完成，才能继续执行。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#_53-mvcc-了解吗-怎么实现的)53.MVCC 了解吗？怎么实现的？

MVCC(Multi Version Concurrency Control)，中文名是多版本并发控制，简单来说就是通过维护数据历史版本，从而解决并发访问情况下的读一致性问题。关于它的实现，要抓住几个关键点，**隐式字段、undo 日志、版本链、快照读&当前读、Read View**。

**版本链**

对于 InnoDB 存储引擎，每一行记录都有两个隐藏列**DB_TRX_ID、DB_ROLL_PTR**

- `DB_TRX_ID`，事务 ID，每次修改时，都会把该事务 ID 复制给`DB_TRX_ID`；
- `DB_ROLL_PTR`，回滚指针，指向回滚段的 undo 日志。

![表隐藏列](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/mysql-81b091fb-77d9-440e-940e-253b905c0be3.jpg)表隐藏列

假如有一张`user`表，表中只有一行记录，当时插入的事务 id 为 80。此时，该条记录的示例图如下：

![img](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/mysql-80ebc2b3-ae63-417d-9307-f6a7811f7965.jpg)

接下来有两个`DB_TRX_ID`分别为`100`、`200`的事务对这条记录进行`update`操作，整个过程如下：

![update 操作](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/mysql-bf4ff00d-01bd-4170-a17b-6919f7873ea4.jpg)update 操作

由于每次变动都会先把`undo`日志记录下来，并用`DB_ROLL_PTR`指向`undo`日志地址。因此可以认为，**对该条记录的修改日志串联起来就形成了一个`版本链`，版本链的头节点就是当前记录最新的值**。如下：

![MVCC](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/mysql-765b3d83-14eb-4b56-8940-9d60bfaf1737.jpg)MVCC

**ReadView**

> 对于`Read Committed`和`Repeatable Read`隔离级别来说，都需要读取已经提交的事务所修改的记录，也就是说如果版本链中某个版本的修改没有提交，那么该版本的记录时不能被读取的。所以需要确定在`Read Committed`和`Repeatable Read`隔离级别下，版本链中哪个版本是能被当前事务读取的。于是就引入了`ReadView`这个概念来解决这个问题。

Read View 就是事务执行**快照读**时，产生的读视图，相当于某时刻表记录的一个快照，通过这个快照，我们可以获取：

![事务和 ReadView](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/mysql-4451a8c6-8e90-4941-a6be-c09533fa6c03.jpg)事务和 ReadView

- m_ids ：表示在生成 ReadView 时当前系统中活跃的读写事务的事务 id 列表。
- min_trx_id ：表示在生成 ReadView 时当前系统中活跃的读写事务中最小的 事务 id ，也就是 m_ids 中的最小值。
- max_trx_id ：表示生成 ReadView 时系统中应该分配给下一个事务的 id 值。
- creator_trx_id ：表示生成该 ReadView 的事务的 事务 id

有了这个 ReadView ，这样在访问某条记录时，只需要按照下边的步骤判断记录的某个版本是否可见：

- 如果被访问版本的 DB_TRX_ID 属性值与 ReadView 中的 creator_trx_id 值相同，意味着当前事务在访问它自己修改过的记录，所以该版本可以被当前事务访问。
- 如果被访问版本的 DB_TRX_ID 属性值小于 ReadView 中的 min_trx_id 值，表明生成该版本的事务在当前事务生成 ReadView 前已经提交，所以该版本可以被当前事务访问。
- 如果被访问版本的 DB_TRX_ID 属性值大于 ReadView 中的 max_trx_id 值，表明生成该版本的事务在当前事务生成 ReadView 后才开启，所以该版本不可以被当前事务访问。
- 如果被访问版本的 DB_TRX_ID 属性值在 ReadView 的 min_trx_id 和 max_trx_id 之间，那就需要判断一下 trx_id 属性值是不是在 m_ids 列表中，如果在，说明创建 ReadView 时生成该版本的事务还是活跃的，该版本不可以被访问；如果不在，说明创建 ReadView 时生成该版本的事务已经被提交，该版本可以被访问。

如果某个版本的数据对当前事务不可见的话，那就顺着版本链找到下一个版本的数据，继续按照上边的步骤判断可见性，依此类推，直到版本链中的最后一个版本。如果最后一个版本也不可见的话，那么就意味着该条记录对该事务完全不可见，查询结果就不包含该记录。

在 MySQL 中， READ COMMITTED 和 REPEATABLE READ 隔离级别的的一个非常大的区别就是它们生成 ReadView 的时机不同。

READ COMMITTED 是**每次读取数据前都生成一个 ReadView**，这样就能保证自己每次都能读到其它事务提交的数据；REPEATABLE READ 是在**第一次读取数据时生成一个 ReadView**，这样就能保证后续读取的结果完全一致。

GitHub 上标星 7600+ 的开源知识库《[二哥的 Java 进阶之路open in new window](https://github.com/itwanger/toBeBetterJavaer)》第一版 PDF 终于来了！包括Java基础语法、数组&字符串、OOP、集合框架、Java IO、异常处理、Java 新特性、网络编程、NIO、并发编程、JVM等等，共计 32 万余字，可以说是通俗易懂、风趣幽默……详情戳：[太赞了，GitHub 上标星 7600+ 的 Java 教程open in new window](https://tobebetterjavaer.com/overview/)

微信搜 **沉默王二** 或扫描下方二维码关注二哥的原创公众号沉默王二，回复 **222** 即可免费领取。

![img](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/gongzhonghao.png)

## [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#高可用-性能)高可用/性能

### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#_54-数据库读写分离了解吗)54.数据库读写分离了解吗？

读写分离的基本原理是将数据库读写操作分散到不同的节点上，下面是基本架构图：

![读写分离](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/mysql-31df767c-db05-4de4-a05b-a45bcf76c1bf.jpg)读写分离

读写分离的基本实现是:

- 数据库服务器搭建主从集群，一主一从、一主多从都可以。
- 数据库主机负责读写操作，从机只负责读操作。
- 数据库主机通过复制将数据同步到从机，每台数据库服务器都存储了所有的业务数据。
- 业务服务器将写操作发给数据库主机，将读操作发给数据库从机。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#_55-那读写分离的分配怎么实现呢)55.那读写分离的分配怎么实现呢？

将读写操作区分开来，然后访问不同的数据库服务器，一般有两种方式：程序代码封装和中间件封装。

1. 程序代码封装

程序代码封装指在代码中抽象一个数据访问层（所以有的文章也称这种方式为 "中间层封装" ） ，实现读写操作分离和数据库服务器连接的管理。例如，基于 Hibernate 进行简单封装，就可以实现读写分离：

![业务代码封装](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/mysql-771eb01f-3f1a-4437-8e1b-affe4de36ec3.jpg)业务代码封装

目前开源的实现方案中，淘宝的 TDDL (Taobao Distributed Data Layer, 外号：头都大了）是比较有名的。

1. 中间件封装

中间件封装指的是独立一套系统出来，实现读写操作分离和数据库服务器连接的管理。中间件对业务服务器提供 SQL 兼容的协议，业务服务器无须自己进行读写分离。

对于业务服务器来说，访问中间件和访问数据库没有区别，事实上在业务服务器看来，中间件就是一个数据库服务器。

其基本架构是：

![数据库中间件](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/mysql-f2313613-25bd-4065-8f63-969a4b5757a7.jpg)数据库中间件

### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#_56-主从复制原理了解吗)56.主从复制原理了解吗？

- master 数据写入，更新 binlog
- master 创建一个 dump 线程向 slave 推送 binlog
- slave 连接到 master 的时候，会创建一个 IO 线程接收 binlog，并记录到 relay log 中继日志中
- slave 再开启一个 sql 线程读取 relay log 事件并在 slave 执行，完成同步
- slave 记录自己的 binglog

![主从复制](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/mysql-1bfbfcb5-2392-4f98-be1b-a66204da09e5.jpg)主从复制

### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#_57-主从同步延迟怎么处理)57.主从同步延迟怎么处理？

**主从同步延迟的原因**

一个服务器开放Ｎ个链接给客户端来连接的，这样有会有大并发的更新操作, 但是从服务器的里面读取 binlog 的线程仅有一个，当某个 SQL 在从服务器上执行的时间稍长 或者由于某个 SQL 要进行锁表就会导致，主服务器的 SQL 大量积压，未被同步到从服务器里。这就导致了主从不一致， 也就是主从延迟。

**主从同步延迟的解决办法**

解决主从复制延迟有几种常见的方法:

1. 写操作后的读操作指定发给数据库主服务器

例如，注册账号完成后，登录时读取账号的读操作也发给数据库主服务器。这种方式和业务强绑定，对业务的侵入和影响较大，如果哪个新来的程序员不知道这样写代码，就会导致一个 bug。

1. 读从机失败后再读一次主机

这就是通常所说的 "二次读取" ，二次读取和业务无绑定，只需要对底层数据库访问的 API 进行封装即可，实现代价较小，不足之处在于如果有很多二次读取，将大大增加主机的读操作压力。例如，黑客暴力破解账号，会导致大量的二次读取操作，主机可能顶不住读操作的压力从而崩溃。

1. 关键业务读写操作全部指向主机，非关键业务采用读写分离

例如，对于一个用户管理系统来说，注册 + 登录的业务读写操作全部访问主机，用户的介绍、爰好、等级等业务，可以采用读写分离，因为即使用户改了自己的自我介绍，在查询时却看到了自我介绍还是旧的，业务影响与不能登录相比就小很多，还可以忍受。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#_58-你们一般是怎么分库的呢)58.你们一般是怎么分库的呢？

- 垂直分库：以表为依据，按照业务归属不同，将不同的表拆分到不同的库中。

![垂直分库](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/mysql-2a43af18-617b-4502-b66a-894c2ff4c6c3.jpg)垂直分库

- 水平分库：以字段为依据，按照一定策略（hash、range 等），将一个库中的数据拆分到多个库中。

![水平分库](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/mysql-debe0fb1-d7f7-4ef2-8c99-13c9377138b6.jpg)水平分库

### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#_59-那你们是怎么分表的)59.那你们是怎么分表的？

- 水平分表：以字段为依据，按照一定策略（hash、range 等），将一个表中的数据拆分到多个表中。
- 垂直分表：以字段为依据，按照字段的活跃性，将表中字段拆到不同的表（主表和扩展表）中。

![表拆分](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/mysql-7cba6ce0-c8bb-4f51-9c3b-e5a44e724c79.jpg)表拆分

### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#_60-水平分表有哪几种路由方式)60.水平分表有哪几种路由方式？

什么是路由呢？就是数据应该分到哪一张表。

水平分表主要有三种路由方式：

- **范围路由**：选取有序的数据列 （例如，整形、时间戳等） 作为路由的条件，不同分段分散到不同的数据库表中。

我们可以观察一些支付系统，发现只能查一年范围内的支付记录，这个可能就是支付公司按照时间进行了分表。

![范围路由](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/mysql-b3882ca3-1d04-44e2-9015-7e6c867255a0.jpg)范围路由

范围路由设计的复杂点主要体现在分段大小的选取上，分段太小会导致切分后子表数量过多，增加维护复杂度；分段太大可能会导致单表依然存在性能问题，一般建议分段大小在 100 万至 2000 万之间，具体需要根据业务选取合适的分段大小。

范围路由的优点是可以随着数据的增加平滑地扩充新的表。例如，现在的用户是 100 万，如果增加到 1000 万，只需要增加新的表就可以了，原有的数据不需要动。范围路由的一个比较隐含的缺点是分布不均匀，假如按照 1000 万来进行分表，有可能某个分段实际存储的数据量只有 1000 条，而另外一个分段实际存储的数据量有 900 万条。

- **Hash 路由**：选取某个列 （或者某几个列组合也可以） 的值进行 Hash 运算，然后根据 Hash 结果分散到不同的数据库表中。

同样以订单 id 为例，假如我们一开始就规划了 4 个数据库表，路由算法可以简单地用 id % 4 的值来表示数据所属的数据库表编号，id 为 12 的订单放到编号为 50 的子表中，id 为 13 的订单放到编号为 61 的字表中。

![Hash 路由](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/mysql-e01e7757-c337-48c8-95db-2f7cfd2bc036.jpg)Hash 路由

Hash 路由设计的复杂点主要体现在初始表数量的选取上，表数量太多维护比较麻烦，表数量太少又可能导致单表性能存在问题。而用了 Hash 路由后，增加子表数量是非常麻烦的，所有数据都要重分布。Hash 路由的优缺点和范围路由基本相反，Hash 路由的优点是表分布比较均匀，缺点是扩充新的表很麻烦，所有数据都要重分布。

- **配置路由**：配置路由就是路由表，用一张独立的表来记录路由信息。同样以订单 id 为例，我们新增一张 order_router 表，这个表包含 orderjd 和 tablejd 两列 , 根据 orderjd 就可以查询对应的 table_id。

配置路由设计简单，使用起来非常灵活，尤其是在扩充表的时候，只需要迁移指定的数据，然后修改路由表就可以了。

![配置路由](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/mysql-fcd34332-d38d-455a-875d-d4afd37cac72.jpg)配置路由

配置路由的缺点就是必须多查询一次，会影响整体性能；而且路由表本身如果太大（例如，几亿条数据） ，性能同样可能成为瓶颈，如果我们再次将路由表分库分表，则又面临一个死循环式的路由算法选择问题。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#_61-不停机扩容怎么实现)61.不停机扩容怎么实现？

实际上，不停机扩容，实操起来是个非常麻烦而且很有风险的操作，当然，面试回答起来就简单很多。

- **第一阶段：在线双写，查询走老库**

1. 建立好新的库表结构，数据写入久库的同时，也写入拆分的新库
2. 数据迁移，使用数据迁移程序，将旧库中的历史数据迁移到新库
3. 使用定时任务，新旧库的数据对比，把差异补齐

![img](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/mysql-2d4d94c9-e816-47fc-93dd-a835b1318099.jpg)

- **第二阶段：在线双写，查询走新库**

1. 完成了历史数据的同步和校验
2. 把对数据的读切换到新库

![img](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/mysql-5cf01486-72c1-4eab-9f6e-a19c31569f46.jpg)

- **第三阶段：旧库下线**

1. 旧库不再写入新的数据
2. 经过一段时间，确定旧库没有请求之后，就可以下线老库

![img](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/sidebar/sanfene/mysql-a122d6d5-fff2-4ccd-8ddb-a9282eb2e2da.jpg)

### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#_62-常用的分库分表中间件有哪些)62.常用的分库分表中间件有哪些？

- sharding-jdbc
- Mycat

### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#_63-那你觉得分库分表会带来什么问题呢)63.那你觉得分库分表会带来什么问题呢？

从分库的角度来讲：

- **事务的问题**

使用关系型数据库，有很大一点在于它保证事务完整性。

而分库之后单机事务就用不上了，必须使用分布式事务来解决。

- **跨库 JOIN 问题**

在一个库中的时候我们还可以利用 JOIN 来连表查询，而跨库了之后就无法使用 JOIN 了。

此时的解决方案就是**在业务代码中进行关联**，也就是先把一个表的数据查出来，然后通过得到的结果再去查另一张表，然后利用代码来关联得到最终的结果。

这种方式实现起来稍微比较复杂，不过也是可以接受的。

还有可以**适当的冗余一些字段**。比如以前的表就存储一个关联 ID，但是业务时常要求返回对应的 Name 或者其他字段。这时候就可以把这些字段冗余到当前表中，来去除需要关联的操作。

还有一种方式就是**数据异构**，通过 binlog 同步等方式，把需要跨库 join 的数据异构到 ES 等存储结构中，通过 ES 进行查询。

从分表的角度来看：

- **跨节点的 count,order by,group by 以及聚合函数问题**

只能由业务代码来实现或者用中间件将各表中的数据汇总、排序、分页然后返回。

- **数据迁移，容量规划，扩容等问题**

数据的迁移，容量如何规划，未来是否可能再次需要扩容，等等，都是需要考虑的问题。

- **ID 问题**

数据库表被切分后，不能再依赖数据库自身的主键生成机制，所以需要一些手段来保证全局主键唯一。

1. 还是自增，只不过自增步长设置一下。比如现在有三张表，步长设置为 3，三张表 ID 初始值分别是 1、2、3。这样第一张表的 ID 增长是 1、4、7。第二张表是 2、5、8。第三张表是 3、6、9，这样就不会重复了。
2. UUID，这种最简单，但是不连续的主键插入会导致严重的页分裂，性能比较差。
3. 分布式 ID，比较出名的就是 Twitter 开源的 sonwflake 雪花算法

GitHub 上标星 7600+ 的开源知识库《[二哥的 Java 进阶之路open in new window](https://github.com/itwanger/toBeBetterJavaer)》第一版 PDF 终于来了！包括Java基础语法、数组&字符串、OOP、集合框架、Java IO、异常处理、Java 新特性、网络编程、NIO、并发编程、JVM等等，共计 32 万余字，可以说是通俗易懂、风趣幽默……详情戳：[太赞了，GitHub 上标星 7600+ 的 Java 教程open in new window](https://tobebetterjavaer.com/overview/)

微信搜 **沉默王二** 或扫描下方二维码关注二哥的原创公众号沉默王二，回复 **222** 即可免费领取。

![img](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/gongzhonghao.png)

## [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#运维)运维

### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#_64-百万级别以上的数据如何删除)64.百万级别以上的数据如何删除？

关于索引：由于索引需要额外的维护成本，因为索引文件是单独存在的文件,所以当我们对数据的增加,修改,删除,都会产生额外的对索引文件的操作,这些操作需要消耗额外的 IO,会降低增/改/删的执行效率。

所以，在我们删除数据库百万级别数据的时候，查询 MySQL 官方手册得知删除数据的速度和创建的索引数量是成正比的。

1. 所以我们想要删除百万数据的时候可以先删除索引
2. 然后删除其中无用数据
3. 删除完成后重新创建索引创建索引也非常快

### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#_65-百万千万级大表如何添加字段)65.百万千万级大表如何添加字段？

当线上的数据库数据量到达几百万、上千万的时候，加一个字段就没那么简单，因为可能会长时间锁表。

大表添加字段，通常有这些做法：

- 通过中间表转换过去

创建一个临时的新表，把旧表的结构完全复制过去，添加字段，再把旧表数据复制过去，删除旧表，新表命名为旧表的名称，这种方式可能回丢掉一些数据。

- 用 pt-online-schema-change

`pt-online-schema-change`是 percona 公司开发的一个工具，它可以在线修改表结构，它的原理也是通过中间表。

- 先在从库添加 再进行主从切换

如果一张表数据量大且是热表（读写特别频繁），则可以考虑先在从库添加，再进行主从切换，切换后再将其他几个节点上添加字段。

### [#](https://tobebetterjavaer.com/sidebar/sanfene/mysql.html#_66-mysql-数据库-cpu-飙升的话-要怎么处理呢)66.MySQL 数据库 cpu 飙升的话，要怎么处理呢？

排查过程：

（1）使用 top 命令观察，确定是 mysqld 导致还是其他原因。

（2）如果是 mysqld 导致的，show processlist，查看 session 情况，确定是不是有消耗资源的 sql 在运行。

（3）找出消耗高的 sql，看看执行计划是否准确， 索引是否缺失，数据量是否太大。

处理：

（1）kill 掉这些线程 (同时观察 cpu 使用率是否下降)，

（2）进行相应的调整 (比如说加索引、改 sql、改内存参数)

（3）重新跑这些 SQL。

其他情况：

也有可能是每个 sql 消耗资源并不多，但是突然之间，有大量的 session 连进来导致 cpu 飙升，这种情况就需要跟应用一起来分析为何连接数会激增，再做出相应的调整，比如说限制连接数等

------

*没有什么使我停留——除了目的，纵然岸旁有玫瑰、有绿荫、有宁静的港湾，我是不系之舟*。
