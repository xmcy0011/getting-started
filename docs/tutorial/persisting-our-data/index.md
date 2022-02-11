
<!-- In case you didn't notice, our todo list is being wiped clean every single time
we launch the container. Why is this? Let's dive into how the container is working. -->
你是否注意到，当我们启动容器时，我们的待办事项列表每次都会被清除干净。

为什么会这样？让我们深入了解容器的工作原理

<!-- ## The Container's Filesystem -->
## 容器的文件系统

<!-- When a container runs, it uses the various layers from an image for its filesystem.
Each container also gets its own "scratch space" to create/update/remove files. Any
changes won't be seen in another container, _even if_ they are using the same image. -->

当容器运行时，它会将镜像中的各个图层用于其文件系统。每个容器还将获得自己的“暂存空间”来创建/更新/删除文件。所以，即使使用相同的镜像，也不会在另一个容器中看到任何更改。

<!-- ### Seeing this in Practice -->
### 实践

<!-- To see this in action, we're going to start two containers and create a file in each.
What you'll see is that the files created in one container aren't available in another. -->
为了确定这一点，我们将启动两个容器，并在每个容器中创建一个文件。

您将看到，在一个容器中创建的文件在另一个容器中不可用。

1. 启动一个 `ubuntu` 容器，该容器将创建一个具有随机数（1到10000之间）的名为 `/data.txt` 的文件。

    ```bash
    docker run -d ubuntu bash -c "shuf -i 1-10000 -n 1 -o /data.txt && tail -f /dev/null"
    ```

    <!-- In case you're curious about the command, we're starting a bash shell and invoking two
    commands (why we have the `&&`). The first portion picks a single random number and writes
    it to `/data.txt`. The second command is simply watching a file to keep the container running. -->
    让我们来解释一下这些命令。我们将启动一个 `bash shell` 并调用两个命令（这就是为什么时候`&&`）。第一部分选择一个随机数字并把它写到 `/data.txt`文件中。第二部分只是为了保持容器一直处于运行状态（否则执行完毕后，立马就会停止）。

2. 我们可以通过命令 `exec` 进入到容器中来查看输出。为此，请打开仪表板，然后单击ubuntu镜像容器的第一个按钮。

    ![Dashboard open CLI into ubuntu container](dashboard-open-cli-ubuntu.png){: style=width:75% }
{: .text-center }

    您将看到一个在ubuntu容器中运行的终端。运行以下命令以查看/data.txt文件的内容。之后关闭此终端。

    ```bash
    cat /data.txt
    ```

    如果您更喜欢命令行，您可以使用 `docker exec` 命令来做同样的事情。你需要得到容器的ID（使用 `docker ps` 获取它），并使用以下命令获取内容。

    ```bash
    docker exec <container-id> cat /data.txt
    ```

    你应该会看到一个随机的数字！

3. 现在，让我们开始另一个 `ubuntu` 容器（相同的镜像），我们将看到该容器中没有此文件。

    ```bash
    docker run -it ubuntu ls /
    ```

    看看！那里没有 `data.txt` 文件！那是因为只有第一个容器把它写在了暂存空间中。

4. 继续使用 `docker rm -f` 命令删除第一个容器。

<!-- ## Container Volumes -->
## 容器卷（Container Volumes）

<!-- With the previous experiment, we saw that each container starts from the image definition each time it starts. 
While containers can create, update, and delete files, those changes are lost when the container is removed 
and all changes are isolated to that container. With volumes, we can change all of this. -->
通过之前的实验，我们看到每个容器每次启动时都从头开始。

虽然容器可以创建、更新和删除文件，但这些更改在删除容器时丢失，并且所有的更改都被隔离到该容器中。通过 `卷（Volumes）`，我们可以改变这一切。

<!-- 卷（[Volumes](https://docs.docker.com/storage/volumes/) ）provide the ability to connect specific filesystem paths of 
the container back to the host machine. If a directory in the container is mounted, changes in that
directory are also seen on the host machine. If we mount that same directory across container restarts, we'd see
the same files. -->
[卷](https://docs.docker.com/storage/volumes/) 为容器提供了连接宿主机特定文件系统路径的能力。如果容器已装载宿主机的目录，则在容器中的对该目录的更改，在宿主机上也能看到。如果我们重新启动一个容器，装载同一目录，我们将看到相同的文件。

<!-- There are two main types of volumes. We will eventually use both, but we will start with **named volumes**. -->
卷主要有两种类型。我们最终将同时使用这两个，但我们将从命名卷（`named volumes`）开始。

> 译者注：另外一种就是绑定挂载，也就是下一节的内容。

<!-- ## Persisting our Todo Data -->
## 持久化我们Todo应用的数据

<!-- By default, the todo app stores its data in a [SQLite Database](https://www.sqlite.org/index.html) at
`/etc/todos/todo.db`. If you're not familiar with SQLite, no worries! It's simply a relational database in 
which all of the data is stored in a single file. While this isn't the best for large-scale applications,
it works for small demos. We'll talk about switching this to a different database engine later. -->
默认情况下，todo应用程序将其数据存储在 [SQLite数据库](https://www.sqlite.org/index.html) /etc/todos/todo.db 的路径中。如果您不熟悉SQLite，别担心！它只是一个关系数据库。所有数据都存储在一个文件中。虽然这不是大型应用程序的最佳选择，但它适用于小型Demo应用。我们稍后将讨论将其切换到其他数据库引擎。

<!-- With the database being a single file, if we can persist that file on the host and make it available to the
next container, it should be able to pick up where the last one left off. By creating a volume and attaching
(often called "mounting") it to the directory the data is stored in, we can persist the data. As our container 
writes to the `todo.db` file, it will be persisted to the host in the volume. -->
由于数据库是一个文件，如果我们可以在主机上保留该文件，并将其提供给下一个容器，它应该能够从最后一个容器中断的地方恢复。通过创建卷并挂载到数据存储目录中，我们就可以持久化存储数据了。当我们的容器把数据写入到todo.db文件中时，会同时将它持久化到卷对应的主机中。

<!-- As mentioned, we are going to use a **named volume**. Think of a named volume as simply a bucket of data. 
Docker maintains the physical location on the disk and you only need to remember the name of the volume. 
Every time you use the volume, Docker will make sure the correct data is provided. -->
如前所述，我们将使用 `命名卷`。命名卷可以简单地视为一个数据桶（PS：或者一个`数据容器`）。Docker维护磁盘上的物理位置，您只需要记住卷的名称。每次您使用卷时，Docker都会确保提供正确的数据。

1. 使用 `docker volume create` 命令创建 `卷`。

    ```bash
    docker volume create todo-db
    ```

2. 在仪表板中再次停止todo应用程序容器（或使用 `docker rm -f <id>` ），因为它仍在运行，就无法使用数据卷。

3. 启动todo应用程序容器，但添加 `-v` 标志以指定挂载卷。我们将使用命名卷并挂载它到 `/etc/todos` ，它将捕获在该路径上创建的所有文件。
    ```bash
    docker run -dp 3000:3000 -v todo-db:/etc/todos getting-started
    ```

4. 容器启动后，打开应用程序，并将一些项目添加到您的待办事项列表中。

    ![Items added to todo list](items-added.png){: style="width: 55%; " }
    {: .text-center }

5. 删除容器。使用仪表板或 `docker ps` 获取ID，然后 `docker rm -f <id>` 将其强制删除。

6. 使用上面的相同命令启动新容器。

7. 打开应用程序。您应该会看到您的项目仍在列表中！

8. 实践完毕，请删除容器。

万岁！您现在已经学会了如何持久化数据！

!!! info "提示"
    <!-- While named volumes and bind mounts (which we'll talk about in a minute) are the two main types of volumes supported
    by a default Docker engine installation, there are many volume driver plugins available to support NFS, SFTP, NetApp, 
    and more! This will be especially important once you start running containers on multiple hosts in a clustered
    environment with Swarm, Kubernetes, etc. -->
    命名卷和绑定挂载（我们将在一分钟内讨论）是支持的两种主要卷类型，如果使用默认的选项安装Docker引擎的话。有许多卷驱动程序插件可用于支持NFS、SFTP、NetApp等等！一旦您开始在集群中的多个主机上运行容器，比如Swarm、Kubernetes等会显得尤其重要。

<!-- ## Diving into our Volume -->
## 进入卷

<!-- A lot of people frequently ask "Where is Docker _actually_ storing my data when I use a named volume?" If you want to know, 
you can use the `docker volume inspect` command. -->
很多人经常问“当我使用命名卷时，Docker实际上在哪里存储我的数据？”如果你想知道，您可以使用 `docker volume inspect` 命令。

```bash
docker volume inspect todo-db
[
    {
        "CreatedAt": "2019-09-26T02:18:36Z",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/todo-db/_data",
        "Name": "todo-db",
        "Options": {},
        "Scope": "local"
    }
]
```

<!-- The `Mountpoint` is the actual location on the disk where the data is stored. Note that on most machines, you will
need to have root access to access this directory from the host. But, that's where it is! -->
Mountpoint是存储数据磁盘上的实际位置。请注意，在大多数机器上，您将需要root访问权限才能从主机访问此目录。

!!! info "直接在 Docker Desktop 上访问卷数据"
    <!-- While running in Docker Desktop, the Docker commands are actually running inside a small VM on your machine.
    If you wanted to look at the actual contents of the Mountpoint directory, you would need to first get inside
    of the VM. -->
    在Docker Desktop中运行时，Docker命令实际上是在计算机上的一个小虚拟机中运行的。如果您想查看Mountpoint目录的实际内容，您需要首先进入虚拟机。

## 总结

<!-- At this point, we have a functioning application that can survive restarts! We can show it off to our investors and
hope they can catch our vision! -->
此时，我们有一个功能正常的应用程序，可以在重新启动后继续运行！我们可以向投资者炫耀它，并且希望他们能抓住我们的愿景！

<!-- However, we saw earlier that rebuilding images for every change takes quite a bit of time. There's got to be a better
way to make changes, right? With bind mounts (which we hinted at earlier), there is a better way! Let's take a look at that now! -->
然而，在早些时候，我们发现每个变化重建镜像都需要相当长的时间。一定有更好的方法，对吗？那就是使用绑定挂载（我们之前暗示过），接下来，让我们一起看看什么是绑定挂载，以及如何使用。