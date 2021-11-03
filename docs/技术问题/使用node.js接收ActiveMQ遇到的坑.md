# 使用node.js接收ActiveMQ遇到的坑

​	因为业务需求，需要用到node.js的方式来连接ActiveMQ。在这里排一下坑。

​	首先，因为师傅提前告知要用到MQ，所以提前百度原理了解到**发布/订阅**和 **点对点**两种模式，由于在开发中是第一次使用ActiveMQ(~~重点是别人家的ActiveMQ~~)，直接使用了师傅当初在现场测试好用的`node.js连接ActiveMQ`代码，代码如下:

```javascript
var Stomp = require('stomp-client'); 
var destination ="/topic/topic_name";
 var client= new Stomp("localhost",61613,"admin", "admin");
 client.connect(function(sessionId) {
 try {
  client.subscribe(destination, function(body, headers){
   console.log("This is the body of a message on the subscribed queue:",body);   
  });
  client.publish(destination,"Oh herrow")
 }catch (error) {
  console.log(error);
 }
});
```

​	OK，端口换成厂商给的61616，改改通道和ip直接粘代码去连AMQ，=_=

​	当然没有这么顺利  `Uncaught Error: server has gone away`

![image-20210126120656615.png](https://i.loli.net/2021/01/26/Luq2OMCfsFTGVge.png)

​	哪写错了?还是哪没改对?厂商的activeMQ不好使?

​	反复试,反复百度,验证的结果是厂商给的activemq可以登录8161amq的服务平台,而且人家厂商都能从amq收到消息了,所以我感觉肯定不是人家的事,应该是我自己的问题,转而去验证我的代码

​	又是一通百度...网上使用nodejs连接amq的代码几乎都试过了,嗯...还是熟悉的错误.....

​	怎么办,叫人吧，师傅这有个问题解决不了,师傅能不能来帮我看一下?师傅上线,从官网找,从源码找,一下午过去了...emm毕竟师傅还有自己的工作要忙,我继续找吧...找到最后怀疑人生,源码里有一部分基层用的是我看不懂的语言...厂商那边还半天不给回复...行...我自己慢慢排查,最后发现在stomp-client的底层 new stocket时我要连接mq的信息全部都存成空了,而且我尝试写了个java程序可以访问厂商的61616端口，但是node就是不行，代码如下：

```java
public static void main(String[] args) {
    // ConnectionFactory ：连接工厂，JMS 用它创建连接
    ConnectionFactory connectionFactory;
    // Connection ：JMS 客户端到JMS Provider 的连接
    Connection connection = null;
    // Session： 一个发送或接收消息的线程
    Session session;
    // Destination ：消息的目的地;消息发送给谁.
    Destination destination;
    // 消费者，消息接收者
    MessageConsumer consumer;
    connectionFactory = new ActiveMQConnectionFactory(
            ActiveMQConnection.DEFAULT_USER,
            ActiveMQConnection.DEFAULT_PASSWORD,
            "tcp://localhost:61616");
    try {
        // 构造从工厂得到连接对象
        connection = connectionFactory.createConnection();
        // 启动
        connection.start();
        // 获取操作连接
        session = connection.createSession(Boolean.FALSE,Session.AUTO_ACKNOWLEDGE);
        // 获取session注意参数值xingbo.xu-queue是一个服务器的queue，须在在ActiveMq的console配置
        destination =  session.createTopic("MIDDLEWARE_ALARM");
        consumer = session.createConsumer(destination);
        while (true) {
            //设置接收者接收消息的时间，为了便于测试，这里谁定为100s
            TextMessage message = (TextMessage) consumer.receive(100000);
            if (null != message) {
                System.out.println("收到消息" + message.getText());
            } else {
                break;
            }
        }
    } catch (Exception e) {
        e.printStackTrace();
    } finally {
        try {
            if (null != connection)
                connection.close();
        } catch (Throwable ignore) {
        }
    }
}
```
​	卡在这一步我感觉我已经竭尽全力了,好在我师傅发现了这个表,activemq使用的每个协议对应的端口都是固定的(这个端口可以在activemq的配置文件里改)

|   协议   | ActiveMQ连接端口 |
| :------: | :--------------: |
|    ws    |      61614       |
| openwire |      61616       |
|  stomp   |      61613       |
|   amqp   |       5672       |
|   mqtt   |       1883       |

​	在这我之前使用的一直是厂商给的61616端口,这个端口对应的协议是openwire(之前用的java就是通过这个实现的),显然我使用nodejs时应该改成stomp协议对应的61613端口，这样我的代码就不会有问题了=_= +1

`Uncaught Error: connect ETIMEDOUT localhost:61613`

![image-20210126125500142.png](https://i.loli.net/2021/01/26/Qkgdto14Nh6SVJf.png)

​	怎么办怎么办....绝望之际我灵机一动部署一个本地的amq看看到底是怎么回事吧

​	部署完毕再用原来写好的代码测试，居然成功了

![image-20210126131735480.png](https://i.loli.net/2021/01/26/nrUXpbT4JF6PYvD.png)

​	我开始怀疑是厂商那边对stomp协议做了手脚,然而他们并不熟悉activemq...他们只是要求openwire协议的61616端口能走得通就行,其他的比如我要用61613的协议他们是不管的不会调的.....(此处省略)

​	好吧,问题逐渐清晰了,厂商给搭建的activemq环境并不健全,那现在就有两种解决办法了，第一种是从服务端使用openwire协议接收厂商amq的消息然后再拿到客户端来处理,第二种是由于之前我师傅在现场测试的demo是好用的，所以现场走得通stomp协议的话就可以继续使用nodejs。

​	我之前部署amq的时候发现有一个测试文件在amq的安装包里，测试的是websocket的61614端口但是我也查到过nodejs写法的websocket连接，于是把测试文件发给现场的处理人员，最终得到的结果是可以接收。

​	所以我选择了第二种方法，测试的是61614所以不便再用61613这个端口了，所以我选择使用stompjs通过websocket协议接收amq,代码如下：

```javascript
const Stomp = require('stompjs');
var client = Stomp.overWS('ws://localhost:61614');
var connectCallback = function (frame) {
        var subscription = client.subscribe('/topic/topic_name', onMessage);
      };
      var onMessage = function (message) {
      if (message.body) {
          console.log("got message with body " ,body)
        } else {
          logger.error("got empty message");
        }
      };
      var errorCallback = function (error) {
          logger.error("ActiveMQ连接出错",error);
      };
      client.connect('admin','admin', connectCallback, errorCallback);
```

​	OK，现在我终于可以接到自己amq的消息了,而且也确认了现场环境的amq可以正常访问,那我怎么才能接到厂商的数据用于测试呢?经过和厂商的不断沟通,发现厂商有一个下传功能,在厂商的平台上配置我的mq地址，厂商的消息就可以转发到了我的amq上了(其实就是把我的amq当做另外一个消费者)，我可以继续的^_^开发了

**总结一下**

​	使用nodejs连接MQ要注意使用的协议对应的端口不同，厂商提供的环境不一定是都好用的，和厂商的沟通要勤快，遇到问题要从多方面考虑才能找到解决办法