# PyCharm + docker+ Deepo 远程调试环境搭建教程

本文主要参考链接：

1、[PyCharm+Docker：打造最舒适的深度学习炼丹炉](https://zhuanlan.zhihu.com/p/52827335)

2、[Docker，救你于「深度学习环境配置」的苦海](https://zhuanlan.zhihu.com/p/64493662)

3、[ufoym/deepo](ufoym/deepo)

4、[nvidia-docker](https://github.com/NVIDIA/nvidia-docker)

5、[PyCharm实现高效远程调试代码](https://www.cnblogs.com/xuegqcto/p/8621689.html)

## 一、工作站需要安装软件：

- Ubuntu 18.04(推荐) 或者16.04
- Docker
- Nvidia显卡驱动(不需要安装cuda和cudnn)

## 二、显卡驱动安装过程

1. 禁用第三方显示驱动

   默认安装了第三方开源的驱动程序nouveau，安装Nvidia显卡驱动首先需要禁用nouveau，不然会碰到冲突的问题，导致无法安装nvidia显卡驱动。

编辑文件`blacklist.conf`

`sudo vim /etc/modprobe.d/blacklist.conf`

若未安装vim则`sudo apt-get install vim`安装或使用`vi`

在文件最后部分插入以下两行内容

```
blacklist nouveau
```

```
options nouveau modeset=0
```

更新系统

```
sudo update-initramfs -u
```

重启系统（一定要重启）

```
 sudo reboot
```

由于开源显卡驱动被禁用，所以屏幕显示分辨率会比较差，字体窗口都会显得过大，如果出现这种情况，基本可以省略下一个步骤：

- 验证nouveau是否已禁用，没有信息显示，则已禁用成功

```
lsmod | grep nouveau
```

![img](https://img-blog.csdn.net/20170403121940850)



2. 直接在系统中查找“软件和更新”，在附加应用里面找到显卡的驱动（推荐430或者更新版本），如何点击应用更改，等待完成即可。

   ![](C:\Users\ruifeng\Desktop\2019-09-03 19-24-41屏幕截图.png)

3、完成后，打开一个终端，输入`nvidia-smi`，如果能显示一下信息，则安装成功。请确保CUDA的版本大于10.0

![1567510231340](C:\Users\ruifeng\AppData\Roaming\Typora\typora-user-images\1567510231340.png)

4、如果上述方法无法正确安装显卡驱动，那么请[参考网络](https://blog.csdn.net/u014682691/article/details/80605201)上的解决方法，这里不再赘述。

## 三、安装Docker

以下内容根据 [官方文档](https://docs.docker.com/engine/installation/linux/docker-ce/debian/) 修改而来。

如果你过去安装过 docker，先删掉，否则跳过这一步:

```
sudo apt-get remove docker docker-engine docker.io
```

首先安装依赖:

```
sudo apt-get install apt-transport-https ca-certificates curl gnupg2 software-properties-common
```

信任 Docker 的 GPG 公钥:

```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

对于 amd64 架构（64位）的计算机，添加软件仓库:

```bash
sudo add-apt-repository \
   "deb [arch=amd64] https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
```

最后安装

```
sudo apt-get update
sudo apt-get install docker-ce
```

等待完成，输入`docker --version`验证是否成功，成功安装会显示版本信息。

![1567510831510](C:\Users\ruifeng\AppData\Roaming\Typora\typora-user-images\1567510831510.png)

最后，把docker添加到用户组，免没出都需要`sudo`的烦恼

我们就为当前用户添加到`docker`属组即可。

- 如果还没有 docker group 就添加一个：

  ```
  sudo groupadd docker
  ```

- 将用户加入该 group 内。然后退出并重新登录就生效啦。

  ```
  sudo gpasswd -a ${USER} docker
  ```

- 重启 docker 服务

  ```
  sudo service docker restart
  ```

- 切换当前会话到新 group 或者重启 X 会话

  ```
  newgrp - docker
  ```

  > 注意:最后一步是必须的，否则因为 groups 命令获取到的是缓存的组信息，刚添加的组信息未能生效，所以 docker images 执行时同样有错。

### Docker最常用操作

#### 基本概念

image，镜像，是一个个配置好的环境。 container，容器，是image的具体实例。 image和container的关系，相当于面向对象中类与对象的关系。

**如何查询命令参数：** `docker`可以看docker客户端有那些基本命令； 对应每一条命令，想看看具体是做什么的，可以在后面加一个`--help`查看具体用法，例如对于run命令： `docker run --help`

### 1、容器的相关操作

#### 容器的创建

`docker run [-it] some-image` 

创建某个镜像的容器。

**注意，同一个镜像可以通过这种方式创建任意多个container.** 加上`-it`之后，可以创建之后，马上进入交互模式。

#### 容器的查看

列出当前运行的容器

```
docker ps
```

![1567512261695](C:\Users\ruifeng\AppData\Roaming\Typora\typora-user-images\1567512261695.png)

列出所有的容器，包括运行的和不运行的

```
docker ps -a
```

![1567512288152](C:\Users\ruifeng\AppData\Roaming\Typora\typora-user-images\1567512288152.png)

#### 容器的删除

docker rm container-id删除某个容器



### 2.容器的启动、进入、退出：

#### 启动容器

```
docker start [-i] container-id
```

**启动**某个容器，必须是已经创建的。 加上`-i` 参数之后，可以直接**进入**交互模式：

![img](https://pic3.zhimg.com/80/v2-e5215c59a47b48a83b2664345020dca6_hd.jpg)



除了通过`-i`**进入**交互模式，还有一种方法，那就是通过`attach`命令

```
docker attach container-id
```

![img](https://pic1.zhimg.com/80/v2-4aead64e35225bc2d30258bf7f7c9a30_hd.png)

#### 退出容器

进入交互模式之后，怎么退出呢： - 想退出但是保持容器运行，按`CTRL+Q+P`三个键 - 退出，并关闭停止容器，按`CTRL+D`或者输入`exit`再回车

注：`Ctrl+P+Q`按的时候有时候会不灵，多按几次！

#### 容器的停止、重启：

 `docker stop container-id` `docker restart container-id`



## 四、安装nvidia-docker

[官方链接](https://github.com/NVIDIA/nvidia-docker)

确保docker的版本为19.03，第一次使用Docker 19.03和GPUs，参考以下过程

### Ubuntu 16.04/18.04, Debian Jessie/Stretch/Buster

```
# Add the package repositories
$ distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
$ curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
$ curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list

$ sudo apt-get update && sudo apt-get install -y nvidia-container-toolkit
$ sudo systemctl restart docker
```

### 验证是否正确安装

```
docker run --runtime=nvidia --rm nvidia/cuda:9.0-base nvidia-smi
```

![1567511949078](C:\Users\ruifeng\AppData\Roaming\Typora\typora-user-images\1567511949078.png)

## 五、安装Deepo

输入以下命令，安装一个全家桶集合，详情[请查看](https://github.com/ufoym/deepo)

```
sudo docker pull registry.docker-cn.com/ufoym/deepo
```

现在您可以尝试以下命令：

```
nvidia-docker run --rm ufoym/deepo nvidia-smi
```

如果能看到下图，则成功了

![1567512091800](C:\Users\ruifeng\AppData\Roaming\Typora\typora-user-images\1567512091800.png)





## 六、创建可以远程访问的容器

首先创建一个交互容器，如下图，进入到容器里面，当前为root身份登录

```
docker run -it ufoym/deepo
```

![1567512980498](C:\Users\ruifeng\AppData\Roaming\Typora\typora-user-images\1567512980498.png)

先升级更新一下，输入

`apt-get update                  # apt-get 升级`

接着安装`openssh-server`，输入如下命令，

`apt install openssh-server`

修改SSH配置文件以下选项，去掉#注释，将四个选项启用：

```ruby
$ vi /etc/ssh/sshd_config

RSAAuthentication yes #启用 RSA 认证
PubkeyAuthentication yes #启用公钥私钥配对认证方式
AuthorizedKeysFile .ssh/authorized_keys #公钥文件路径（和上面生成的文件同）
PermitRootLogin yes #root能使用ssh登录
```

![img](https://upload-images.jianshu.io/upload_images/1255061-780f73fc18d30177.png?imageMogr2/auto-orient/strip|imageView2/2/w/994/format/webp)

重启ssh服务，并设置开机启动：

```ruby
service sshd restart
chkconfig sshd on
```

#### 更改root密码

使用如下命令，设置并记住你的root密码，后面远程连接需要用到

```
passwd
```

#### 退出容器并保存更改

使用exit命令或者ctrl+C来退出当前运行的容器：

```csharp
 exit
```

当结束后，我们使用 exit 来退出，现在我们的容器已经被我们改变了，使用 docker commit 命令来提交更新后的副本。

```ruby
sudo docker commit -m 'install openssh' -a 'Docker Newbee' f8513ec39154  deepo_ssh
```

其中，`-m` 来指定提交的说明信息，跟我们使用的版本控制工具一样；

`-a` 可以指定更新的用户信息；之后是用来创建镜像的容器的ID；最后指定目标镜像的仓库名和 tag 信息。创建成功后会返回这个镜像的 ID 信息。

提交后docker中就会多出一个deepo_ssh的一个镜像。

[这一部分可以参考博客](https://www.jianshu.com/p/426f0d8e6cbf)



安装好ssh服务后，可以尝试在远程使用jupyter notebook 和pycharm了



### Docker jupyter notebook 服务 

本部分[参考知乎](https://zhuanlan.zhihu.com/p/64493662)，并作了一些格式调整

深度学习jupyter notebook镜像已经创建：

![img](https://pic3.zhimg.com/80/v2-f665f047d4f735ec8db9aa9da954c9d6_hd.png)



### 1.如何创建自己的可以远程访问的容器：

```text
sudo nvidia-docker run -it -p 7777:8888 --ipc=host -v /home/shcd/Documents/gby:/gby --name gby-notebook  90be7604e476
```

其中：

- `-it`为直接进入交互式
-  `-p 7777:8888`是把主机的7777端口映射到容器的8888端口 
- `--ipc=host`可以让容器与主机共享内存 
- `--name xxxxx`可以给容器定义一个个性化名字
- `-v /home/shcd/Documents/gby:/gby`可以将主机上的`/home/shcd/Documents/gby`地址挂载到容器里，并命名为`/data`文件夹，这样这个文件夹的内容可以在容器和主机之间共享了。因为容器一旦关闭，容器中的所有改动都会清除，所以这样挂载一个地址可以吧容器内的数据保存到本地。 `- 90be7604e476`则是你安装的`jupyter`镜像的id，可以在刚刚`docker images`命令后面查看，当然你也可以直接写全名`ufoym/deepo:all-py36-jupyter`

经过上面的操作，你应该可以直接进入容器了，这时你用`ls`命令，应该可以看到一个新的文件夹`gby`产生了！

![img](https://pic2.zhimg.com/80/v2-b631b0cbf380dd181a48a40fbbca4f35_hd.png)



### 2.创建了容器之后，我们可以进而启动jupyter notebook：

使用如下命令

```text
jupyter notebook --no-browser --ip=0.0.0.0 --allow-root --NotebookApp.token= --notebook-dir='/gby'
```

其中： 

-  `--no-browser`即不通过浏览器启动，
- `--ip`指定容器的ip
- `--allow-root`允许root模型运行

- `--NotebookApp.token`可以指定jupyter 登录密码，可以为空 
-  `--notebook-dir='/gby'`指定jupyter的根目录

### 3.开启本地与服务器的端口映射，从而远程登录jupyter：

在**本地机器**上，执行如下命令：

```text
ssh username@host-ip -L 1234:127.0.0.1:7777
```

这样，可以将本地的`1234`端口，映射到服务器的`localhost`的`7777`端口（即你前面创建jupyter容器时候的指定的服务器端口） 这样，你在本地电脑的浏览器里输入`localhost:1234`，即可登录到服务器上的`jupyter notebook`了！



![img](https://pic4.zhimg.com/80/v2-58c1b172b8eb459662652f7f69df6547_hd.png)

![img](https://pic1.zhimg.com/80/v2-986c1899ba970ff49e6d3d218981e134_hd.jpg)



当我第一次看到这个画面的时候，简直激动地要跳起来！ **既能远程访问高性能服务器，又可以像在本地一样便捷地操作**，你说激动不激动你说激动不激动？

### 4.容器的备份

之前好不容易配置好的环境，突然被学校服务器要重装！？怎么办？ 你想到的一定是：**能不能把配置好的环境备份一份，后面直接重新加载进来？**

方法也很简单： 一般情况下，我们想备份的是容器，因为我们具体的配置都是在容器中进行的，而镜像一般都是直接在网上下载的，我们不做什么改动。

先通过`docker ps`或者`docker ps -a`来查看你想备份的容器的id， 然后通过：

```text
docker commit -p [your-container-id] [your-backup-name]
```

来将id为your-container-id的容器创建成一个镜像快照。

接着，你通过`docker images`就可以查看到刚刚创建好的镜像快照了。 然后，通过：

```text
docker save -o [path-you-want-to-save/your-backup-name.tar]] [your-backup-name]
```

把那个镜像打包成tar文件，保存到服务器上。 后面就可以把服务器上打包好的tar文件，下载到本地了。

恢复： `docker load -i your-backup-name.tar` `docker run -d -p 80:80 your-backup-name`

## 七、PyCharm的配置

[原文链接](https://zhuanlan.zhihu.com/p/52827335)

打开PyCharm`*Tools > Deployment > Configuration*`, 新建一个`*SFTP*`服务器，名字自己取：

![img](https://pic1.zhimg.com/80/v2-807caec22eb9db3b2fd9672441abc280_hd.jpg)

输入如下图配置，注意这里的端口是你刚刚设置的映射到容器22端口的宿主机中的端口，我这里使用的是8022，账号密码是你刚刚自己设置的，这里的Root Path设置一个远程docker容器里的路径:

![img](https://pic3.zhimg.com/80/v2-3feca89d7009d03fed135e82c9b7f15e_hd.jpg)

配置完点击*Test SFTP connection*,如果成功就恭喜你，可以进行下一步了。

最后在*Mappings*中配置路径，这里的路径是你本地存放代码的路径，与刚刚配置的*Root Path*相互映射（意思是Mapping里本机的路径映射到远程的Root Path）*，*方便以后在本地和远程docker中进行代码和其他文件同步。

![img](https://pic3.zhimg.com/80/v2-f465d3e377fd7290dc067f2dee34436a_hd.jpg)



####         在PyCharm里配置远程解释器

点击PyCharm的`File > Setting > Project > Project Interpreter`右边的设置按钮新建一个项目的远程解释器：

![img](https://pic3.zhimg.com/80/v2-9e1e60001c72dea1254dfbd759539186_hd.jpg)

![img](https://pic1.zhimg.com/80/v2-336a8f3f90fce03f8ab4d1fab9600154_hd.jpg)

配置完成以后在项目解释器界面就会出现如下图所示，可以看到此时已经完成远程解释器的本地化：

![img](https://pic4.zhimg.com/80/v2-ec57a3d1f720a829323d6976e7493fc7_hd.jpg)

配置完成以后需要等本地和远程的环境同步一下，到这里，恭喜你，可以用最舒服的姿势。。。写代码了。

配置完成以后的日常是这样的：

![img](https://pic2.zhimg.com/80/v2-a1653d8f384fd5b6e06adb5ad4887b85_hd.jpg)

左边是本地的文件，修改之后可以随时右键deployment->upload到远程主机，或者直接在本地调试运行；最右边是远程主机的文件，假如直接在远程修改了文件刷新一下同样可以右键下载到本地，但是我不建议这样做，因为这样很容易带来冲突（毕竟没有很好的版本控制）。**目前最好的实践是在局域网的服务器上，时延低，同步速度快。**保你安装的是专业版的PyCharm，只有专业版能进行远程调试。
