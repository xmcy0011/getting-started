
<!-- As a small feature request, we've been asked by the product team to
change the "empty text" when we don't have any todo list items. They
would like to transition it to the following: -->
现在有一个小需求，当我们没有任何待办事项时，产品团队要求我们更改“空文本”为以下内容：

> You have no todo items yet! Add one above!

<!-- Pretty simple, right? Let's make the change. -->
很简单，对吗？那就让我们来做改掉它吧！

<!-- ## Updating our Source Code -->
## 更新我们的源代码

<!-- 1. In the `src/static/js/app.js` file, update line 56 to use the new empty text. -->
1. 在 `src/static/js/app.js` 文件中，更新第56行以使用新的文本。
    ```diff
    - <p className="text-center">No items yet! Add one above!</p>
    + <p className="text-center">You have no todo items yet! Add one above!</p>
    ```
1. 让我们使用之前的命令构建新镜像。
    ```bash
    docker build -t getting-started .
    ```
1. 然后，启动更改过代码的新的容器镜像。

    ```bash
    docker run -dp 3000:3000 getting-started
    ```

<!-- **Uh oh!** You probably saw an error like this (the IDs will be different): -->
**Oh!** 您可能看到这样的错误（ID会不同）：

```bash
docker: Error response from daemon: 
driver failed programming external connectivity on endpoint laughing_burnell(bb242b2ca4d67eba76e79474fb36bb5125708ebdabd7f45c8eaf16caaabde9dd): 
Bind for 0.0.0.0:3000 failed: port is already allocated.
```

<!-- So, what happened? We aren't able to start the new container because our old container is still
running. The reason this is a problem is because that container is using the host's port 3000 and
only one process on the machine (containers included) can listen to a specific port. To fix this, 
we need to remove the old container. -->
那么，发生了什么事？我们无法启动新容器，因为我们的旧容器仍然在运行。

这是个问题的原因是，该容器正在使用主机的端口3000，但是机器上同时只能有一个进程（包括容器）可以监听特定端口。

要解决这个问题，我们需要拆除旧的容器。

<!-- ## Replacing our Old Container -->
## 拆除旧容器

<!-- To remove a container, it first needs to be stopped. Once it has stopped, it can be removed. We have two
ways that we can remove the old container. Feel free to choose the path that you're most comfortable with. -->

要拆卸旧容器，首先需要停止它。一旦它停止，它就可以被移除。

我们有两个可以移除旧容器的方法，请随意选择您最喜欢的。

<!-- ### Removing a container using the CLI -->
## 使用CLI删除容器

<!-- 1. Get the ID of the container by using the `docker ps` command. -->
1. 使用 `docker ps` 命令获取旧容器的ID。

    ```bash
    docker ps
    ```

2. 然后使用`docker stop`命令停止容器。

    ```bash
    # 用 docker ps 获取到的ID替换 <the-container-id> 
    docker stop <the-container-id>
    ```

3. 当容器停止后，您就可以使用docker rm命令将其删除。

    ```bash
    docker rm <the-container-id>
    ```

!!! info "提示"
    <!-- You can stop and remove a container in a single command by adding the "force" flag
    to the `docker rm` command. For example: `docker rm -f <the-container-id>` -->
    您可以通过添加“强制”标志来停止和删除单个命令中的容器去 `docker rm` 命令。例如：`docker rm -f <the-container-id>`

<!-- ### Removing a container using the Docker Dashboard -->
## 使用Docker仪表板删除容器

<!-- If you open the Docker dashboard, you can remove a container with two clicks! It's certainly
much easier than having to look up the container ID and remove it. -->
如果您打开Docker仪表板，只需单击两下即可删除容器！这要比查找容器ID并删除它容易得多。

1. 打开仪表板后，将鼠标悬停在应用程序容器上，您将看到一系列操作按钮出现在右侧。

2. 单击垃圾桶图标删除容器。

3. 确认移除，您就完成了！

![Docker Dashboard - removing a container](dashboard-removing-container.png)


<!-- ### Starting our updated app container -->
## 启动更新的应用程序容器

1. 现在，就可以成功启动我们更新过的容器了。

    ```bash
    docker run -dp 3000:3000 getting-started
    ```

1. 刷新浏览器或者重新输入地址：[http://localhost:3000](http://localhost:3000)，你就能看到最新的你改过的文本啦！

![Updated application with updated empty text](todo-list-updated-empty-text.png){: style="width:55%" }
{: .text-center }

<!-- ## Recap -->
## 总结

当我们编译镜像并且更新过后，我们观察到：

- 我们待办事项列表中的所有项目都已消失！这不是一个很好的应用程序！我们将很快讨论这个问题。
- 这么小的变化涉及到很多步骤。

在接下来的一节中，我们将讨论如何查看代码更新，而无需在每次进行更改时重建和启动新容器。

在讨论持久性之前，我们将快速了解如何与其他人共享这些镜像。