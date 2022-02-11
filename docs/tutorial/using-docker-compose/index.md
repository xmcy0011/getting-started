
<!-- [Docker Compose](https://docs.docker.com/compose/) is a tool that was developed to help define and
share multi-container applications. With Compose, we can create a YAML file to define the services
and with a single command, can spin everything up or tear it all down.  -->

[Docker Compose](https://docs.docker.com/compose)是一种能帮助我们开发和共享多容器应用程序的工具。使用Docker Compose，我们只需要创建一个YAML文件，通过一个命令，就可以启动所有容器。

<!-- The _big_ advantage of using Compose is you can define your application stack in a file, keep it at the root of
your project repo (it's now version controlled), and easily enable someone else to contribute to your project.  -->
使用Compose最大的优势在于，您可以通过一个YAML文件来定义您的应用程序容器，就可以轻松地让其他人为您的项目做出贡献。

<!-- Someone would only need to clone your repo and start the compose app. In fact, you might see quite a few projects
on GitHub/GitLab doing exactly this now. -->

因此，其他开发者只需要 clone 您的代码仓库，然后通过compose启动应用即可。事实上，您可能在GitHub/GitLab上会看到很多项目，现在就是这样做的。

<!-- So, how do we get started? -->
那么，我们如何开始呢？

<!-- ## Installing Docker Compose -->
## 安装Docker Compose

<!-- If you installed Docker Desktop/Toolbox for either Windows or Mac, you already have Docker Compose!
Play-with-Docker instances already have Docker Compose installed as well. If you are on 
a Linux machine, you will need to install Docker Compose using 
[the instructions here](https://docs.docker.com/compose/install/).  -->
如果您是**Windows或Mac平台**，只需要安装Docker Desktop/Toolbox工具即可！如果您是Linux机器，则需要单独安装Docker Compose，可以参考[此处](https://docs.docker.com/compose/install/)。

<!-- After installation, you should be able to run the following and see version information. -->
安装完成后，可以通过下面的命令查看是否安装成功：

```bash
$ docker-compose version
```


## 创建Compose File

1. 在项目根目录下，创建一个名为 `docker-compose.yml` 的文件。

1. 在这个文件的开头，我们通过 `version` 来定义compose file的版本。大多数情况下，我们可以使用最新的版本，具体可以参考：[Compose file reference](https://docs.docker.com/compose/compose-file/)，这篇教程使用3.7版本（PS：作者2022/01/07翻译时，已经出了3.8版本）

    ```yaml
    version: "3.7"
    ```

1. 接下来，我们用 `services` 标签来声明我们要编排的容器，他们都放在services节点下：

    ```yaml hl_lines="3"
    version: "3.7"

    services:
    ```

<!-- And now, we'll start migrating a service at a time into the compose file. -->
现在，我们将开始把应用迁移到compose文件中。


<!-- ## Defining the App Service -->
## 定义App Service

<!-- To remember, this was the command we were using to define our app container. -->

这是启动app容器的命令：

```bash
docker run -dp 3000:3000 \
  -w /app -v "$(pwd):/app" \
  --network todo-app \
  -e MYSQL_HOST=mysql \
  -e MYSQL_USER=root \
  -e MYSQL_PASSWORD=secret \
  -e MYSQL_DB=todos \
  node:12-alpine \
  sh -c "yarn install && yarn run dev"
```

<!-- If you are using PowerShell then use this command. -->
如果您使用的是PowerShell，请使用以下命令：

```powershell
docker run -dp 3000:3000 `
  -w /app -v "$(pwd):/app" `
  --network todo-app `
  -e MYSQL_HOST=mysql `
  -e MYSQL_USER=root `
  -e MYSQL_PASSWORD=secret `
  -e MYSQL_DB=todos `
  node:12-alpine `
  sh -c "yarn install && yarn run dev"
```

<!-- 1. First, let's define the service entry and the image for the container. We can pick any name for the service. 
   The name will automatically become a network alias, which will be useful when defining our MySQL service. -->
1. 首先，让我们为容器定义服务条目和镜像。我们可以为服务选择任何名称。该名称将自动成为网络别名，这在定义MySQL服务时非常有用。
    ```yaml hl_lines="4 5"
    version: "3.7"

    services:
      app:
        image: node:12-alpine
    ```

<!-- 1. Typically, you will see the command close to the `image` definition, although there is no requirement on ordering.
   So, let's go ahead and move that into our file. -->
1. compose file不要求排序，所以我们把上面docker命令中，紧挨着image的下一段命令也移动过来：

    ```yaml hl_lines="6"
    version: "3.7"

    services:
      app:
        image: node:12-alpine
        command: sh -c "yarn install && yarn run dev"
    ```


1. 同样的把 `-p 3000:3000` 移动到 `ports` 定义下，这里我们使用
[short syntax](https://docs.docker.com/compose/compose-file/compose-file-v3/#short-syntax-1) 语法, 另一种是 [long syntax](https://docs.docker.com/compose/compose-file/compose-file-v3/#long-syntax-1) 也同样适用。

    ```yaml hl_lines="7 8"
    version: "3.7"

    services:
      app:
        image: node:12-alpine
        command: sh -c "yarn install && yarn run dev"
        ports:
          - 3000:3000
    ```

1. 接下来，让我们使用 `working_dir` 来定义工作目录 (`-w /app`)， `volumes` 来定义卷映射 (`-v "$(pwd):/app"`)。volumes 同样有 [short syntax](https://docs.docker.com/compose/compose-file/compose-file-v3/#short-syntax-3) and [long syntax](https://docs.docker.com/compose/compose-file/compose-file-v3/#long-syntax-3) 2种语法。Docker Compose卷定义的一个优点是我们可以使用当前目录中的相对路径。

    ```yaml hl_lines="9 10 11"
    version: "3.7"

    services:
      app:
        image: node:12-alpine
        command: sh -c "yarn install && yarn run dev"
        ports:
          - 3000:3000
        working_dir: /app
        volumes:
          - ./:/app
    ```

1. 最后，我们使用 `environment` 来声明容器所需要的环境变量。

    ```yaml hl_lines="12 13 14 15 16"
    version: "3.7"

    services:
      app:
        image: node:12-alpine
        command: sh -c "yarn install && yarn run dev"
        ports:
          - 3000:3000
        working_dir: /app
        volumes:
          - ./:/app
        environment:
          MYSQL_HOST: mysql
          MYSQL_USER: root
          MYSQL_PASSWORD: secret
          MYSQL_DB: todos
    ```

  
<!-- ### Defining the MySQL Service -->
### 定义MySQL Service

<!-- Now, it's time to define the MySQL service. The command that we used for that container was the following: -->
这是启动 MySQL容器的命令：

```bash
docker run -d \
  --network todo-app --network-alias mysql \
  -v todo-mysql-data:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=secret \
  -e MYSQL_DATABASE=todos \
  mysql:5.7
```

PowerShell版本命令如下：

```powershell
docker run -d `
  --network todo-app --network-alias mysql `
  -v todo-mysql-data:/var/lib/mysql `
  -e MYSQL_ROOT_PASSWORD=secret `
  -e MYSQL_DATABASE=todos `
  mysql:5.7
```

<!-- 1. We will first define the new service and name it `mysql` so it automatically gets the network alias. We'll
   go ahead and specify the image to use as well. -->
1. 首先，我们需要一个新的服务，为了方便它自动获取网络别名，我们把这个服务命名为 `mysql`。同样的，我们把docker命令中镜像部分的内容移动过来：
    ```yaml hl_lines="6 7"
    version: "3.7"

    services:
      app:
        # The app service definition
      mysql:
        image: mysql:5.7
    ```

<!-- 1. Next, we'll define the volume mapping. When we ran the container with `docker run`, the named volume was created
   automatically. However, that doesn't happen when running with Compose. We need to define the volume in the top-level
   `volumes:` section and then specify the mountpoint in the service config. By simply providing only the volume name,
   the default options are used. There are [many more options available](https://docs.docker.com/compose/compose-file/#volume-configuration-reference) though. -->
1. 接下来，让我们定义卷容器和映射。当我们通过 `docker run` 启动容器时，命名卷容器（named volume）会被自动创建。然而，在docker compose中需要手动显示创建。我们需要创建一个和 `services` 同级的 `volumes` 节点，来指定如何挂载。我们可以仅指定卷名称，来使用默认的配置。

    ```yaml hl_lines="8 9 10 11 12"
    version: "3.7"

    services:
      app:
        # The app service definition
      mysql:
        image: mysql:5.7
        # todo-mysql-data：引用volumes下的卷容器，保持名称一致
        volumes:
          - todo-mysql-data:/var/lib/mysql
    
    volumes:
      todo-mysql-data:
    ```

1. 最后，我们通过 `environment` 来给容器提供一下环境变量：

    ```yaml hl_lines="10 11 12"
    version: "3.7"

    services:
      app:
        # The app service definition
      mysql:
        image: mysql:5.7
        volumes:
          - todo-mysql-data:/var/lib/mysql
        environment: 
          MYSQL_ROOT_PASSWORD: secret
          MYSQL_DATABASE: todos
    
    volumes:
      todo-mysql-data:
    ```

<!-- At this point, our complete `docker-compose.yml` should look like this: -->
现在，完整的 `docker-compose.yml` 文件看起来应该是这样的：

```yaml
version: "3.7"

services:
  app:
    image: node:12-alpine
    command: sh -c "yarn install && yarn run dev"
    ports:
      - 3000:3000
    working_dir: /app
    volumes:
      - ./:/app
    environment:
      MYSQL_HOST: mysql
      MYSQL_USER: root
      MYSQL_PASSWORD: secret
      MYSQL_DB: todos

  mysql:
    image: mysql:5.7
    volumes:
      - todo-mysql-data:/var/lib/mysql
    environment: 
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: todos

volumes:
  todo-mysql-data:
```

PS：
- services：声明了一组服务，它由容器app和mysql构成。服务启动时，会把这2个容器启动。启动后，在docker desktop中，会看到这2个容器放在了一个服务下面。
- volumes 和 services 一定是同级的，别搞错了。
- image 声明了启动容器所需的镜像，某些情况下，我们希望从源码编译，可以这样写：
```yaml
services:
  im_http: # http 服务
    container_name: im_http
    build: # 指定从dockerfile编译
      context: .
      dockerfile: app/im_http/Dockerfile
```


<!-- ## Running our Application Stack -->
## 启动服务

<!-- Now that we have our `docker-compose.yml` file, we can start it up! -->
现在，有了 `docker-compose.yml` 文件后，我们就可以启动它了！

<!-- 1. Make sure no other copies of the app/db are running first (`docker ps` and `docker rm -f <ids>`). -->
1. 确认没有其他名为 `app, mysql`的容器在运行(使用`docker ps` and `docker rm -f <ids>`)。

<!-- 1. Start up the application stack using the `docker-compose up` command. We'll add the `-d` flag to run everything in the background. -->
1. 然后通过 `docker-compose up`命令来启动所有应用程序（容器 或者说 一组容器），`-d` 标志代表让所有容器都在后台运行。

    ```bash
    docker-compose up -d
    ```

    运行后，应该能看到如下的输出：

    ```plaintext
    Creating network "app_default" with the default driver
    Creating volume "app_todo-mysql-data" with default driver
    Creating app_app_1   ... done
    Creating app_mysql_1 ... done
    ```

    <!-- You'll notice that the volume was created as well as a network! By default, Docker Compose automatically creates a 
    network specifically for the application stack (which is why we didn't define one in the compose file). -->
    您会注意到，第一行和第二行自动创建了卷容器和网络！默认情况下，Docker Compose会自动创建专门针对应用程序堆栈的网络（这就是为什么我们没有在compose文件中再定义网络的原因）。

<!-- 1. Let's look at the logs using the `docker-compose logs -f` command. You'll see the logs from each of the services interleaved
    into a single stream. This is incredibly useful when you want to watch for timing-related issues. The `-f` flag "follows" the
    log, so will give you live output as it's generated. -->
1. 让我们通过 `docker-compose logs -f` 命令来查看一下日志，您会看到不同容器的日志一起输出了，当您想要关注与时间相关的问题时，这非常有用。 `-f` 标志指示日志实时输出。

    如果出了点意外，您可能会看到如下的输出：

    ```plaintext
    mysql_1  | 2019-10-03T03:07:16.083639Z 0 [Note] mysqld: ready for connections.
    mysql_1  | Version: '5.7.27'  socket: '/var/run/mysqld/mysqld.sock'  port: 3306  MySQL Community Server (GPL)
    app_1    | Connected to mysql db at host mysql
    app_1    | Listening on port 3000
    ```

    <!-- The service name is displayed at the beginning of the line (often colored) to help distinguish messages. If you want to
    view the logs for a specific service, you can add the service name to the end of the logs command (for example,
    `docker-compose logs -f app`). -->
    服务名称显示在行的开头（通常为彩色），以帮助区分消息。如果您想查看特定服务的日志，您可以将服务名称添加到logs命令的末尾（例如，`docker-compose logs -f app`）

    !!! info "提示 - 服务依赖：App应用等待DB先启动"
        <!-- When the app is starting up, it actually sits and waits for MySQL to be up and ready before trying to connect to it.
        Docker doesn't have any built-in support to wait for another container to be fully up, running, and ready
        before starting another container. For Node-based projects, you can use the 
        [wait-port](https://github.com/dwmkerr/wait-port) dependency. Similar projects exist for other languages/frameworks. -->
        在实际项目中，我们希望应用程序等待MySQL先启动就绪后，再去尝试连接它。但是，Docker没有任何内置的支持来等待实现这一点。对于基于节点（Node-based）的项目，可以使用 [wait-port](https://github.com/dwmkerr/wait-port) ，其他语言/框架也有类似的项目。

<!-- 1. At this point, you should be able to open your app and see it running. 
And hey! We're down to a single command! -->
1. 此时，您应该能够打开应用程序并看到它正在运行。`嘿！我们只有一个命令！`

<!-- ## Seeing our App Stack in Docker Dashboard -->

## 通过Docker Dashboard查看应用程序

<!-- If we look at the Docker Dashboard, we'll see that there is a group named **app**. This is the "project name" from Docker
Compose and used to group the containers together. By default, the project name is simply the name of the directory that the
`docker-compose.yml` was located in. -->
如果我们打开Docker Dashboard，我们可以看到一个`“app”`的分组，它来自于Docker Compose中的"project name"的值，而这个值通常和`docker-compose.yml`中指定的值保持一致。

![Docker Dashboard with app project](dashboard-app-project-collapsed.png)

<!-- If you twirl down the app, you will see the two containers we defined in the compose file. The names are also a little
more descriptive, as they follow the pattern of `<project-name>_<service-name>_<replica-number>`. So, it's very easy to
quickly see what container is our app and which container is the mysql database. -->

展开app后，您会看到我们在compose文件中定义的两个容器。名字的生成规则是`<project-name>_<service-name>_<replica-number>`，因此，我们可以很容易的快速查看应用程序的容器以及mysql数据库的容器。

![Docker Dashboard with app project expanded](dashboard-app-project-expanded.png)


<!-- ## Tearing it All Down -->
## 停止并拆掉所有容器

<!-- When you're ready to tear it all down, simply run `docker-compose down` or hit the trash can on the Docker Dashboard 
for the entire app. The containers will stop and the network will be removed. -->

当您准备把它全部拆掉时（译者注：停止容器组内所有容器，并且移除），只需运`docker compose down`或点击Docker Dashboard `app` 对应的垃圾桶。即可停止并删除所有容器，同时容器网络也将被移除。

!!! warning 移除容器卷（Volumes）
    <!-- By default, named volumes in your compose file are NOT removed when running `docker-compose down`. If you want to
    remove the volumes, you will need to add the `--volumes` flag. -->
    默认情况下, 当您运行`docker-compose down`时，命名容器卷（named volumes）通常不会被移除，除非您指定`--volumes`标志。
    同样的，当您从 Docker Dashboard 中删除 `app` 时，也不会自动移除命名容器卷（named volumes）。
    <!-- The Docker Dashboard does _not_ remove volumes when you delete the app stack. -->

<!-- Once torn down, you can switch to another project, run `docker-compose up` and be ready to contribute to that project! It really
doesn't get much simpler than that! -->

一旦拆除，您就可以切换到另一个项目，运行`docker compose up`，并准备为该项目做出贡献！真的是再简单不过了！

## 总结

<!-- In this section, we learned about Docker Compose and how it helps us dramatically simplify the defining and
sharing of multi-service applications. We created a Compose file by translating the commands we were
using into the appropriate compose format. -->

在本节中，我们学习了`Docker Compose`，以及了解到它是如何帮助我们极大地简化定义和操作共享多服务应用程序的：只需要创建一个Compose文件，它会帮助我们翻译docker所需要的命令。

<!-- At this point, we're starting to wrap up the tutorial. However, there are a few best practices about
image building we want to cover, as there is a big issue with the Dockerfile we've been using. So,
let's take a look! -->
至此，我们的整个教程就结束了。然而，还有一个我们想讨论的话题：`镜像构建最佳实践`，因为前面的教程中，我们使用Dockerfile创建镜像还存在一个大问题，所以让我们一起看一看！