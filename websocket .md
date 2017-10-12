## websocket

create by yangjianan 12/10/2017

# 目录

- websocket应用场景
- 什么是websocket
- 由轮询到WebSocket
- API介绍使用
- WebSocket和Socket的区别

# websocket应用场景?
以下都是我们生活周围普遍存在的场景：

1. 社交订阅
2. 体育实况更新
3. 股票基金报价
4. 多媒体聊天
5. 多玩家在线游戏
6. 等等...

以上的场景具有特征：1、即时通讯、2.服务器主动推送 、3.长链接

# 什么是websocket?

WebSocket是基于TCP的一种新的网络协议。它实现了浏览器与服务器全双工通信。

> 什么是双工通信？
双工通信简单举例来说就是在同一时刻信息可以进行双向传输，和打电话一样，说的同时也能听，边说边听。 

## 实现原理
在实现websocket连线过程中，需要通过浏览器发出websocket连线请求，然后服务器发出回应，这个过程通常称为“握手” 。浏览器和服务器只需要做一个握手的动作，然后之间就形成了一条快速通道，两者之间就可以互相传送数据。这样为我们实现即时服务带来了两大好处：

1. 互相沟通的Header是很小的-大概只有 2 Bytes
2. 服务器的推送，服务器不再被动的接收到浏览器的请求之后才返回数据，而是在有新数据时就主动推送给浏览器。

# 由轮询到WebSocket?

WebSocket未出现之前，如何实现主动推送的呢？

1.轮询
> 客户端和服务器之间会一直进行连接，每隔一段时间就询问一次。客户端会轮询，有没有新消息。这种方式连接数会很多，一个接受，一个发送。而且每次发送请求都会有Http的Header，会很耗流量，也会消耗CPU的利用率。

2.长轮询
> 长轮询是对轮询的改进版，客户端发送HTTP给服务器之后，有没有新消息，如果没有新消息，就一直等待。当有新消息的时候，才会返回给客户端。在某种程度上减小了网络带宽和CPU利用率等问题。但是这种方式还是有一种弊端：例如假设服务器端的数据更新速度很快，服务器在传送一个数据包给客户端后必须等待客户端的下一个Get请求到来，才能传递第二个更新的数据包给客户端，那么这样的话，客户端显示实时数据最快的时间为2×RTT（往返时间），而且如果在网络拥塞的情况下，这个时间用户是不能接受的，比如在股市的的报价上。另外，由于http数据包的头部数据量往往很大（通常有400多个字节），但是真正被服务器需要的数据却很少（有时只有10个字节左右），这样的数据包在网络上周期性的传输，难免对网络带宽是一种浪费。

传统HTTP客户端与服务器请求响应模式如下图所示：

![](https://raw.githubusercontent.com/yangjianan/workFile/master/img/01.jpg)

3.WebSocket（即时通讯，目的替代轮询）
> 支持客户端和服务器端的双向通信，而且协议的头部又没有HTTP的Header那么大，WebSocket流量消耗方面，相同的每秒客户端轮询的次数，当次数高达数万每秒的高频率次数的时候，WebSocket消耗流量仅为轮询的几百分之一。

WebSocket模式客户端与服务器请求响应模式如下图：

![](https://raw.githubusercontent.com/yangjianan/workFile/master/img/02.jpg)


# API介绍使用

![](https://raw.githubusercontent.com/yangjianan/workFile/master/img/4.png)

列子：
	
	<!DOCTYPE html>
	<html>
		<head>
			<meta charset="UTF-8">
			<title></title>
		</head>
		<body>
			<div>
				<a href="javascript:WebSocketConnect()">运行连接 WebSocket</a>
			</div>
				
			<div>
				<input id="text" type="text" />
				<a href="javascript:sendMsg()">发送信息</a>
				<a href="javascript:closeWebSocket()">关闭连接</a>
			</div>
			
			<div>
				<a href="javascript:closeWebSocket()">关闭连接</a> 
			</div>
		</body>
		
		<script type="text/javascript">
			
	      	//创建一个websocket实例
			var websocket = new WebSocket("ws://localhost:8080/webSocket"); //服务端访问地址
			
			//检查是否支持、是否连接成功
			function WebSocketConnect() {
				
				if("WebSocket" in window) {
					
					alert("您的浏览器支持 WebSocket!");
					
					//WebSocket事件:
					//连接成功建立的回调	
					websocket.onopen = function() {
						alert("连接成功");
					};
			
					//接收到消息的回调
					websocket.onmessage = function(event) {
						var respMsg = event.data;
						alert("数据已接收" + respMsg);
					};
			
					//连接发生错误的回调  
					websocket.onerror = function() {
						alert("error");
					};
			
					//连接关闭的回调  
					websocket.onclose = function() {
						alert("连接已关闭");
					}
			
				} else {
					// 浏览器不支持 WebSocket
					alert("您的浏览器不支持 WebSocket!");
				}
			}
	
			//WebSocket方法：
			//发送消息  
			function sendMsg() {
				var message = document.getElementById('text').value;
				websocket.send(message);
			}
			
			//关闭连接  
			function closeWebSocket() {
				websocket.close();
			}

			//监听窗口关闭事件，当窗口关闭时，主动去关闭websocket连接，防止连接还没断开就关闭窗口，server端会抛异常。  
			window.onbeforeunload = function() {
				websocket.close();
			}

		</script>
	</html>


# WebSocket和Socket的区别

区别：
> Socket是传输控制层协议，WebSocket是应用层协议。

Socket是应用层与TCP/IP协议族通信的中间软件抽象层，它是一组接口。在设计模式中，Socket其实就是一个门面模式，它把复杂的TCP/IP协议族隐藏在Socket接口后面，对用户来说，一组简单的接口就是全部，让Socket去组织数据，以符合指定的协议。

比如：主机 A 的应用程序要能和主机 B 的应用程序通信，必须通过 Socket 建立连接，而建立 Socket 连接必须需要底层 TCP/IP 协议来建立 TCP 连接。建立 TCP 连接需要底层 IP 协议来寻址网络中的主机。我们知道网络层使用的 IP 协议可以帮助我们根据 IP 地址来找到目标主机，但是一台主机上可能运行着多个应用程序，如何才能与指定的应用程序通信就要通过 TCP 或 UPD 的地址也就是端口号来指定。这样就可以通过一个 Socket 实例唯一代表一个主机上的一个应用程序的通信链路了。

而 WebSocket 则不同，它是一个完整的 应用层协议，包含一套标准的 API 。
所以，从使用上来说，WebSocket 更易用，而 Socket 更灵活。

# WebSocket兼容性

![](https://raw.githubusercontent.com/yangjianan/workFile/master/img/03.jpg)


# 扩展

实现WebSocket心跳重连 -> [跳转](http://mp.weixin.qq.com/s/sZ6v5jzHc7jqKE5QW5x4CA)


