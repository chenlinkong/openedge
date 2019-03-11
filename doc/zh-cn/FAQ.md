本文主要提供OpenEdge在各系统、平台部署、启动的相关问题及解决方案。

**问题1**: 在以容器模式启动OpenEdge时，提示缺少启动依赖配置项。

![图片](../images/setup/docker-engine-conf-miss.png)

**参考方案**: 如上图所示，OpenEdge启动缺少配置依赖文件，参考[OpenEdge设计文档](./overview/OpenEdge-design.md)及[GitHub项目开源包](https://github.com/baidu/openedge)example文件夹补充相应配置文件即可。

**问题2**: Ubuntu/Debian 下输入命令 ```docker info``` 后显示如下信息：

```
WARNING: No swap limit support
```

**参考方案**:

1. 修改 ```/etc/default/grub``` 文件，在其中，编辑或者添加 ```GRUB_CMDLINE_LINUX``` 为如下内容：
	
	> GRUB_CMDLINE_LINUX="cgroup_enable=memory swapaccount=1"

2. 保存后执行命令 ```sudo update-grub```，完成后重启系统生效。

***注：***如果执行第二步时提示出错，可能是 grub 设置有误，请检查后重复步骤1和步骤2.

**问题3**: WARNING: Your kernel does not support swap limit capabilities. Limitation discarded.

**参考方案**: 参考问题2。


**问题4**: Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get http://%2Fvar%2Frun%2Fdocker.sock/v1.38/images/json: dial unix /var/run/docker.sock: connect: permission denied.

1. 提供管理员权限
2. 通过以下命令添加当前用户到docker用户组：

```shell
sudo usermod -aG docker ${USER}
su - ${USER}
``` 

如提示没有 docker group，使用如下命令创建新docker用户组后再执行上述命令：

```shell
sudo groupadd docker
```

**问题5**: Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?

按照问题4解决方案执行后如仍报出此问题，重新启动docker服务即可。

例，CentOs 下启动命令：

```shell
systemctl start docker
```

**问题6**: failed to create master: Error response from daemon: client version 1.39 is too new. Maximum supported API version is 1.38.

设置环境变量DOCKER_API_VERSION=1.38

例，CentOs 下启动命令：

```shell
export DOCKER_API_VERSION=1.38
```

**问题7**: BIE如何接入NB-IoT?

NB-IoT是一种网络制式，和2/3/4G类似，带宽窄功耗低。NB-IoT支持基于TCP的MQTT通信协议，因此可以使用NB-IoT卡连接百度云物接入，部署OpenEdge应用和BIE云管理通信。但国内三大运营商中，电信对他们的NB卡做了限制，仅允许连接电信的云服务 IP，所以目前只能使用移动NB卡和联通NB卡连接百度云服务。

**问题8**: var/run/openedge.sock: address already in use

删除var/run/openedge.sock，重启OpenEdge。

**问题9**: OpenEdge支持数据计算后将计算结果推送Kafka吗？

支持，您可以[利用Python运行时编写Python脚本](https://github.com/baidu/openedge/blob/master/doc/zh-cn/customize/How-to-write-a-python-script-for-python-runtime.md)，向Hub订阅消息，并将消息逐个写入Kafka。您也可以[自定义模块](https://github.com/baidu/openedge/blob/master/doc/zh-cn/customize/How-to-develop-a-customize-module-for-openedge.md)，该模块向Hub订阅消息，然后批量写入Kafka。
      
**问题10**: Openedge配置更改的方式有哪些？只能通过[云端管理套件](https://cloud.baidu.com/product/bie.html)进行配置更改吗？

目前，我们推荐通过云端管理套件进行配置定义和下发，但您也可以手动更改核心设备上的配置文件，然后重启Openedge使之生效。