## 4.5、创建K8S集群

现在创建第一个Kubernetes集群，可以使用自定义选项。你可以添加云主机、内部虚拟机或物理主机作为集群节点，节点可以运行任何一种或多种主流Linux发行版:

1、在全局视图下，点击菜单中的集群 , 并点击添加集群；

2、选择**自定义**，并设置集群名称,其他参数可不用修改，点击下一步；

![](https://cdn.cloudcared.cn/wp-content/uploads/2020/04/WX20200401.png)

3、选择节点运行的角色
默认会勾选`Worker`角色，根据需要可以一次勾选多种角色。比如，假设我只有一个节点，那就需要把所有角色都选择上，选择后上面的命令行会自动添加相应的命令参数

![](https://cdn.cloudcared.cn/wp-content/uploads/2020/04/WX20200403.png)

4、其他参数保持默认，点击命令行右侧的复制按钮，复制命令参数，如果是多台主机，根据角色的不同，需要复制多次。

5、登录预添加集群的主机，执行以上复制的命令；

![](https://cdn.cloudcared.cn/wp-content/uploads/2020/04/WX20200406.png)

6、在主机上执行完命令后，最后点击完成；

7、回到全局视图，可以查看集群的部署状态；

![](https://cdn.cloudcared.cn/wp-content/uploads/2020/04/WX20200407.png)

![](https://cdn.cloudcared.cn/wp-content/uploads/2020/04/WX20200408.png)

8、集群创建完成

![](https://cdn.cloudcared.cn/wp-content/uploads/2020/04/WX20200409.png)

![](https://cdn.cloudcared.cn/wp-content/uploads/2020/04/WX20200410.png)

