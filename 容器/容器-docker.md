### docker原理

#### 命名空间

让容器内的资源跟宿主隔离。Linux包含七种命名空间，CLONE_NEWCGROUP、CLONE_NEWIPC、CLONE_NEWNET、CLONE_NEWS、CLONE_NEWPID、CLONE_NEWUSER、CLONE_NEWUTS

#### 网络

分为四种none,bridge,container,Host。默认为网桥模式，会为所有容器设置IP地址、docker服务在宿主机上启动后会创建虚拟网桥docker0，该主机上启动的所有服务默认都会跟该网桥相连。通过iptables进行数据包转发。

#### AUFS

把多个文件系统联合到同一个挂载点的文件系统服务。

#### CGGROUP

通过GCGroup进行CPU和内存等系统资源的分配和隔离。