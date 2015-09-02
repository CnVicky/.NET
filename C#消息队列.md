##C#消息队列实践##

###突破Http协议的并发连接数限制###
在Http协议中，规定了同个Http请求的并发连接数最大为2. 这个数值，可谓是太小了。

而目前的浏览器，已基本不再遵循这个限制，但是Dot Net平台上的 System.Net 还是默认遵循了这个标准的。

从而造成了，在使用HttpWebRequset 或者 WebClient 利用多线程的方式，访问某个网站时，经常出现 连接被异常关闭 的错误，大大降低了效率。

这个限制的值，是可以自己设置或配置的。

System.Net.ServicePointManager.DefaultConnectionLimit 就是设置的地方。 可以根据实际情况，来设置这个值的大小，不过，建议不要超过1024，推荐为512，已经足够了。

当然，也可以直接在程序的 app.config中配置这个值。

此值设置后，只对以后发起的HTTP请求有效。

	ServicePointManager.DefaultConnectionLimit = int.MaxValue;

###分布式队列###

使用ThreadPool 和MSMQ 进行消息收发。MSMQ 是一个分布式队列，通过MSMQ **一个应用程序可以异步地与另外一个应用程序通信**。

典型的场景：向维护一个队列的MSMQ 服务器发送消息，MSMQ 发送方与MSMQ 服务器(特定队列)之间创建一个连接并向那个队列发送消息。一个MSMQ 接收器接收由MSMQ发送方发送的消息。MSMQ 接收方需要监听一个特定的队列以接收发送到这个队列上的消息。MSMQ服务器在MSMQ发送方和接收方之间起到了一个中转的作用，但MSMQ发送方不知道还有一个MSMQ接收方，反之亦然。

###内存队列###

特点：快

ConcurrentQueue<T>
问题：队列里没有消息的时候，队列的消费者不能让CPU空转，CPU空转会直接导致CPU占用100%，导致机器无法工作

解决办法：使用 BlockingCollection<T> 这种集合能提供在队列内无元素的时候block当前线程的功能

	private BlockingCollection<T> _queue = new BlockingCollection<T>(new ConcurrentQueue<T>());
	_queue.Add(message)//并发入队
	_queue.Take();//并发出队
	
	_queue.IsCompleted //判断是否完成或为空



