<!-- 
Now that we've built an image, let's share it! To share Docker images, you have to use a Docker
registry. The default registry is Docker Hub and is where all of the images we've used have come from. -->
既然我们已经建立了一个镜像，让我们分享它吧！要共享Docker映像，您必须使用Docker
注册。默认的注册中心是Docker Hub，是我们使用的所有镜像的来源。

<!-- ## Create a Repo -->
## 创建仓库

<!-- To push an image, we first need to create a repo on Docker Hub. -->
要推送镜像，我们首先需要在Docker Hub上创建一个仓库。

1. 如果您需要，请前往 [Docker Hub](https://hub.docker.com) 并登录。

2. 单击 **创建仓库** 按钮。

3. 输入仓库名称, 可以使用 `getting-started`，然后确保其设置为 `Public`。

4. 单击 **创建** 按钮!

<!-- If you look on the right-side of the page, you'll see a section named **Docker commands**. This gives
an example command that you will need to run to push to this repo. -->
如果您查看页面右侧，您将看到一个名为`Docker commands`的部分，这给了您一个可以推送镜像到此仓库的命令。

![Docker command with push example](push-command.png){: style=width:75% }
{: .text-center }

<!-- ## Pushing our Image -->
## 推送镜像

<!-- 1. In the command line, try running the push command you see on Docker Hub. Note that your command
   will be using your namespace, not "docker". -->
1. 在命令行中，尝试运行您在Docker Hub上看到的推送命令。请注意，下面命令中的“docker”需要替换为您自己的账户名。
    ```bash
    $ docker push docker/getting-started
    The push refers to repository [docker.io/docker/getting-started]
    An image does not exist locally with the tag: docker/getting-started
    ```

    为什么失败了？推送命令在本地查找一个名为 `docker/geting-started` 的镜像，但没有成功。如果你运行 `docker image ls`，你会发现确实找不到一个同名的镜像。要解决这个问题，我们需要“标记”现有的镜像，给它起另一个别名。

2. 首先，使用`docker login -u YOUR-USER-NAME`登录docker hub。

3. 然后使用 `docker tag` 命令，给 `getting-started` 镜像起一个新名字。记得把下面的`YOUR-USER-NAME`替换成你自己的Docker ID.

    ```bash
    docker tag getting-started YOUR-USER-NAME/getting-started
    ```

4. 现在再次尝试您的推送命令。如果您的命令是从Docker Hub中复制的，您可以删除tagname部分，因为我们没有在镜像名称中添加标签。如果您没有指定标签，Docker将使用一个名为latest的标签。
    ```bash
    docker push YOUR-USER-NAME/getting-started
    ```

<!-- ## Running our Image on a New Instance -->
## 在新实例上运行我们的镜像

<!-- Now that our image has been built and pushed into a registry, let's try running our app on a brand
new instance that has never seen this container image! To do this, we will use Play with Docker. -->
现在我们的镜像已经构建并成功推送docker hub仓库中，让我们尝试在一个从没有拉取过该镜像的新实例中运行该容器！为此，我们将使用 `Play with Docker`。

1. 打开浏览器并访问 [Play with Docker](http://play-with-docker.com)。

2. 登录您的Docker Hub账号。

3. 登录后，单击左侧栏中的“+ ADD NEW INSTANCE”链接。（如果您没有看到它，请让您的浏览器更宽一点。）几秒钟后，浏览器中将打开一个终端窗口。

    ![Play with Docker add new instance](pwd-add-new-instance.png){: style=width:75% }
{: .text-center }

4. 在终端中，启动您新推送的应用程序容器。

    ```bash
    docker run -dp 3000:3000 YOUR-USER-NAME/getting-started
    ```

    你应该看到镜像被拉下来并最终启动！

5. 启动后，单击`3000`角标，然后您应该就会看到您修改过后的应用程序。如果没有显示3000徽章，您可以单击“打开端口”按钮并输入3000。

## 总结

<!-- In this section, we learned how to share our images by pushing them to a registry. We then went to a
brand new instance and were able to run the freshly pushed image. This is quite common in CI pipelines,
where the pipeline will create the image and push it to a registry and then the production environment
can use the latest version of the image. -->
在本节中，我们通过将镜像推送到docker hub仓库来学习如何共享镜像。然后我们启动了一家全新的实例，并能够运行我们新推送的镜像。这在CI/CD中相当常见，其会将创建的镜像推送到仓库，然后在生产环境中就可以部署使用最新版本的镜像了。

<!-- Now that we have that figured out, let's circle back around to what we noticed at the end of the last
section. As a reminder, we noticed that when we restarted the app, we lost all of our todo list items.
That's obviously not a great user experience, so let's learn how we can persist the data across 
restarts! -->
既然我们已经弄清楚了如何共享镜像，那就让我们回到上次结束时另外一个我们观察到的问题：当我们重新启动应用程序时，我们丢失了所有待办事项列表。这显然不是一个好的用户体验，所以让我们学习如何将数据持久化，即使重启也不丢失！