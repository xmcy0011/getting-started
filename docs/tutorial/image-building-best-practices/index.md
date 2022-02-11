## 安全扫描（Security Scanning）

<!-- When you have built an image, it is good practice to scan it for security vulnerabilities using the `docker scan` command.
Docker has partnered with [Snyk](http://snyk.io) to provide the vulnerability scanning service. -->
构建镜像后，最好使用`docker scan`命令对其进行安全漏洞扫描。
Docker已与[Snyk](http://snyk.io)合作提供漏洞扫描服务。

<!-- For example, to scan the `getting-started` image you created earlier in the tutorial, you can just type -->
比如，扫描您在教程早期创建的`getting-started`镜像：

```bash
docker scan getting-started
```

<!-- The scan uses a constantly updated database of vulnerabilities, so the output you see will vary as new
vulnerabilities are discovered, but it might look something like this: -->
它的原理是使用一个不断更新的漏洞数据库，因此您看到的输出会随着新的漏洞而变化，可能是这样的：

```plaintext
✗ Low severity vulnerability found in freetype/freetype
  Description: CVE-2020-15999
  Info: https://snyk.io/vuln/SNYK-ALPINE310-FREETYPE-1019641
  Introduced through: freetype/freetype@2.10.0-r0, gd/libgd@2.2.5-r2
  From: freetype/freetype@2.10.0-r0
  From: gd/libgd@2.2.5-r2 > freetype/freetype@2.10.0-r0
  Fixed in: 2.10.0-r1

✗ Medium severity vulnerability found in libxml2/libxml2
  Description: Out-of-bounds Read
  Info: https://snyk.io/vuln/SNYK-ALPINE310-LIBXML2-674791
  Introduced through: libxml2/libxml2@2.9.9-r3, libxslt/libxslt@1.1.33-r3, nginx-module-xslt/nginx-module-xslt@1.17.9-r1
  From: libxml2/libxml2@2.9.9-r3
  From: libxslt/libxslt@1.1.33-r3 > libxml2/libxml2@2.9.9-r3
  From: nginx-module-xslt/nginx-module-xslt@1.17.9-r1 > libxml2/libxml2@2.9.9-r3
  Fixed in: 2.9.9-r4
```

<!-- The output lists the type of vulnerability, a URL to learn more, and importantly which version of the relevant library
fixes the vulnerability. -->
这些输出列出了漏洞的类型、要了解更多信息的URL，以及最重要的是相关库的哪个版本修复了该漏洞。

<!-- There are several other options, which you can read about in the [docker scan documentation](https://docs.docker.com/engine/scan/). -->
您可以通过阅读这个文档 [docker scan documentation](https://docs.docker.com/engine/scan/) 了解更多信息。

<!-- As well as scanning your newly built image on the command line, you can also [configure Docker Hub](https://docs.docker.com/docker-hub/vulnerability-scanning/)
to scan all newly pushed images automatically, and you can then see the results in both Docker Hub and Docker Desktop. -->
除了在命令行上扫描新构建的镜像，还可以 [配置Docker Hub](https://docs.docker.com/docker-hub/vulnerability-scanning/) 自动扫描所有新推送的镜像，然后可以在Docker Hub和Docker Desktop中看到结果。

![Hub vulnerability scanning](hvs.png){: style=width:75% }
{: .text-center }

<!-- ## Image Layering -->
## 镜像分层（Image Layering）

<!-- Did you know that you can look at what makes up an image? Using the `docker image history`
command, you can see the command that was used to create each layer within an image. -->

您知道吗，您可以查看镜像的组成？使用`docker image history`命令，可以看到用于在镜像中创建每个层的命令。

<!-- 1. Use the `docker image history` command to see the layers in the `getting-started` image you
   created earlier in the tutorial. -->

<!-- 1. You'll notice that several of the lines are truncated. If you add the `--no-trunc` flag, you'll get the
   full output (yes... funny how you use a truncated flag to get untruncated output, huh?) -->

1. 使用 `docker image history` 命令查看您在教程早期创建的`getting-started`镜像：
    ```bash
    docker image history getting-started
    ```

    <!-- You should get output that looks something like this (dates/IDs may be different). -->
    您应该会得到下面的输出（dates/IDs 可能不一样）：

    ```plaintext
    IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
    a78a40cbf866        18 seconds ago      /bin/sh -c #(nop)  CMD ["node" "src/index.j…    0B                  
    f1d1808565d6        19 seconds ago      /bin/sh -c yarn install --production            85.4MB              
    a2c054d14948        36 seconds ago      /bin/sh -c #(nop) COPY dir:5dc710ad87c789593…   198kB               
    9577ae713121        37 seconds ago      /bin/sh -c #(nop) WORKDIR /app                  0B                  
    b95baba1cfdb        13 days ago         /bin/sh -c #(nop)  CMD ["node"]                 0B                  
    <missing>           13 days ago         /bin/sh -c #(nop)  ENTRYPOINT ["docker-entry…   0B                  
    <missing>           13 days ago         /bin/sh -c #(nop) COPY file:238737301d473041…   116B                
    <missing>           13 days ago         /bin/sh -c apk add --no-cache --virtual .bui…   5.35MB              
    <missing>           13 days ago         /bin/sh -c #(nop)  ENV YARN_VERSION=1.21.1      0B                  
    <missing>           13 days ago         /bin/sh -c addgroup -g 1000 node     && addu…   74.3MB              
    <missing>           13 days ago         /bin/sh -c #(nop)  ENV NODE_VERSION=12.14.1     0B                  
    <missing>           13 days ago         /bin/sh -c #(nop)  CMD ["/bin/sh"]              0B                  
    <missing>           13 days ago         /bin/sh -c #(nop) ADD file:e69d441d729412d24…   5.59MB   
    ```

    <!-- Each of the lines represents a layer in the image. The display here shows the base at the bottom with
    the newest layer at the top. Using this, you can also quickly see the size of each layer, helping 
    diagnose large images. -->
    每一行代表镜像的一个层，底部是基础层，最新的层在顶部。使用此功能，您还可以快速查看每层的大小，以诊断大型镜像，帮助其瘦身。

1. 您会注意到有几行被截断了。如果添加 `--no trunc` 标志，您将获得完整输出（有趣的是，您如何使用 trunc 标志来获得不受信任的输出？）
    ```bash
    docker image history --no-trunc getting-started
    ```


<!-- ## Layer Caching -->
## 镜像缓存（Layer Caching）

<!-- Now that you've seen the layering in action, there's an important lesson to learn to help decrease build
times for your container images. -->
现在您已经看到了分层的作用，有一个重要的教训要学习，以帮助减少创建容器镜像的时间。

<!-- > Once a layer changes, all downstream layers have to be recreated as well -->
> 一旦某一层改变, 所有下游层也必须重新创建

<!-- Let's look at the Dockerfile we were using one more time... -->
让我们再看一次我们使用的Dockerfile：
```dockerfile
FROM node:12-alpine
WORKDIR /app
COPY . .
RUN yarn install --production
CMD ["node", "src/index.js"]
```

<!-- Going back to the image history output, we see that each command in the Dockerfile becomes a new layer in the image.
You might remember that when we made a change to the image, the yarn dependencies had to be reinstalled. Is there a
way to fix this? It doesn't make much sense to ship around the same dependencies every time we build, right? -->
回到镜像历史（image history）输出，我们看到Dockerfile中的每个命令都成为镜像中的一个新层。您可能还记得，当我们更改镜像时，必须重新安装依赖项。有没有方法能解决这个问题？我们每次构建时都要安装相同的依赖项，这没有多大意义，对吧？

<!-- To fix this, we need to restructure our Dockerfile to help support the caching of the dependencies. For Node-based
applications, those dependencies are defined in the `package.json` file. So, what if we copied only that file in first,
install the dependencies, and _then_ copy in everything else? Then, we only recreate the yarn dependencies if there was
a change to the `package.json`. Make sense? -->
为了解决这个问题，我们需要重新构造Dockerfile，以帮助支持依赖项的缓存。对node.js（Node-based
applications）应用程序而言，这些依赖项定义在 `package.json` 文件中。那么，如果我们先只复制那个文件，安装依赖项，然后再复制其他内容呢？然后，我们仅需要在`package.json`改变时，再重新创建依赖项，有道理吗？

<!-- 1. Update the Dockerfile to copy in the `package.json` first, install dependencies, and then copy everything else in. -->

<!-- 1. Create a file named `.dockerignore` in the same folder as the Dockerfile with the following contents. -->

<!-- 1. Build a new image using `docker build`. -->

<!-- 1. Now, make a change to the `src/static/index.html` file (like change the `<title>` to say "The Awesome Todo App"). -->

<!-- 1. Build the Docker image now using `docker build -t getting-started .` again. This time, your output should look a little different. -->

1. 更新Dockerfile，首先拷贝 `package.json` , 然后安装依赖, 最后拷贝其他的一切：
    ```dockerfile hl_lines="3 4 5"
    FROM node:12-alpine
    WORKDIR /app
    # copy
    COPY package.json yarn.lock ./
    # 安装依赖
    RUN yarn install --production
    # 拷贝余下的
    COPY . .
    CMD ["node", "src/index.js"]
    ```

1. 在和Dockerfile同目录下创建 `.dockerignore` 文件，写入以下内容：

    ```ignore
    node_modules
    ```

    <!-- `.dockerignore` files are an easy way to selectively copy only image relevant files.
    You can read more about this
    [here](https://docs.docker.com/engine/reference/builder/#dockerignore-file).
    In this case, the `node_modules` folder should be omitted in the second `COPY` step because otherwise,
    it would possibly overwrite files which were created by the command in the `RUN` step.
    For further details on why this is recommended for Node.js applications and other best practices,
    have a look at their guide on
    [Dockerizing a Node.js web app](https://nodejs.org/en/docs/guides/nodejs-docker-webapp/). -->
    `.dockerignore`文件是一种选择性复制镜像相关文件的简单方法。您可以阅读更多关于这方面的内容 [这里](https://docs.docker.com/engine/reference/builder/#dockerignore-file)。这样的话，在第二个`copy`命令中不会拷贝 `node_modules` 文件夹，否则，它可能会覆盖 `RUN` 命令创建的文件。有关为什么建议对node.js使用此选项的更多详细信息，请看看他们的指南 [Dockerizing a Node.js web app](https://nodejs.org/en/docs/guides/nodejs-docker-webapp/)。

1. 使用`docker Build`创建新映像：

    ```bash
    docker build -t getting-started .
    ```

    <!-- You should see output like this... -->
    您会看到类似下面的输出：

    ```plaintext
    Sending build context to Docker daemon  219.1kB
    Step 1/6 : FROM node:12-alpine
    ---> b0dc3a5e5e9e
    Step 2/6 : WORKDIR /app
    ---> Using cache
    ---> 9577ae713121
    Step 3/6 : COPY package.json yarn.lock ./
    ---> bd5306f49fc8
    Step 4/6 : RUN yarn install --production
    ---> Running in d53a06c9e4c2
    yarn install v1.17.3
    [1/4] Resolving packages...
    [2/4] Fetching packages...
    info fsevents@1.2.9: The platform "linux" is incompatible with this module.
    info "fsevents@1.2.9" is an optional dependency and failed compatibility check. Excluding it from installation.
    [3/4] Linking dependencies...
    [4/4] Building fresh packages...
    Done in 10.89s.
    Removing intermediate container d53a06c9e4c2
    ---> 4e68fbc2d704
    Step 5/6 : COPY . .
    ---> a239a11f68d8
    Step 6/6 : CMD ["node", "src/index.js"]
    ---> Running in 49999f68df8f
    Removing intermediate container 49999f68df8f
    ---> e709c03bc597
    Successfully built e709c03bc597
    Successfully tagged getting-started:latest
    ```

    <!-- You'll see that all layers were rebuilt. Perfectly fine since we changed the Dockerfile quite a bit. -->
    您将看到所有层都已重建，非常好，因为我们对Dockerfile做了很多修改。

1. 现在，对 `src/static/index.html` 进行更改（比如将`<title>`改为 `很棒的Todo应用`）。

1. 现在使用 `docker build -t getting started` 再创建Docker镜像一次，这次，您的输出看起来应该有点不同。

    ```plaintext hl_lines="5 8 11"
    Sending build context to Docker daemon  219.1kB
    Step 1/6 : FROM node:12-alpine
    ---> b0dc3a5e5e9e
    Step 2/6 : WORKDIR /app
    ---> Using cache
    ---> 9577ae713121
    Step 3/6 : COPY package.json yarn.lock ./
    ---> Using cache
    ---> bd5306f49fc8
    Step 4/6 : RUN yarn install --production
    ---> Using cache
    ---> 4e68fbc2d704
    Step 5/6 : COPY . .
    ---> cccde25a3d9a
    Step 6/6 : CMD ["node", "src/index.js"]
    ---> Running in 2be75662c150
    Removing intermediate container 2be75662c150
    ---> 458e5c6f080c
    Successfully built 458e5c6f080c
    Successfully tagged getting-started:latest
    ```

    <!-- First off, you should notice that the build was MUCH faster! And, you'll see that steps 1-4 all have `Using cache`. So, hooray! We're using the build cache. Pushing and pulling this image and updates to it will be much faster as well. Hooray! -->
    首先，您应该注意到构建速度要快得多！您也会发现步骤1-4都有`使用缓存`。所以，万岁！我们正在使用构建缓存。推拉此镜像并对其进行更新也会快得多，好极了！


<!-- ## Multi-Stage Builds -->
## 多阶段构建（Multi-Stage Builds）

<!-- While we're not going to dive into it too much in this tutorial, multi-stage builds are an incredibly powerful
tool to help use multiple stages to create an image. There are several advantages for them: -->
虽然在本教程中我们不会过多地深入讨论，但多阶段构建是一个非常强大的工具。该工具有助于使用多个阶段创建镜像，它们有几个优点：

<!-- - Separate build-time dependencies from runtime dependencies
- Reduce overall image size by shipping _only_ what your app needs to run -->
- 将构建时依赖项与运行时依赖项分开
- 通过只发送应用程序运行所需的内容来减少整体镜像大小

<!-- ### Maven/Tomcat Example -->
### Maven/Tomcat 示例

<!-- When building Java-based applications, a JDK is needed to compile the source code to Java bytecode. However,
that JDK isn't needed in production. Also, you might be using tools like Maven or Gradle to help build the app.
Those also aren't needed in our final image. Multi-stage builds help. -->
在构建基于Java的应用程序时，需要使用JDK将源代码编译为Java字节码。然而生产中不需要JDK。此外，您可能正在使用Maven或Gradle等工具来帮助构建应用程序。在我们的最终的镜像中也不需要这些。

以下是一个多阶段构建的示例：

```dockerfile
FROM maven AS build
WORKDIR /app
COPY . .
RUN mvn package

FROM tomcat
COPY --from=build /app/target/file.war /usr/local/tomcat/webapps 
```

<!-- In this example, we use one stage (called `build`) to perform the actual Java build using Maven. In the second
stage (starting at `FROM tomcat`), we copy in files from the `build` stage. The final image is only the last stage
being created (which can be overridden using the `--target` flag). -->
在本例中，我们通过第一个阶段（称为`build`）来使用Maven执行实际的Java构建。第二个阶段（从`FROM tomcat`开始），我们从`build`阶段复制文件。最终，生成的镜像只包含第一个阶段（build）创建的内容（也可以使用`--target`标志覆盖）。


<!-- ### React Example -->
### React 示例

<!-- When building React applications, we need a Node environment to compile the JS code (typically JSX), SASS stylesheets,
and more into static HTML, JS, and CSS. If we aren't doing server-side rendering, we don't even need a Node environment
for our production build. Why not ship the static resources in a static nginx container? -->

在构建React应用程序时，我们需要一个Node环境来编译JS代码（通常是JSX）、SASS样式表、还有更多的其他容包如静态HTML、JS和CSS等。如果我们不进行服务端渲染，生产环境中我们甚至不需要Node环境。为什么不将静态资源放在一个静态的nginx容器中？

```dockerfile
FROM node:12 AS build
WORKDIR /app
COPY package* yarn.lock ./
RUN yarn install
COPY public ./public
COPY src ./src
RUN yarn run build

FROM nginx:alpine
COPY --from=build /app/build /usr/share/nginx/html
```

<!-- Here, we are using a `node:12` image to perform the build (maximizing layer caching) and then copying the output
into an nginx container. Cool, huh? -->
在这里，我们使用一个`node:12`基础镜像来执行构建（最大化层缓存），然后复制输出文件放入nginx容器中，酷吧？

## 总结

<!-- By understanding a little bit about how images are structured, we can build images faster and ship fewer changes.
Scanning images gives us confidence that the containers we are running and distributing are secure.
Multi-stage builds also help us reduce overall image size and increase final container security by separating
build-time dependencies from runtime dependencies. -->
通过认识理解镜像的结构，我们可以更快地构建镜像，并进行更少的更改。  

- 扫描镜像让我们确信我们正在运行和分发的容器是安全的。
- 多阶段构建通过分离运行时依赖和编译时依赖，来帮助我们减少整体镜像大小，同时也提高了最终容器的安全性。