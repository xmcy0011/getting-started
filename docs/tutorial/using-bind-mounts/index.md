
<!-- In the previous chapter, we talked about and used a **named volume** to persist the data in our database.
Named volumes are great if we simply want to store data, as we don't have to worry about _where_ the data
is stored. -->
在上一章中，我们讨论并使用 `命名卷（named volume）` 来持久化数据库中的数据。如果我们只想存储数据，命名卷就很棒，因为我们不必担心数据在哪里存储。

<!-- With **bind mounts**, we control the exact mountpoint on the host. We can use this to persist data, but is often
used to provide additional data into containers. When working on an application, we can use a bind mount to
mount our source code into the container to let it see code changes, respond, and let us see the changes right
away. -->

除此之外，我们还可以使用 `绑定挂载（bind mounts）` 来存储数据，和命名卷（PS：译者更喜欢称其为数据卷或者数据卷容器）不同的是：
- 您可以控制主机上的确切挂载点，将容器的数据存储在宿主机何处将由您来决定。
- 用它来向容器提供额外的数据。比如我们可以使用绑定挂载来将我们的源代码装载到容器中，让它看到代码更改（这一特性对于前端应用特别有帮助）或者加载宿主机的配置文件等等。

<!-- For Node-based applications, [nodemon](https://npmjs.com/package/nodemon) is a great tool to watch for file
changes and then restart the application. There are equivalent tools in most other languages and frameworks. -->
对于基于NodeJs的应用程序来说，`nodemon` 是监视文件的绝佳工具，一旦其发现文件改动，就会自动重新启动应用程序。大多数其他语言和框架都有同等的工具。

<!-- ## Quick Volume Type Comparisons -->
## 快速比较2中卷类型

<!-- Bind mounts and named volumes are the two main types of volumes that come with the Docker engine. However, additional
volume drivers are available to support other use cases ([SFTP](https://github.com/vieux/docker-volume-sshfs), [Ceph](https://ceph.com/geen-categorie/getting-started-with-the-docker-rbd-volume-plugin/), [NetApp](https://netappdvp.readthedocs.io/en/stable/), [S3](https://github.com/elementar/docker-s3-volume), and more). -->
绑定挂载和命名卷是Docker引擎默认的两种主要卷类型。然而，额外的
卷驱动程序可用于支持其他用例（[SFTP](https://github.com/vieux/docker-volume-sshfs)、[Ceph](https://ceph.com/geen-categorie/getting-started-with-the-docker-rbd-volume-plugin/)、[NetApp](https://netappdvp.readthedocs.io/en/stable/)、[S3](https://github.com/elementar/docker-s3-volume), and more)等）。

<!-- |   | Named Volumes | Bind Mounts |
| - | ------------- | ----------- |
| Host Location | Docker chooses | You control |
| Mount Example (using `-v`) | my-volume:/usr/local/data | /path/to/data:/usr/local/data |
| Populates new volume with container contents | Yes | No |
| Supports Volume Drivers | Yes | No | -->
|	| 命名卷 | 绑定挂载 |
|--|--|--|
| 主机位置 | Docker选择 | 你控制 |
| 挂载示例（使用`-v`）| my-volume:/usr/local/data| /your/host/path:/usr/local/data |
| 用容器内容填充新卷 | 是 | 否 |
| 支持卷驱动程序 | 是 | 否 |

<!-- ## Starting a Dev-Mode Container -->
## 启动开发模式的容器

<!-- To run our container to support a development workflow, we will do the following: -->
要运行我们的容器以支持开发工作流程，我们将执行以下操作：
<!-- - Mount our source code into the container
- Install all dependencies, including the "dev" dependencies
- Start nodemon to watch for filesystem changes -->
- 将我们的源代码装入容器
- 安装所有依赖项，包括“开发”依赖项
- 启动 `nodemon` 以监视文件系统更改

<!-- So, let's do it! -->
所以，我们开始吧！

<!-- 1. Make sure you don't have any previous `getting-started` containers running.
2. Run the following command. We'll explain what's going on afterwards: -->
1. 确保您之前没有任何 `getting-started` 容器在运行。
2. 运行以下命令。我们将稍后解释发生了什么：
    ```bash
    docker run -dp 3000:3000 \
        -w /app -v "$(pwd):/app" \
        node:12-alpine \
        sh -c "yarn install && yarn run dev"
    ```

    如果您使用的是PowerShell，请使用此命令。

    ```powershell
    docker run -dp 3000:3000 `
        -w /app -v "$(pwd):/app" `
        node:12-alpine `
        sh -c "yarn install && yarn run dev"
    ```

    - `-dp 3000:3000`：和以前一样。后台模式运行容器，并创建端口映射。
    - `-w /app`：设置命令的“工作目录”。
    - `-v "$(pwd):/app"`：使用绑定挂载将当前主机目录绑定到容器中的/app目录。
    - `node:12-alpine`：要使用的镜像。请注意，这是从我们应用程序Dockerfile中指定的基础镜像。
    - `sh -c "yarn install && yarn run dev"`：我们使用sh命令启动一个shell（带alpine名字的镜像没有bash工具），然后运行yarn install安装所有依赖项，最后运行yarn run dev启动Node应用程序。如果我们查看package.json，我们将看到dev脚本正在启动nodemon。

1. 当使用 `docker logs -f <container-id>` 查看到实时日志后，你就知道你成功启动了应用。

    ```bash
    docker logs -f <container-id>
    $ nodemon src/index.js
    [nodemon] 1.19.2
    [nodemon] to restart at any time, enter `rs`
    [nodemon] watching dir(s): *.*
    [nodemon] starting `node src/index.js`
    Using sqlite database at /etc/todos/todo.db
    Listening on port 3000
    ```

    我们可以通过同时按住 `Ctrl`+`C` 退出监听实时日志。

1. 现在，让我们更改一下应用程序。在 `src/static/js/app.js` 文件的 `109行`，让我们将“添加项目”按钮更改为"添加"。
    ```diff
    - {submitting ? 'Adding...' : 'Add Item'}
    + {submitting ? 'Adding...' : 'Add'}
    ```

1. 只需刷新页面（或打开页面），您几乎可以立即在浏览器中看到更改效果。当然，有可能Node服务器需要几秒钟重新启动，因此如果发生错误，您只需在几秒钟后尝试刷新即可。

    ![Screenshot of updated label for Add button](updated-add-button.png){: style="width:75%;"}
    {: .text-center }

1. 请随时进行任何其他您想要的更改。完成后，停止容器，并通过命令 `docker build -t getting-started .` 构建新镜像。

<!-- Using bind mounts is _very_ common for local development setups. The advantage is that the dev machine doesn't need to have
all of the build tools and environments installed. With a single `docker run` command, the dev environment is pulled and ready
to go. We'll talk about Docker Compose in a future step, as this will help simplify our commands (we're already getting a lot of flags). -->
使用绑定挂载一个非常常见的场景是：`充当本地开发的编译环境`。开发机不需要安装所有构建工具和环境，只需要使用单个`docker run`命令，开发环境就能一键就绪。

我们将在未来进一步讨论 `Docker Compose`，因为这将有助于简化我们的命令（我们已经接触了很多命令参数）。

## 总结

<!-- At this point, we can persist our database and respond rapidly to the needs and demands of our investors and founders. Hooray!
But, guess what? We received great news! -->
此时，我们可以持久化存储我们的数据库，并快速响应投资者和创始人的需求和要求。万岁！
但是，你猜怎么着？我们收到了好消息！

<!-- **Your project has been selected for future development!**  -->
**未来的规划中您的项目已被选中！**

<!-- In order to prepare for production, we need to migrate our database from working in SQLite to something that can scale a
little better. For simplicity, we'll keep with a relational database and switch our application to use MySQL. But, how 
should we run MySQL? How do we allow the containers to talk to each other? We'll talk about that next! -->
为了准备投入生产，我们需要将数据库从在SQLite中迁移，使用扩展性好一点的数据库。为了简单起见，我们将继续使用关系型数据库，从SQLite切换到使用MySQL。但是，我们应该如何运行MySQL？我们如何允许容器相互访问？接下来，让我们一起看看！