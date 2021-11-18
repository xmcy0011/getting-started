---
next_page: app.md
---

<!-- ## The command you just ran -->
## 你刚刚运行的命令

<!-- Congratulations! You have started the container for this tutorial! -->
恭喜你！你已经成功启动了容器。

<!-- Let's first explain the command that you just ran. In case you forgot, -->
让我们先解释一下你刚刚运行的命令。万一你忘了，

<!-- here's the command: -->
以下是命令：


```cli
docker run -d -p 80:80 docker/getting-started
```

<!-- You'll notice a few flags being used. Here's some more info on them: -->
你会注意到有一些参数，以下是关于它们的解释：

<!-- - `-d` - run the container in detached mode (in the background) -->
- `-d`：以分离模式（在后台）运行容器
<!-- - `-p 80:80` - map port 80 of the host to port 80 in the container -->
- `-p 80:80`：将主机的端口80映射到容器中的端口80
<!-- - `docker/getting-started` - the image to use -->
- `docker/getting-started`：要使用的镜像

<!-- !!! info "Pro tip"
    You can combine single character flags to shorten the full command.
    As an example, the command above could be written as:
    ```
    docker run -dp 80:80 docker/getting-started
    ``` -->
!!! info 提示
    您可以组合单个字符标志来缩短命令。
    例如，上面的命令可以写成：
    ```
    docker run -dp 80:80 docker/getting-started
    ```

<!-- ## The Docker Dashboard -->
## Docker仪表板

<!-- Before going too far, we want to highlight the Docker Dashboard, which gives
you a quick view of the containers running on your machine. It gives you quick
access to container logs, lets you get a shell inside the container, and lets you
easily manage container lifecycle (stop, remove, etc.).  -->
在深入介绍之前，我们想突出介绍一下Docker仪表板，它提供了让我们快速查看机器上运行容器的功能。并且它让你能很方便的查看容器日志，允许您直接通过shell进入容器，并允许您轻松管理容器生命周期（停止、移除等）。

<!-- To access the dashboard, follow the instructions for either 
[Mac](https://docs.docker.com/docker-for-mac/dashboard/) or 
[Windows](https://docs.docker.com/docker-for-windows/dashboard/). If you open the dashboard
now, you will see this tutorial running! The container name (`jolly_bouman` below) is a
randomly created name. So, you'll most likely have a different name. -->

要访问仪表板，请根据你的平台进行设置：
- [Mac](https://docs.docker.com/docker-for-mac/dashboard/)
- [Windows](https://docs.docker.com/docker-for-windows/dashboard/)

如果你打开仪表板，您将看到本教程正在运行！容器名称（下面的`jolly_bouman`）是一个随机创建名称。所以，你很可能有不同的名字。

![Tutorial container running in Docker Dashboard](tutorial-in-dashboard.png)


<!-- ## What is a container? -->
## 什么是容器？

<!-- Now that you've run a container, what _is_ a container? Simply put, a container is
simply another process on your machine that has been isolated from all other processes
on the host machine. That isolation leverages [kernel namespaces and cgroups](https://medium.com/@saschagrunert/demystifying-containers-part-i-kernel-space-2c53d6979504), features that have been 
in Linux for a long time. Docker has worked to make these capabilities approachable and easy to use. -->

现在你已经运行了一个容器，那么什么是容器呢？

简而言之，容器只是一个进程，一个与您主机上所有其他进程隔离的进程。

这种隔离技术利用了[内核命名空间和cgroups](https://medium.com/@saschagrunert/demystifying-containers-part-i-kernel-space-2c53d6979504)实现。这并不是新技术，在Docker之前，他们已经在Linux中呆了很长时间。Docker一直致力于使这些功能易于理解和使用。

<!-- !!! info "Creating Containers from Scratch"
    If you'd like to see how containers are built from scratch, Liz Rice from Aqua Security
    has a fantastic talk in which she creates a container from scratch in Go. While she makes
    a simple container, this talk doesn't go into networking, using images for the filesystem, 
    and more. But, it gives a _fantastic_ deep dive into how things are working.

    <iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/8fi7uSYlOdc" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe> -->

!!! info 从零开始创建容器
    如果您想了解容器是如何从零开始建造的，来自Aqua Security的Liz Rice
    有一个精彩的演讲，她使用Go来从头开始创建容器。这个简单的容器足够简单，以至于它不会涉及网络、文件系统等等。但是，它对你理解容器是如何工作的有十分惊人的帮助。

    PS：译者注，原文是Youtube链接，我转换成了B站链接分享给大家。
    <iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/8fi7uSYlOdc" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


<!-- ## What is a container image? -->
## 什么是镜像？

<!-- When running a container, it uses an isolated filesystem. This custom filesystem is provided 
by a **container image**. Since the image contains the container's filesystem, it must contain everything 
needed to run an application - all dependencies, configuration, scripts, binaries, etc. The 
image also contains other configuration for the container, such as environment variables,
a default command to run, and other metadata. -->

当我们运行容器时，它使用独立的文件系统。提供此自定义文件系统的东西，我们称之为容器镜像，简称“镜像”。

由于镜像包含容器的文件系统，所以它必须包含所有内容。例如运行应用程序、依赖项、配置、脚本、二进制文件等。

除此之外，镜像还包含容器的其他配置，如环境变量，要运行的默认命令和其他元数据等。

<!-- We'll dive deeper into images later on, covering topics such as layering, best practices, and more. -->
我们将稍后深入研究镜像，涵盖分层、最佳实践等主题。

!!! info
    <!-- If you're familiar with `chroot`, think of a container as an extended version of `chroot`. The
    filesystem is simply coming from the image. But, a container adds additional isolation not
    available when simply using chroot. -->
    如果您熟悉`chroot`，请将容器视为`chroot`的扩展版本。文件系统来自于镜像，但如果只是简单的使用`chroot`，容器的文件系统将不具备隔离性。

## 译者

github：https://github.com/xmcy0011
公众号：Go和分布式IM
最后更新日期：2021-11-18