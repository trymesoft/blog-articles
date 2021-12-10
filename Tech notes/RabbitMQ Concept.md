Posted on 2017-04-15.

> 消息服务擅长于解决多系统、异构系统间的数据交换（消息通知/通讯）问题，也可以把它用于系统间服务的相互调用（RPC）。

---

#### RabbitMQ 简介

> AMQP，即 Advanced Message Queuing Protocol，高级消息队列协议，是应用层协议的一个开放标准，为面向消息的中间件设计。
> 消息中间件主要用于组件之间的解耦，消息的发送者无需知道消息使用者的存在，反之亦然。

  AMQP 的主要特征是面向消息、队列、路由（包括点对点和发布/订阅）、可靠性、安全。

  RabbitMQ 是一个开源的 AMQP 实现，服务器端用 Erlang 语言编写，支持多种客户端，如：Python、Ruby、.NET、Java、JMS、C、PHP、ActionScript、XMPP、STOMP 等，支持 AJAX。用于在分布式系统中存储转发消息，在易用性、扩展性、高可用性等方面表现不俗。

---

##### 常用 API 对象

> ConnectionFactory、Connection、Channel 都是 RabbitMQ 对外提供的 API 中最基本的对象。  
> Connection 是 RabbitMQ 的 socket 链接，它封装了 socket 协议相关部分逻辑。  
> ConnectionFactory 为 Connection 的制造工厂。 
> Channel 是我们与 RabbitMQ 打交道的最重要的一个接口，我们大部分的业务操作是在 Channel 这个接口中完成的，包括定义 Queue、定义 Exchange、绑定 Queue 与 Exchange、发布消息等。

---

##### Queue

> 1. Queue（队列）是 RabbitMQ 的内部对象，用于存储消息  
>    <img src="https://tryme.wang/usr/images/sina/5cd95d1f8227a.jpg" align="center" alt="image"> 
> 2. RabbitMQ 中的消息都只能存储在 Queue 中，生产者（P）生产消息并**最终**（中间会经过交换器）投递到 Queue 中，消费者（C）可以从 Queue 中获取消息并消费。  
>    <img src="https://tryme.wang/usr/images/sina/5cd95d1fce57c.jpg" alt="image" align="center">
> 3. 多个消费者可以订阅同一个 Queue，这时 Queue 中的消息会被**平均分摊**给多个消费者进行处理，而不是每个消费者都收到所有的消息并处理。  
>    <img src="https://tryme.wang/usr/images/sina/5cd95d22511d7.jpg" align="center" alt="image"> 

---

##### 消息回执（Message acknowledgment）

> 在实际应用中，可能会发生消费者收到 Queue 中的消息，但没有处理完成就宕机（或出现其他意外）的情况，这种情况下就可能会导致**消息丢失**。为了避免这种情况发生，我们可以要求消费者在消费完消息后发送一个回执给 RabbitMQ，RabbitMQ 收到**消息回执**（Message acknowledgment）后才将该消息从 Queue 中移除；
> 如果 RabbitMQ 没有收到回执并检测到消费者的 RabbitMQ 连接断开，则 RabbitMQ 会将该消息发送给其他消费者（如果存在多个消费者）进行处理。这里不存在 timeout 概念， 一个消费者处理消息时间再长也不会导致该消息被发送给其他消费者，除非它的 RabbitMQ 连接断开。

  这里会产生另外一个问题，如果开发人员在处理完业务逻辑后，忘记发送回执给 RabbitMQ，这将会导致严重的 bug——Queue 中堆积的消息会越来越多，消费者重启后会重复消费这些消息并重复执行业务逻辑…

  **另外 publish message（发送消息）是没有 ack 的。**

---

##### 消息持久化（Message durability）

> 如果我们希望即使在 RabbitMQ 服务重启的情况下，也不会丢失消息，我们可以将 Queue 与 Message 都设置为可持久化的（durable）, 这样可以保证绝大部分情况下我们的 RabbitMQ 消息不会丢失。但依然解决不了小概率丢失事件的发生（比如 RabbitMQ 服务器已经接收到生产者的消息，但还没来得及持久化该消息时 RabbitMQ 服务器就断电了），如果我们需要对这种小概率事件也要管理起来，那么我们要用到事务。

---

##### Prefetch count

> 如果有多个消费者同时订阅同一个 Queue 中的消息，Queue 中的消息会被平摊给多个消费者。这时如果每个消息的处理时间不同，就有可能会导致某些消费者一直在忙，而另外一些消费者很快就处理完手头工作并一直空闲的情况。我们可以通过设置 prefetchCount 来限制 Queue 每次发送给每个消费者的消息数，比如我们设置 prefetchCount=1，则 Queue 每次给每个消费者发送一条消息；消费者处理完这条消息后 Queue 会再给该消费者发送一条消息。

---

##### 交换器（Exchange）

> 实际的情况下，生产者将消息发送到 Exchange（交换器，下图中的 X），由 Exchange 将消息路由到一个或多个 Queue 中（或者丢弃）。  
> <img src="https://tryme.wang/usr/images/sina/5cd95d2345bd1.jpg" align="center" alt="image"> 

---

##### 绑定（Binding）

 > RabbitMQ 中通过 Binding 将 Exchange 与 Queue关联起来，这样 RabbitMQ 就知道如何正确地将消息路由到指定的 Queue 了。  
 > <img src="https://tryme.wang/usr/images/sina/5cd95d23e7f60.jpg" align="center" alt="image"> 

---

##### Routing key、Binding key

 > 生产者在将消息发送给 Exchange 的时候，一般会指定一个 routing key，来指定这个消息的**路由规则**，而这个 **routing key 需要与 Exchange Type 及 binding key 联合使用才能最终生效**。

   在 Exchange Type 与 binding key 固定的情况下（在正常使用时一般这些内容都是固定配置好的），我们的生产者就可以在发送消息给 Exchange 时，通过指定 routing key 来决定消息流向哪里。

   RabbitMQ 为 routing key 设定的长度限制为 255 bytes。 

 > 在绑定（Binding）Exchange 与 Queue 的同时，一般会指定一个 binding key；消费者将消息发送给 Exchange 时，一般会指定一个 routing key；当 binding key 与 routing key 相匹配时，消息将会被路由到对应的 Queue 中。

---

##### Exchange Types

- fanout  
  把所有发送到该 Exchange 的消息路由到所有与它绑定的 Queue 中。
  <img src="https://tryme.wang/usr/images/sina/5cd95d2497247.jpg" align="center" alt="image"> 

- direct  
  把消息路由到那些 binding key 与 routing key 完全匹配的 Queue 中。
  <img src="https://tryme.wang/usr/images/sina/5cd95d25b3888.jpg" align="center" alt="image">   
  以 routingKey=”error” 发送消息到 Exchange，则消息会路由到 Queue1（amqp.gen-S9b…，这是由 RabbitMQ 自动生成的 Queue 名称）和 Queue2（amqp.gen-Agl…）；如果我们以 routingKey=”info”或 routingKey=”warning”来发送消息，则消息只会路由到 Queue2。如果我们以其他 routingKey 发送消息，则消息不会路由到这两个 Queue 中。 

- topic
  前面讲到 direct 类型的 Exchange 路由规则是完全匹配 binding key 与 routing key，但这种严格的匹配方式在很多情况下不能满足实际业务需求。topic 类型的 Exchange 在匹配规则上进行了扩展，它与 direct 类型的 Exchage 相似，也是将消息路由到 binding key 与 routing key 相匹配的 Queue 中，但这里的匹配规则有些不同，它约定：  

  1> routing key 为一个句点号“.”分隔的字符串（我们将被句点号“.”分隔开的每一段独立的字符串称为一个单词），如“stock.usd.nyse”、“nyse.vmw”、“quick.orange.rabbit”  

  2> binding key 与 routing key 一样也是句点号“.”分隔的字符串  

  3> binding key 中可以存在两种特殊字符“*”与“#”，用于做模糊匹配，其中“*”用于匹配一个单词，“#”用于匹配多个单词（可以是零个）  
  <img src="https://tryme.wang/usr/images/sina/5cd95d2716a3c.jpg" align="center" alt="image">  
  以上图中的配置为例，  

  routingKey=”quick.orange.rabbit”的消息会同时路由到 Q1 与 Q2  

  routingKey=”lazy.orange.fox”的消息会路由到 Q1 与 Q2  

  routingKey=”lazy.brown.fox”的消息会路由到 Q2  

  routingKey=”lazy.pink.rabbit”的消息会路由到 Q2（只会投递给 Q2 一次，虽然这个 routingKey 与 Q2 的两个 bindingKey 都匹配）  

  routingKey=”quick.brown.fox”、routingKey=”orange”、routingKey=”quick.orange.male.rabbit”的消息将会被丢弃，因为它们没有匹配任何 bindingKey。 

- headers  
  headers 类型的 Exchange 不依赖于 routing key 与 binding key 的匹配规则来路由消息，而是根据发送的消息内容中的 headers 属性进行匹配。

  在绑定 Queue 与 Exchange 时指定一组键值对；当消息发送到 Exchange 时，RabbitMQ 会取到该消息的 headers（也是一个键值对的形式），对比其中的键值对是否完全匹配 Queue 与 Exchange 绑定时指定的键值对；如果完全匹配则消息会路由到该 Queue，否则不会路由到该 Queue。

---

##### 远程调用（RPC）

 > MQ 本身是基于异步的消息处理，前面的示例中所有的生产者（P）将消息发送到 RabbitMQ 后不会知道消费者（C）处理成功或者失败（甚至连有没有消费者来处理这条消息都不知道）。

   但实际的应用场景中，我们很可能需要一些同步处理，需要同步等待服务端将我的消息处理完成后再进行下一步处理。这相当于 RPC（Remote Procedure Call，远程过程调用）。在 RabbitMQ 中也支持 RPC。  

  <img src="https://tryme.wang/usr/images/sina/5cd95d43ed4a7.jpg" align="center" alt="image"> 

   RabbitMQ 中实现 RPC 的机制是：  

 > 1. 客户端发送请求（消息）时，在消息的属性（MessageProperties，在 AMQP 协议中定义了 14 种 properties，这些属性会随着消息一起发送）中设置两个值 replyTo（一个 Queue 名称，用于告诉服务器处理完成后将通知我的消息发送到这个 Queue 中）和 correlationId（此次请求的标识号，服务器处理完成后需要将此属性返还，客户端将根据这个 id 了解哪条请求被成功执行了或执行失败）  
 > 2. 服务器端收到消息并处理  
 > 3. 服务器端处理完消息后，将生成一条应答消息到 replyTo 指定的 Queue，同时带上 correlationId 属性  
 > 4. 客户端之前已订阅 replyTo 指定的 Queue，从中收到服务器的应答消息后，根据其中的 correlationId 属性分析哪条请求被执行了，根据执行结果进行后续业务处理

 

 **参考：** [RabbitMQ ](