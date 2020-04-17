## GuestBook

典型的Hello World例子是在屏幕终端输出一句话“Hello World”，而这里的Hello World例子是一个Web留言板应用，并且是一个基于PHP+Redis的两层分布书架构的应用，前端PHP Web网站通过访问后端Redis数据库来完成用户留言的查询和添加等功能，更重要的是这个传统的经典案例部署在Kubernetes集群中，具备Redis读写分离能力。

留言板页面很简单，如图所示，首页将显示访客的留言，留言内容是从Redis中查询得到的，首页提供一个文本输入框允许访客添加留言，添加的留言将被写入Redis中。

[![3bNBIs.png](https://s2.ax1x.com/2020/03/06/3bNBIs.png)](https://imgchr.com/i/3bNBIs)

留言板的系统部署架构如图，为了实现读写分离，在Redis层采用了一个Master与两个Slave的高可用集群模式进行部署，其中Master实例用于前端写操作（添加留言），而两个Slave实例则用于前端读操作（读取留言），PHP的Web层同样启动3个实例组成集群，实现客户端对网站访问的负载均衡。

![3ba0Cn.png](https://s2.ax1x.com/2020/03/06/3ba0Cn.png)

- redis-master： 用于前端Web应用进行“写”留言的操作，其中已经保存了一条内容为“Hello World”的留言。
- guestbook-redis-slave：用于前端Web应用进行“读”留言的操作，并与 redis-master 的数据保持同步
- guestbook-php-frontend：PHP Web服务，在网页上展示留言内容，也提供一个文本输入框供访客添加留言。
