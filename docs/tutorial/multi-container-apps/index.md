
到目前为止，我们一直在使用单个应用程序容器。但是现在，我们想将MySQL添加到应用程序中。那么你可能会问：  

- MySQL将在哪里运行？  
- 如果是在容器中运行，那应该是和应用程序放在同一个容器，还是应该单独启动一个MySQL容器？

一般来说，`每个容器都应该只做一件事（译者注：如只运行一个进程），并把它做好`。一些原因如下：
<!-- - There's a good chance you'd have to scale APIs and front-ends differently than databases
- Separate containers let you version and update versions in isolation
- While you may use a container for the database locally, you may want to use a managed service
  for the database in production. You don't want to ship your database engine with your app then.
- Running multiple processes will require a process manager (the container only starts one process), 
  which adds complexity to container startup/shutdown -->
- 您很有可能使用不同的数据库，来扩展API和前端应用
- 你可以给容器打上版本号，并且独立更新其版本
- 虽然您可以为容器在本地创建数据库，但在生产中您可能有独立的数据库服务。那么，您肯定不希望应用程序发布时，还需要带一个多余的数据库引擎。
- 运行多个进程需要一个进程管理器（容器只启动一个进程），这增加了容器启动/关机的复杂性

<!-- And there are more reasons. So, we will update our application to work like this: -->
因此，我们将把应用改成如下多容器运行方式（译者注：2个容器，1个运行Todo App，1个运行MySQL）：

![Todo App connected to MySQL container](multi-app-architecture.png)
{: .text-center }


<!-- ## Container Networking -->
## 容器网络

<!-- Remember that containers, by default, run in isolation and don't know anything about other processes
or containers on the same machine. So, how do we allow one container to talk to another? The answer is
**networking**. Now, you don't have to be a network engineer (hooray!). Simply remember this rule... -->

默认情况下，容器是孤立运行的，对其他进程或同一机器上的其他容器一无所知。那么，我们如何让一个容器与另一个容器通信呢？答案是`网络`。现在，你不必成为一名网络工程师（万岁！）。只要记住这条规则：

> 如果两个容器位于同一网络上，它们就可以相互通信。反之，就不能。

<!-- ## Starting MySQL -->
## 启动MySQL

<!-- There are two ways to put a container on a network: 1) Assign it at start or 2) connect an existing container.
For now, we will create the network first and attach the MySQL container at startup. -->

将容器连接到网络有两种方法：

1. 在容器启动时给它分配一个网络。
2. 连接现有容器的网络，共用一个网络。

现在，我们使用第一种方式，首先创建网络，然后在启动MySQL容器时，连接这个网络。

1. 创建网络

    ```bash
    docker network create todo-app
    ```

1. 启动MySQL容器，然后把它附加到一个网络上。同时我们会使用一些环境变量，来初始化MySQL数据库（请参考：  [MySQL Docker Hub listing](https://hub.docker.com/_/mysql/) 中 "Environment Variables" 一节)：


    ```bash
    docker run -d \
        --network todo-app --network-alias mysql \
        -v todo-mysql-data:/var/lib/mysql \
        -e MYSQL_ROOT_PASSWORD=secret \
        -e MYSQL_DATABASE=todos \
        mysql:5.7
    ```

    如果你正在使用 PowerShell，请使用下面的命令：

    ```powershell
    docker run -d `
        --network todo-app --network-alias mysql `
        -v todo-mysql-data:/var/lib/mysql `
        -e MYSQL_ROOT_PASSWORD=secret `
        -e MYSQL_DATABASE=todos `
        mysql:5.7
    ```

    你可能会注意，我们指定了一个 `--network-alias` 标志，过一会我们会再谈这个问题。

    !!! info "提示"
        <!-- You'll notice we're using a volume named `todo-mysql-data` here and mounting it at `/var/lib/mysql`, which is
        where MySQL stores its data. However, we never ran a `docker volume create` command. Docker recognizes we want
        to use a named volume and creates one automatically for us. -->
        您可能注意到了，我们在这里使用了一个名为“todo-mysql-data”的数据卷，并将其挂载在“/var/lib/mysql”上，即MySQL存储数据的地方。但是，我们从未运行过“docker volume create”命令显示创建这个数据卷。不用担心，Docker会自动为我们创建。

1. 请确认我们的数据库启动成功，并且处于运行状态，此时我们就可以使用下面的命令来进行验证了：

    ```bash
    docker exec -it <mysql-container-id> mysql -p
    ```

    <!-- When the password prompt comes up, type in **secret**. In the MySQL shell, list the databases and verify you see the `todos` database. -->
    当终端提示您要输入密码的时候，请输入上面启动MySQL Shell中 **secret** 对应的您的真实密码。然后通过下面的命令列出数据库，并确认是否看到了“todos”数据：

    ```cli
    mysql> SHOW DATABASES;
    ```

    <!-- You should see output that looks like this: -->
    您应该看到类似下面的输出：

    ```plaintext
    +--------------------+
    | Database           |
    +--------------------+
    | information_schema |
    | mysql              |
    | performance_schema |
    | sys                |
    | todos              |
    +--------------------+
    5 rows in set (0.00 sec)
    ```

    <!-- Hooray! We have our `todos` database and it's ready for us to use! -->
    好极了！我们的 `todos` 应用有了数据库，并且已经准备好可以使用了！


<!-- ## Connecting to MySQL -->
## 连接到MySQL容器

<!-- Now that we know MySQL is up and running, let's use it! But, the question is... how? If we run
another container on the same network, how do we find the container (remember each container has its own IP
address)? -->
现在我们知道MySQL容器已经启动并运行，让我们来使用它吧！

但问题是。。。如果我们的容器跑在同个一网络上，我们该如何找到这个容器（请注意，每个容器都有自己的IP地址，因此这2个容器的IP地址是不同的）?

<!-- To figure it out, we're going to make use of the [nicolaka/netshoot](https://github.com/nicolaka/netshoot) container,
which ships with a _lot_ of tools that are useful for troubleshooting or debugging networking issues. -->
为了解决这个问题，我们将使用 [nicolaka/Netshot](https://github.com/nicolaka/netshoot) 容器镜像，它附带了很多工具，这些工具对于解决或调试网络问题非常有用。

<!-- 1. Start a new container using the nicolaka/netshoot image. Make sure to connect it to the same network. -->
1. 从 **nicolaka/netshoot** 镜像启动一个新的容器，并请确认他们连接到了同一个网络上：
    ```bash
    docker run -it --network todo-app nicolaka/netshoot
    ```

1. 在容器内部，我们将使用'dig'命令，这是一个很有用的DNS工具。我们用它来寻找主机名为“mysql”的IP地址。

    ```bash
    dig mysql
    ```

    <!-- And you'll get an output like this... -->
    您会看到类似下面的输出：

    ```text
    ; <<>> DiG 9.14.1 <<>> mysql
    ;; global options: +cmd
    ;; Got answer:
    ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 32162
    ;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0

    ;; QUESTION SECTION:
    ;mysql.				IN	A

    ;; ANSWER SECTION:
    mysql.			600	IN	A	172.23.0.2

    ;; Query time: 0 msec
    ;; SERVER: 127.0.0.11#53(127.0.0.11)
    ;; WHEN: Tue Oct 01 23:47:24 UTC 2019
    ;; MSG SIZE  rcvd: 44
    ```

    <!-- In the "ANSWER SECTION", you will see an `A` record for `mysql` that resolves to `172.23.0.2`
    (your IP address will most likely have a different value). While `mysql` isn't normally a valid hostname,
    Docker was able to resolve it to the IP address of the container that had that network alias (remember the
    `--network-alias` flag we used earlier?). -->
    在 “ANSWER SECTION” 部分，您将看到一个`mysql. 600 IN A 172.23.0.2`的记录（您的IP地址很可能具有不同的值）。虽然“mysql”通常不是有效的主机名，但是Docker仍然能够将其解析为具有该网络别名的容器的IP地址（请回忆一下我们之前使用的 `--network-alias` 标志？）。

    <!-- What this means is... our app only simply needs to connect to a host named `mysql` and it'll talk to the
    database! It doesn't get much simpler than that! -->
    这意味着。。。我们的应用程序`只需要连接到一个名为“mysql”的主机，它就可以与数据库通信`，是不是再简单不过了？！
    

<!-- ## Running our App with MySQL -->
## 运行我们的程序并连接MySQL

<!-- The todo app supports the setting of a few environment variables to specify MySQL connection settings. They are: -->
todo应用程序支持设置一些环境变量来指定MySQL的设置：

- `MYSQL_HOST`：处于运行状态MySQL Server的主机名
- `MYSQL_USER`：连接MySQL的用户名
- `MYSQL_PASSWORD`：连接MySQL的用户密码
- `MYSQL_DB`：连接MySQL的数据库

!!! warning 请谨慎使用环境变量设置连接信息
    <!-- While using env vars to set connection settings is generally ok for development, it is **HIGHLY DISCOURAGED**
    when running applications in production. Diogo Monica, the former lead of security at Docker, 
    [wrote a fantastic blog post](https://diogomonica.com/2017/03/27/why-you-shouldnt-use-env-variables-for-secret-data/)
    explaining why.  -->
    使用环境变量来设置连接信息，对于测试环境通常是可以的，但是**在生产环境中运行应用程序时**，非常不鼓励这样做。Diogo Monica，Docker的前安全主管，[写了一篇精彩的博文](https://diogomonica.com/2017/03/27/why-you-shouldnt-use-env-variables-for-secret-data/)解释原因。
    
    <!-- A more secure mechanism is to use the secret support provided by your container orchestration framework. In most cases,
    these secrets are mounted as files in the running container. You'll see many apps (including the MySQL image and the todo app)
    also support env vars with a `_FILE` suffix to point to a file containing the variable. -->

    更安全的机制是使用容器编排框架提供的secret支持。在大多数情况下，这些secrets会作为文件装入正在运行的容器中。您同样能看到许多应用程序（包括MySQL镜像和todo应用程序）通过`_FILE`后缀的环境变量，来指定从该文件中加载环境变量。
    
    <!-- As an example, setting the `MYSQL_PASSWORD_FILE` var will cause the app to use the contents of the referenced file 
    as the connection password. Docker doesn't do anything to support these env vars. Your app will need to know to look for
    the variable and get the file contents. -->
    例如，设置 `MYSQL_PASSWORD_FILE` 变量将导致应用程序使用指定的文件内容作为连接密码。Docker没有做任何事情来支持这些环境变量，您的应用程序需要自行处理这个环境变量，然后加载文件内容。


<!-- With all of that explained, let's start our dev-ready container! -->
解释完所有这些之后，让我们启动准备就绪的容器吧！

<!-- 1. We'll specify each of the environment variables above, as well as connect the container to our app network. -->
1. 我们将指定上面每个环境变量的值，并将容器连接到我们的应用程序网络：
    ```bash hl_lines="3 4 5 6 7"
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

    如果你正在使用 PowerShell，请使用下面的命令：

    ```powershell hl_lines="3 4 5 6 7"
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

1. 如果我们查看容器的日志（`docker logs <container id>`），我们应该会看到一条消息，表明它正在使用mysql数据库。
    ```plaintext hl_lines="7"
    # Previous log messages omitted
    $ nodemon src/index.js
    [nodemon] 1.19.2
    [nodemon] to restart at any time, enter `rs`
    [nodemon] watching dir(s): *.*
    [nodemon] starting `node src/index.js`
    Connected to mysql db at host mysql
    Listening on port 3000
    ```

1. 打开应用程序。请在浏览器输入：http://localhost:3000，然后请添加一些待办事项。

1. 连接到MySQL数据库，确定您添加的待办事项已经成功保存进MySQL数据库中了。请注意，密码是**secret**。
    ```bash
    docker exec -it <mysql-container-id> mysql -p todos
    ```

    然后，在MySQL Shell模式下，请运行下面的命令：

    ```plaintext
    mysql> select * from todo_items;
    +--------------------------------------+--------------------+-----------+
    | id                                   | name               | completed |
    +--------------------------------------+--------------------+-----------+
    | c906ff08-60e6-44e6-8f49-ed56a0853e85 | Do amazing things! |         0 |
    | 2912a79e-8486-4bc3-a4c5-460793a575ab | Be awesome!        |         0 |
    +--------------------------------------+--------------------+-----------+
    ```

    显然，您的表将看起来不同，因为它有您自己的待办事项。但是，你应该看到它们确实存储在哪里了！

<!-- If you take a quick look at the Docker Dashboard, you'll see that we have two app containers running. But, there's
no real indication that they are grouped together in a single app. We'll see how to make that better shortly! -->
如果您快速查看Docker仪表板，您将看到有两个应用程序容器正在运行。但是，没有迹象表明，他们是组合在一起运行的。我们很快就会看到如何让它变得更好！

![Docker Dashboard showing two ungrouped app containers](dashboard-multi-container-app.png)

## 总结

<!-- At this point, we have an application that now stores its data in an external database running in a separate
container. We learned a little bit about container networking and saw how service discovery can be performed
using DNS. -->
此时，我们有了1个应用程序，它把数据存储在另外一个运行的数据库容器中。

然后，我们还学习了一点容器网络知识，并了解了如何使用DNS查看另外一个容器的IP。

<!-- But, there's a good chance you are starting to feel a little overwhelmed with everything you need to do to start up
this application. We have to create a network, start containers, specify all of the environment variables, expose
ports, and more! That's a lot to remember and it's certainly making things harder to pass along to someone else. -->
但是，这里有一个很大的挑战，让你开始觉得有点不知所措：为了启动这个应用我们必须创建一个网络，启动容器，指定所有环境变量，映射端口等等！要记住的东西很多，这无疑会让事情变得复杂，且更难以分享给其他人使用。

<!-- In the next section, we'll talk about Docker Compose. With Docker Compose, we can share our application stacks in a
much easier way and let others spin them up with a single (and simple) command! -->
在下一节中，我们将讨论 `Docker Compose`（译者注：单机环境下容器编排工具）。通过Docker Compose，我们能更容易把我们的应用分享给别人，并且让他们只需要1个（简单的）命令就能启动所有的东西！