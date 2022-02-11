
<!-- Although we're done with our workshop, there's still a LOT more to learn about containers!
We're not going to go deep-dive here, but here are a few other areas to look at next! -->

虽然我们的教程已经结束了，但关于容器还有很多东西要学习！我们不打算在这里进行深入研究，但下面还有一些其他方面要看！

<!-- ## Container Orchestration -->
## 容器编排（Container Orchestration）
<!-- 
Running containers in production is tough. You don't want to log into a machine and simply run a
`docker run` or `docker-compose up`. Why not? Well, what happens if the containers die? How do you
scale across several machines? Container orchestration solves this problem. Tools like Kubernetes,
Swarm, Nomad, and ECS all help solve this problem, all in slightly different ways. -->

在生产中运行容器是困难的。当有1000台服务器的时候，你肯定不想登录到每一台机器，然后运行`docker run`或`docker compose up`挨个启动容器。如果容器进程挂了怎么办？你怎么跨多台机器进行扩容？
`容器编排`解决了这个问题，比如`Kubernetes`这样的工具，Swarm、Nomad和ECS都有助于解决这个问题，它们的方式略有不同。

<!-- The general idea is that you have "managers" who receive **expected state**. This state might be
"I want to run two instances of my web app and expose port 80." The managers then look at all of the
machines in the cluster and delegate work to "worker" nodes. The managers watch for changes (such as
a container quitting) and then work to make **actual state** reflect the expected state. -->
容器编排通常的工作思路是：你指定一个**预期状态**给"managers"。比如“我想运行两个web应用实例，并公开80端口”。然后"managers"会查看所有的集群中的机器，并将工作委托给“worker”节点。"managers"关注变化（比如容器退出），然后使**实际状态**反映预期状态。

译者注：docker compose是单机容器编排工具，Kubernetes(k8s)是集群容器编排工具。

<!-- ## Cloud Native Computing Foundation Projects -->
## 云计算基础基金项目（Cloud Native Computing Foundation Projects）

<!-- The CNCF is a vendor-neutral home for various open-source projects, including Kubernetes, Prometheus, 
Envoy, Linkerd, NATS, and more! You can view the [graduated and incubated projects here](https://www.cncf.io/projects/)
and the entire [CNCF Landscape here](https://landscape.cncf.io/). There are a LOT of projects to help
solve problems around monitoring, logging, security, image registries, messaging, and more! -->

CNCF是各种开源项目的供应商中立之家，包括Kubernetes、Prometheus、Envoy、Linkerd、NATS等等！你可以在这里查看[毕业和孵化项目](https://www.cncf.io/projects/)以及整个[CNCF景观](https://landscape.cncf.io/)。这里有很多项目，来帮助解决监控、日志记录、安全、镜像注册、消息传递等方面的问题！

<!-- So, if you're new to the container landscape and cloud-native application development, welcome! Please
connect to the community, ask questions, and keep learning! We're excited to have you! -->
因此，如果您对容器环境和云本机应用程序开发还不熟悉，欢迎您！请联系社区，提出问题，并不断学习！我们很高兴有你！