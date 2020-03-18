### Netty

#### 1.Spring 5、SpringBoot 、Zookeeper、Dubbo等框架都使用了Netty

**2.封装IO操作的框架**

### Java中IO的定义

Input、Output 是相对于内存而言 

read() 将数据input到内存中，write将数据从内存中output

### 阻塞IO、非阻塞IO、异步、同步

#### **阻塞**

使用accept()获取数据的时候，如果内存中没有准备好数据，就会阻塞等待

#### **非阻塞**

获取数据的时候，如果数据没有准备好，则直接返回，线程可以处理其他事情，

主线程轮询查看是否准备好，准备好后通知要获取数据的线程 

#### **同步**

获取数据的线程会一直阻塞到数据被读取到 （SocketServer中，只能读或者写） 

#### **异步**

获取数据的线程可以处理其他操作，而不用等待数据返回

#### BIO(Block IO) 同步阻塞IO NIO(Non-Block/New) 同步非阻塞(Selector机制)  AIO(Async) IO 异步非阻塞（事件监听机制）



### NIO三件套

##### 1.Buffer缓冲区





#####  2.Selector选择器





##### 3.Channel通道

