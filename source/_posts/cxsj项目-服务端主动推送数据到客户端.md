---
title: cxsj项目-服务端主动推送数据到客户端
date: 2020-05-07 13:20:09
tags: 项目 JavaWeb
---

## 需求分析

在智慧停车管理平台(WEB端)项目的公告发布模块中，管理员发布公告后，车主用户可以评论公告（这可以作为用户提问的一种途径），为了让管理员及时看到用户对公告的评论，以便及时回复，需增加一个**通知提醒**功能，就和大多数WEB应用一样，页面顶部有一个通知图标，如果有通知，就高亮显示起到提醒的作用。


## 目标实现效果
当没有任何通知的时候（在我这个案例中，也就是没有用户回复公告的提醒），显示通知图标。
![没有通知的时候页面顶部状态栏](/images/image-20200508134002898.png)
当有通知时，高亮显示未读通知的数目，当用户点击页面顶部“通知”按钮的时候，根据时间降序显示出所有通知，并且重点标注出哪些通知是未读的。

![image-20200508134511709](/images/image-20200508134511709.png)



## 实现思路

### 基本思路

接下来介绍一下本案例中是怎么实现通知回复功能的。

当用户进行评论公告操作后，先将用户的评论插入到数据库的**评论表**中，如果插入成功，生成一条对应的**通知**记录，在数据库设计时，**通知表**(notification)包含了如下字段：

| 字段名     | 作用                                                         |
| ---------- | ------------------------------------------------------------ |
| id         | 通知表的主键                                                 |
| notifier   | 通知发起者id，外键，关联用户表                               |
| receiver   | 通知接收者id，外键，关联用户表                               |
| outerId    | 公告id，外键，关联公告表，表示notifier在编号为outId的公告下回复了receiver |
| gmt_create | 通知创建的时间戳，bigint类型                                 |
| status     | 通知的阅读状态（对于接收者而言），0标识未读，1标识已读       |

结合具体场景对数据表各个字段的解读：

用户A成功回复了管理员发布的公告，则生成一条通知记录，其中，`notifier`字段存用户A的id，`recevier`字段存管理员的id，outId存被回复的公告的id，gmt_create保存系统当前的时间戳，以记录通知生成的时间，status初始化为0表示通知接收者还未读。

持久化通知记录到数据库之后，我们就可以获得每个用户的未读通知数目和全部历史通知数据：

```sql
#查询某用户未读通知的数目
select count(1) from notification where receiver = 用户ID and status = 0

#查询某用户的全部通知
select * from notification where receiver = 用户ID
```

至此就很明朗了：加载WEB页面时，先查询当前用户的未读通知的数目，如果是0，则显示通知图标，如果大于0，则高亮显示未读通知的数目。这个流程可以通过ajax调用后端查询接口实现，代码就不展开了。

现在问题来了，这样做的缺点是什么？

1、**实时性差**，每次需要手动刷新当前页面或者点击别的页面，重新加载后，才会获得当前实时的未读通知数目。

2、**效率不高**，每进入一个新的页面，都会查询当前的未读通知数目，即使距离上一次查询的一段时间内，未读通知数并没有更新，也会查询。



### 改进思路

针对上述这两个问题，有一些改进思路：

#### ”假装实时“的客户端轮询机制

首先，为了解决实时性差的问题，我们可以想到：客户端每隔一小段时间，就使用ajax调用查询接口，获取当前用户未读通知的数目，获取结果后局部刷新到顶部栏通知框内。

这样做一定程度上实现了”实时“通知：比如说，每隔2秒，客户端就发起一次查询当前通知未读数的请求，服务器返回结果，如果不考虑网络等因素的延时，网页顶部的通知状态栏中的数字始终是2s内的未读通知数，对于普通WEB应用来说，这样的实时性是可以接受的。

轮询机制的代码实现（每隔2000ms调用一次查询方法并更新）：

```javascript
setInterval(function(){
    此处写ajax查询的代码
},2000) 
```



接下来，我们来分析一下这种机制的性能：

{% blockquote %}轮询的基本思路就是浏览器每隔一段时间向浏览器发送http请求，服务器端在收到请求后，不论是否有数据更新，都直接进行响应。这种方式实现的即时通信，本质上还是浏览器发送请求，服务器接受请求的一个过程，通过让客户端不断的进行请求，使得客户端能够模拟实时地收到服务器端的数据的变化。　　

这种方式的优点是比较简单，易于理解，实现起来也没有什么技术难点。缺点是显而易见的，这种方式由于需要不断的建立http连接，严重浪费了服务器端和客户端的资源。尤其是在客户端，距离来说，如果有数量级想对比较大的人同时位于基于短轮询的应用中，那么每一个用户的客户端都会疯狂的向服务器端发送http请求，而且不会间断。人数越多，服务器端压力越大，这是很不合理的。

注:此段描述摘自于[此博客](https://www.cnblogs.com/huchong/p/8595644.html)。{% endblockquote %}

总的来说，轮询机制似乎确实在一定程度上能解决实时性的问题，但是效率很差。

由于轮询有一定的时间差，所以我在标题上说是”假装实时“。

#### 真正实时的WebSocket

##### 概念

之所以轮询机制的效率低，是因为，客户端无法得知后台数据什么时候发生了更新，所以每次发送请求时只能”抱着试一试“的态度，请求当前后台某个数据的值，以尽可能获取最近一段时间内的该数据的最新值，概括地说，**和普通客户端请求一样，轮询机制是由客户端发起请求**。而**理想的模型是在服务器端数据有了变化后，可以主动推送给客户端**,这种"主动型"服务器是解决这类问题的很好的方案。Web Socket就是这样的方案。

{% blockquote %}

WebSocket 是 HTML5 开始提供的一种在单个 TCP 连接上进行全双工通讯的协议。

WebSocket 使得客户端和服务器之间的数据交换变得更加简单，允许服务端主动向客户端推送数据。在 WebSocket API 中，浏览器和服务器只需要完成一次握手，两者之间就直接可以创建持久性的连接，并进行双向数据传输。

HTML5 定义的 WebSocket 协议，能更好的节省服务器资源和带宽，并且能够更实时地进行通讯。

注:此段描述摘自于**[菜鸟教程](https://www.runoob.com/html/html5-websocket.html)**。

{% endblockquote %}

简而言之，WebSocket 拓展了客户端获取后台数据的方式，普通情况下，只能由客户端主动拉取(pull)，而用了WebSocket之后，可以通过服务器主动推送（push），客户端被动获取数据。

讲到这里，终于呼应上本文的标题了。

由于我还没系统学过java的网络编程，之前也没接触过WebSocket，于是采用了号称“让websocket更简单”的解决方案——[GoEasy API](https://www.goeasy.io/)，结果证明，使用起来确实极其简单。

理解GoEasy的工作原理很简单，浏览器订阅channel，服务端后客户端都可以向channel发布消息，消息被发布 到channel后，所有订阅了该channel的客户端都可以获得channel里的消息。

![GoEasy的推送机制](/images/image-20200508170624748.png)

图源自[GoEasy的工作原理](https://www.goeasy.io/cn/doc/index.html)。

##### 使用

这里就简单地介绍一下GoEasy的使用过程吧。

1、注册并登录GoEasy账号后，首先[注册一个应用](https://console.goeasy.io/#/createApp)，选免费版即可。

2、后端推送数据到channel

首先，在maven项目的pom.xml中添加如下依赖:

```xml
    <dependencies> 
        ...........
        <!-- GoEasy的依赖 -->
		<dependency>
            <groupId>io.goeasy</groupId>
            <artifactId>goeasy-sdk</artifactId>
            <version>0.3.8</version>
        </dependency>
        
        <!-- 这是gson的依赖，一开始没有加，报错ClassNotFound了 -->
        <dependency>
            <groupId>com.google.code.gson</groupId>
            <artifactId>gson</artifactId>
            <version>2.8.5</version>
        </dependency>
    </dependencies>
```

```java
GoEasy goEasy = new GoEasy( "这里填写第一步注册应用完成后获取到的Common key");
goEasy.publish("my_channel", String.valueOf(replyNum)); //replyNum是我项目中生成新通知后，更新后的"通知未读数目"
```

3、前端从channel中获得消息

首先，下载[js文件](https://cdn.goeasy.io/download/goeasy-1.0.7.js)，引入到thymeleaf的html模板中。

编写获取channel中数据的前端代码：

```javascript
/**
 * 订阅channel，获取最新的未读通知数目
 */
function subscribleReplyNum() {
    var goEasy = new GoEasy({
        host:'hangzhou.goeasy.io', //应用所在的区域地址: 【hangzhou.goeasy.io |singapore.goeasy.io】
        appkey: 这里依旧填写第一步注册应用完成后获取到的Common key
    });
    goEasy.subscribe({
        channel: "my_channel", //“my_channel”对应于发布时的channel名
        onMessage: function (message) {
            console.log("Channel:" + message.channel + " content:" + message.content); 
            $("#not_read_count").text(message.content);	//更新“通知状态栏”的未读通知数目的span的值
        }
    });

}
```

至此，就可以正常使用了，完成如下效果：

![image-20200508174335930](/images/image-20200508174335930.png)