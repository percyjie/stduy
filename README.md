# study
### 1.服务器发布脚本

```sh
#!/bin/sh                              

if [ ! -n "$1" ] ;then

    echo "你没有输入分支号"

    exit

else

    echo "本次拉取的分支号是 $1"

fi  

echo "  ====开始拉取仓库最新代码==== " 

cd /usr/local/xxx;pwd; 


git checkout $1 ;git pull;git status;

echo "         " 

git log --pretty=format:"%h - %an, %ar : %s" -5;

echo "  ====服务器打包===="
# mvn install -Dmaven.test.skip=true;

mvn clean install -Dmaven.test.skip=true;

echo "  =====停止Java应用  等待5秒======"

jps | grep Bootstrap | awk '{print $1;}' | xargs kill -9

#等待5秒

sleep 5 

echo "  ====移动jar包并改名====" 

cd /usr/local/xxx/webapps

rm -rf ROOT.war

cp /usr/local/xxx/web/target/xxx.war /usr/local/tomcat/webapps/;

mv xxx.war ROOT.war;



echo "  =====启动Java应用======"
nohup /usr/local/tomcat/bin/startup.sh & 

#查看日志               
echo "  ===查看日志====";
tail -20f /tmp/logs/xxx.log;

```



### 2.**什么是微服务？**

1.微服务(Microservices Architecture)是一种架构风格，一个大型复杂软件应用由一个或多个微服务组成。系统中的各个微服务可被独立部署，各个微服务之间是松耦合的。每个微服务仅关注于完成一件任务并很好地完成该任务。在所有情况下，每个任务代表着一个小的业务能力。

2.微服务是指开发一个单个 小型的但有业务功能的服务，每个服务都有自己的处理和轻量通讯机制，可以部署在单个或多个服务器上。

3.微服务也指一种种松耦合的、有一定的有界上下文的面向服务架构。也就是说，如果每个服务都要同时修改，那么它们就不是微服务，因为它们紧耦合在一起；如果你需要掌握一个服务太多的上下文场景使用条件，那么它就是一个有上下文边界的服务。

**微服务的优点？**

1.每个微服务都很小，这样能聚焦一个指定的业务功能或业务需求。

2.微服务能够被小团队单独开发，这个小团队是2到5人的开发人员组成。

3.微服务是松耦合的，是有功能意义的服务，无论是在开发阶段或部署阶段都是独立的。

4.微服务能使用不同的语言开发。

5.微服务易于被一个开发人员理解，修改和维护，这样小团队能够更关注自己的工作成果。无需通过合作才能体现价值。

6.微服务允许你利用融合最新技术。

7.微服务只是业务逻辑的代码，不会和HTML,CSS 或其他界面组件混合。

**微服务架构的缺点？**

1.微服务架构可能带来过多的操作。

2.需要DevOps技巧 (http://en.wikipedia.org/wiki/DevOps)。

3.可能双倍的努力。

4.分布式系统可能复杂难以管理。

5.因为分布部署跟踪问题难。

6.当服务数量增加，管理复杂性增加。



###3.Eureka 和 Zookeeper区别？

 1.Eureka是遵守AP原则

   Zookeeper是遵守CP原则   

   Zookeeper保证CP原则：

​     1.1当向注册中心查询服务列表时，我们可以容忍注册中心返回的是几分钟以前的注册信息，

 但不能接受服务直接down掉不可用。也就是说，服务注册功能对可用性的要求高于一致性。

 但是zk会出现这一种情况，当master节点因为网络故障与其他节点失去联系时，剩余注册

 功能就会重新进行leader选举看。问题在于，选举leader的时间太长，30~120s，且选举期间

 整个zk集群都是不可用的，这就导致在选举期间注册服务瘫痪。在云部署的环境下，因网络问题

 使得zk集群失去master节点是较大概率会发生的事，虽然服务能够最终恢复，但是漫长的

 选举时间导致的注册长期不可用是不能容忍的。

​    1.2Eureka看明白了这一点，因此在设计时就优先保证可用性。Eureka各个节点都是平等

 的，几个节点挂掉不会影响正常节点的工作，剩余节点依然可以提供注册和查询服务。

 而Eureka的客户端在向某个Eureka注册或者如果发现链接失败时，则会自动切换至其他节点，

 只要有一台Eureka还在，就能保证注册服务可用（保证可用），只不过查到的信息可能不是

 最新的（不保证一致性）。除此以外，Eureka还有一种自我保护机制，如果在15分钟内超过85%

 的节点都没有正常的心跳，那么Eureka就认为客户端与注册中心出现了网络故障，此时会出现

 一下几种情况：

   （1）Eureka不在从注册列表中移除因为长时间没收到心跳而应该过期的服务

   （2）Eureka仍然能够接受新服务的注册和查询请求，但是不会被同步到其他节点上

​        （保证当前节点依然可用）

   （3）当网络稳定时，当前实例新的注册信息会被同步到其他节点中

 因此，Eureka可以很好的应对因网络故障导致部分节点失去联系的情况，而不会像zk那样

   是整个注册服务瘫痪。

### 3.dubbo自定义异常传递信息丢失问题解决

目前计划对已有的单体项目进行组织架构拆分，调研了分布式系统中常用中间件 Dubbo 和 Spring Cloud，选择了 Dubbo，可以对当前现有项目进行平滑升级改造。但是一开始就遇到了麻烦，自定义异常在传递的过程中变成了 RuntimeException，统一异常处理 **GlobalExceptionHandler** 无法获取异常信息。

## 问题重现

项目进行统一异常处理，抽取了一个通用异常 **ServiceException**，此异常是非受检异常，即继承于 RuntimeException。调研时发现如果服务提供方即 provider 抛出了 **ServiceException** 异常，consumer 服务消费方就会收到一个 RuntimeException 异常，而 **ServiceException** 异常的内容被包含在了 RuntimeException 的异常堆栈中

```
[Request processing failed; nested exception is java.lang.RuntimeException: io.github.mosiki.common.exception.ServiceException: missing_required_parameters
io.github.mosiki.common.exception.ServiceException: missing_required_parameters
    at io.github.mosiki.provider.HelloService.sayHello(HelloService.java:20)
    at com.alibaba.dubbo.common.bytecode.Wrapper1.invokeMethod(Wrapper1.java)
```

而我的统一异常处理是这样的，只处理 `ServiceException` 以及 `Exception`，因此就无法获取到原始异常的信息了。

```
@Slf4j
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ServiceException.class)
    public Result handlerServiceException(ServiceException ex) {
        return Result.failure(ex.getCode(), ex.getMessage());
    }

    @ExceptionHandler({Exception.class})
    public Result handlerException(Exception ex) {
        log.error("发生未知异常：{}", ex);
        return Result.failure(HttpStatus.INTERNAL_SERVER_ERROR.value(), "服务器打了个小盹儿~请稍候再试");
    }
}
```

访问接口将返回如下，异常中原有信息丢失。
![img](http://img12345.5-project.com/blog/20181211061411.png)

上网搜索发现，这是因为 dubbo 的异常处理类 `com.alibaba.dubbo.rpc.filter.ExceptionFilter` 进行处理后的结果，Debug 之后确实如此，dubbo 在此进行了转换。
![img](http://img12345.5-project.com/blog/20181211062145.png)

## 问题解决之道

现在我想要 provider 把自定义的异常原封不动的抛给 consumer 进行处理，于是有了如下思路：

1. 禁用 provider 的 ExceptionFilter
2. 让 GlobalExceptionHandler 处理 consumer 的异常

按照此思路做就很简单了，网上大多文章的办法都比较麻烦，有用 AOP 处理的，甚至还有让自己修改编译源码上传私服的-_-||，本文给出比较简便的方法，提供参考。

## 禁用provider的ExceptionFilter

修改 provider 的配置，我这里使用 yml 配置文件，其他类型如 xml/properties 也同理，设置 provider 的 filter 为 **-exception**，这样异常就不会被处理而是直接抛出了。

```
dubbo:
  application:
    name: provider
  protocol:
    name: dubbo
    port: 20100
  registry:
    address: 127.0.0.1:2181
    protocol: zookeeper
  provider:
    filter: -exception
```

## GlobalExceptionHandler捕获ServiceException

只是禁用了 provider 的 ExceptionHandler 还不能完全达到我们的目的，访问接口，provider 抛出异常 consumer 正确接收为 **ServiceException**。

```
[Request processing failed; nested exception is io.github.mosiki.common.exception.ServiceException: missing_required_parameters] with root cause

io.github.mosiki.common.exception.ServiceException: missing_required_parameters
    at io.github.mosiki.provider.HelloService.sayHello(HelloService.java:20) ~[na:na]
    at com.alibaba.dubbo.common.bytecode.Wrapper1.invokeMethod(Wrapper1.java) ~[na:na]
```

我们处理一下 GlobalExceptionHandler。

SpringBoot 主要这个启动类的位置和全局异常处理器的位置，一定要保证异常处理器在启动类的同级包或者在启动类的子包当中，否则异常处理器将不生效！
![img](http://img12345.5-project.com/blog/20181211060804.png)

## 效果展示

以上两步完成后，重启服务，访问接口测试。
![img](http://img12345.5-project.com/blog/20181211061223.png)

拿到了 provider 抛出的原始自定义异常，如此问题就解决了。