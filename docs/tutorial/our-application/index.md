
<!-- For the rest of this tutorial, we will be working with a simple todo
list manager that is running in Node.js. If you're not familiar with Node.js,
don't worry! No real JavaScript experience is needed! -->
在本教程的其余部分，我们将使用在Node.js中运行的简单的待办事项应用。如果您不熟悉Node.js，别担心！不需要真正的JavaScript开发经验！

<!-- At this point, your development team is quite small and you're simply
building an app to prove out your MVP (minimum viable product). You want
to show how it works and what it's capable of doing without needing to
think about how it will work for a large team, multiple developers, etc. -->
您的开发团队相当小，您只是构建一个应用程序来证明您的MVP（最低可行产品）。你想要展示它是如何工作的，以及它不需要做什么，并思考它如何被大型团队、多个开发人员等使用。

![Todo List Manager Screenshot](todo-list-sample.png)

<!-- ## Getting our App -->
## 下载应用

<!-- Before we can run the application, we need to get the application source code onto 
our machine. For real projects, you will typically clone the repo. But, for this tutorial,
we have created a ZIP file containing the application. -->

在我们运行应用程序之前，我们需要将应用程序源代码放在我们的机器上。在实际项目中，您通常使用git clone来获取源码。但是，对于本教程，为了方便你体验，我们则创建了一个包含该应用程序的ZIP文件。

<!-- 1. [Download the ZIP](/assets/app.zip). Open the ZIP file and make sure you extract the contents. -->
1. [下载压缩包](/assets/app.zip)，然后找一个合适的目录解压。
2. 然后使用您最喜欢的代码编辑器打开项目（比如使用Visual Studio Code），你应该会看到一个 `package.json` 文件和2个目录（`src` and `spec`）.
<!-- 1. Once extracted, use your favorite code editor to open the project. If you're in need of
    an editor, you can use [Visual Studio Code](https://code.visualstudio.com/). You should
    see the `package.json` and two subdirectories (`src` and `spec`). -->

![Screenshot of Visual Studio Code opened with the app loaded](ide-screenshot.png){: style="width:650px;margin-top:20px;"}
    {: .text-center }

<!-- ## Building the App's Container Image -->
## 构建容器映像

<!-- In order to build the application, we need to use a `Dockerfile`. A
Dockerfile is simply a text-based script of instructions that is used to
create a container image. If you've created Dockerfiles before, you might see a few flaws in the Dockerfile below. But, don't worry! We'll go over them. -->
为了构建应用程序镜像，我们需要使用`Dockerfile`。Dockerfile只是一个基于文本的指令脚本，用于创建容器镜像。如果您以前创建过Dockerfile，您可能会在下面的Docker文件中看到一些缺陷。但是，别担心！我们后续会介绍的。

<!-- 1. Create a file named `Dockerfile` in the same folder as the file `package.json` with the following contents. -->
1. 在与`package.json`相同的文件夹中创建一个名为`Dockerfile`的文件，其中包含以下内容：

    ```dockerfile
    FROM node:12-alpine
    #译者注，这一句可能会导致报错，可以去掉，没有影响
    #RUN apk add --no-cache python g++ make
    WORKDIR /app
    COPY . .
    RUN yarn install --production
    CMD ["node", "src/index.js"]
    ```

    <!-- Please check that the file `Dockerfile` has no file extension like `.txt`. Some editors may append this file extension automatically and this would result in an error in the next step. -->
    !!! info
        请检查Dockerfile文件是否有像.txt这样的文件扩展名。一些编辑器可能会自动附加此文件扩展名，这会导致下一步执行失败。

2. 请打开终端，并转到Dockerfile所在`APP`目录，然后使用`docker build`命令构建容器映像。
    ```bash
    docker build -t getting-started .
    ```
    <!-- 2. If you haven't already done so, open a terminal and go to the `app` directory with the `Dockerfile`. Now build the container image using the `docker build` command. -->

    <!-- This command used the Dockerfile to build a new container image. You might
    have noticed that a lot of "layers" were downloaded. This is because we instructed
    the builder that we wanted to start from the `node:12-alpine` image. But, since we
    didn't have that on our machine, that image needed to be downloaded. -->
    此命令使用Dockerfile构建新的容器镜像。你可能注意到下载了很多“层”。这是因为我们想从 `node:12-alpine` 开始构建，但本机上不存在，所以我们需要先下载那个镜像。

    <!-- After the image was downloaded, we copied in our application and used `yarn` to 
    install our application's dependencies. The `CMD` directive specifies the default 
    command to run when starting a container from this image. -->
    当下载基础镜像完成后，开始复制我们的应用程序，然后使用 `yarn` 安装我们应用程序的依赖项。`CMD`指令只有在我们启动容器时会被执行，编辑镜像时则不会。

    <!-- Finally, the `-t` flag tags our image. Think of this simply as a human-readable name
    for the final image. Since we named the image `getting-started`, we can refer to that
    image when we run a container. -->
    最后，使用 `-t` 标志给我们的镜像打一个标签，名字可以是任意的，但最好起一个人类可以看懂的名字。由于我们给镜像命名为getting-started，所以，当我们运行容器后能够看到它。

    <!-- The `.` at the end of the `docker build` command tells that Docker should look for the `Dockerfile` in the current directory. -->
    `docker build` 命令末尾的 `.` 告诉Docker应当在当前目录中查找`Dockerfile`。

<!-- ## Starting an App Container -->
## 启动容器

<!-- Now that we have an image, let's run the application! To do so, we will use the `docker run`
command (remember that from earlier?). -->
现在，我们有了一个容器镜像，接下来，让我们来运行一下试试！我们将使用`docker run`命令（还记得吗？）。

1. 使用 `docker run` 命令启动容器，并指定我们刚刚创建的镜像：
    ```bash
    docker run -dp 3000:3000 getting-started
    ```

    还记得 `-d` 和 `-p` 标志吗？我们正在以“分离”模式运行新容器，并在主机的端口3000与容器的端口3000之间创建映射。没有端口映射，我们将无法访问应用程序。

2. 几秒钟后，打开网页浏览器[http://localhost:3000](http://localhost:3000)。您应该会看到我们的应用程序！
    ![Empty Todo List](todo-list-empty.png){: style="width:450px;margin-top:20px;"}
    {: .text-center }

3. 您可以添加一两个代办事项试一试，看看它是否能添加成功，然后试一试把它标记为完成或者删除。如果一切顺利，则证明您的代办事项已经成功存储在后端！相当快和容易，是吧？

<!-- At this point, you should have a running todo list manager with a few items, all built by you!
Now, let's make a few changes and learn about managing our containers. -->
此时，您应该有一个运行的待办事项列表，其中包含一些由您构建的代办事项！

现在，让我们做一些更改，并学习如何管理我们的容器。

<!-- If you take a quick look at the Docker Dashboard, you should see your two containers running now 
(this tutorial and your freshly launched app container)! -->
如果您快速查看Docker仪表板，您应该会看到您的两个容器现在在运行（本教程和您新推出的应用程序容器）！

![Docker Dashboard with tutorial and app containers running](dashboard-two-containers.png)


<!-- ## Recap -->
## 总结

<!-- In this short section, we learned the very basics about building a container image and created a
Dockerfile to do so. Once we built an image, we started the container and saw the running app! -->
在这个简短的部分中，我们学习了构建镜像的基础知识。

并且通过创建Dockerfile构建了我们的应用容器镜像，我们启动了容器，并看到了正在运行的应用程序！

<!-- Next, we're going to make a modification to our app and learn how to update our running application
with a new image. Along the way, we'll learn a few other useful commands. -->
接下来，我们将修改我们的应用程序，并了解如何通过新的镜像更新我们正在运行的容器。在此过程中，我们将学习其他一些有用的命令。
