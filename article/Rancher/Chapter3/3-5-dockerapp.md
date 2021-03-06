## 3.5、容器应用管理

#### 容器应用堆栈管理

支持使用应用模板一键创建容器应用栈。可以从应用集合，服务集合的角度对容器进行集中的管理和配置。

####应用管理视图

支持以不同方式查看、展现应用容器。至少支持根据命令空间Namespace、应用分类、容器运行节点、列表等方式展示容器列表。

#### 通过上传编排文件直接部署应用

支持通过上yaml编排 文件，一键部署和启动应用。极大提高部署效率，降低部署难度和时间。

#### 查看、下载编排配置文件

支持实时查看kubernetes编排文件配置内容。支持下载和保存kubernetes编排文件，方便配置编辑、保存、重新部署和故障恢复。

#### 统一调用非容器化服务或系统

管理平台可方便的实现容器访问外部的应用和服务，容器可以访问如不适合进行容器化的大型应用（如MySQL/Oracle数据库）。支持通过图形界面创建相应的服务发现记录，包括外部IP地址、外部主机名、DNS别名等多种方法