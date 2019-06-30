最近做了很久新项目上线问题比较多，这周三就遇到一个比较奇怪的，整个过程大概如下：
1. 早上六点多左右，前端同事联系后端老大，说我们宫格挂了，列表页和详情页都进不去。
2. 老大立马联系OPS排查问题，得知凌晨2:00左右一个机房的网络设备更换了，初步定位是由该问题导致的。
3. 八点左右老大重启机器，故障恢复。
虽然重启机器后故障恢复了，但是导致问题的根本原因还需要排查，我们选择了其中一台故障机器，首先在我们的日志平台看到该机器上的应用2点以后错误日志逐渐减少，3点之后基本没有错误日志，但是早上六点多还是不能访问，这显然与现象不符，由此我们怀疑不是应用层面的异常，或许是`tomcat`拒绝了连接。
<div align="center"> <img src="http://tc.ganzhiqiang.wang/1561867001.png" width="800px"><p>应用日志监控</p> </div>

我们通过另一个监控平台，发现服务从2:20点到7:20一直在报`Read Time Out`'，7:20以后都是`Connect Time Out`，
`Connect Time Out`一般是超出了服务的最大连接数
<div align="center"> <img src="http://tc.ganzhiqiang.wang/1561866137.png" width="800px"><p>FailedCause监控</p> </div>
可以看到，我们生产`Tomcat`的配置最大连接数是`10000`，也就是7:20以后请求超过了NIO最大连接数，请求直接被拒绝了。
```xml
<!--使用了NIO协议-->
<Connector port="${port.http.server}" protocol="org.apache.coyote.http11.Http11NioProtocol"
          ...
          maxThreads="1024"   <!--请求处理的最大线程数-->
          maxConnections="10000"  <!--Tomcat在任意时刻接收和处理的最大连接数-->
          acceptCount="150" <!--accept队列的长度；当accept队列中连接的个数达到acceptCount时，队列满，进来的请求一律被拒绝-->
          ...
          />
```
## 下载Dump文件
那么为什么会有如此大连接数，难道之前的请求一直都没有结束或者一直没有释放，于是我们下载了这台机器的`ThreadDump`，发现2:17左右http线程数开始飙升，一直达到`tomcat`最大线程数`1024`，直到8:00点服务器重启一直都没有再降过，另一个重要发现是处于`WAITTING`状态的线程随着线程增多而增多，所以请求线程一直没有被释放，而是被挂起了，到此问题就确定了，只需要确定线程为什么被释放即可。
<div align="center"> 
<img src="http://tc.ganzhiqiang.wang/1561884170.png" width="500px"><p>线程数监控</p>
<img src="http://tc.ganzhiqiang.wang/1561884025.png" width="500px"><p>线程状态监控</p> </div>

详细看一下`dump`文件记录的堆栈消息，进一步确定了问题代码可能在`doPost`中
<div align="center"> <img src="http://tc.ganzhiqiang.wang/1561884561.png" width="800px"><p>堆栈信息</p> </div>

翻了一下代码，最终找到了**元凶**，原来是如果请求不是返回`200`，就会抛出`IOException`，但是没有关掉http链接，导致http线程被挂起，不能处理新请求，也不能被回收，导致了http线程数一直攀升。总结一下，网络设备更换后导致大量请求失败，随后大量http线程被挂起，导致最后`tomcat`拒绝了连接。
<div align="center"> <img src="http://tc.ganzhiqiang.wang/1561885060.png" width="800px"></div>
## HttpClient优化
**配置**






