### 什么是Docker？
---
Docker is an open source platform that uses **containerization technology.** It allows developers to package applications and their dependencies into **light weight, portable containers.**

Docker containers virtualize the operating system. Each individual container **contains only the application and its libraries and dependecies.**

- Docker Engine
> Docker Engine is the core software that's responsible for managing the lifecycle of Docker containers.  creating, running and orchestrating the containers.
>
> Doeker Engine interacts with the host kernel to allocate resources and enforce isolation between containers.
> cgroup   namespace

- Docker images
> Docker images are lightweight, standalone and executable packages that include everything you need to run a piece of software.

- Docker file
> creating images

容器的应用进程直接运行于宿主机的内核，**容器内没有自己的内核，而且也没有进行硬件虚拟**。因此容器要比传统虚拟机更为轻便。 每个容器之间互相隔离，每个容器有自己的文件系统 ，容器之间进程不会相互影响，能区分计算资源。**容器与运行时环境共享相同的操作系统内核和软件包应用程序**，因此整个容器能够在各种开发、测试和生产配置中移动、打开和使用。

本质上，它是一个特殊的进程。通过名称空间（Namespace）、控制组（Control groups）、切根（chroot）技术把资源、文件、设备、状态和配置划分到一个独立的空间。

Docker 可以以两种形式运行在 Windows 上：

以 Hyper-V 虚拟机的形式运行 Linux 格式的容器，或者运行原生的 Windows 容器。

其中前者运行 Linux 格式的应用程序，后者能运行 Windows 应用程序。

[在 Windows 上可以用 Docker 吗？ - 知乎](https://zhuanlan.zhihu.com/p/47007846)

[docker - Can Windows containers be hosted on Linux? - Stack Overflow](https://stackoverflow.com/questions/42158596/can-windows-containers-be-hosted-on-linux)

[Running Docker Windows and Linux Containers Simultaneously - Developer Support](https://devblogs.microsoft.com/premier-developer/running-docker-windows-and-linux-containers-simultaneously/)

### Running Docker Windows and Linux Containers Simultaneously
---
Starting with Docker for Windows version 18.03.0-ce-win59 the Linux Containers on Windows(LCOW) is available as an experimental feature. 

Docker for Windows currently allows you to switch between running Windows or Linux Containers but not both. Linux Containers were hosted in a Linux Virtual Machine **makeing it convenient for testing purposes but not production.**

LCOW will make it possible to have an application that mixes Linux and Windows containers together on a single host.

#### Get started 
With Docker for Windows started and Windows containers selected, you can now run either Windows or Linux Containers simultaneously. The new _–platform=linux_ command line switch is used to pull or start Linux images on Windows.

```
docker pull --platform=linux ubuntu
```


Now start the Linux container and a Windows Server Core container.
```
docker run --platform=linux -d ubuntu /bin/sh -c "while true; do echo hello world; sleep 1; done"
docker run -d microsoft/windowsservercore ping -t 127.0.0.1
```

Both containers are running on a single host.

👉Doesn’t Docker for Windows already run Linux containers? 
>That’s right. Docker for Windows can run Linux or Windows containers, with support for Linux containers via a Hyper-V Moby Linux VM (as of Docker for Windows 17.10 this VM is based on LinuxKit).

>**The setup for running Linux containers with LCOW is a lot simpler than the previous architecture** where a Hyper-V Linux VM runs a Linux Docker daemon, along with all your containers. With LCOW, the Docker daemon runs as a Windows process (same as when running Docker Windows containers), and every time you start a Linux container Docker launches a **minimal Hyper-V hypervisor** running a VM with a Linux kernel, runc and the container processes running on top.

>Because there’s only one Docker daemon, and because that daemon now runs on Windows, **it will soon be possible to run Windows and Linux Docker containers side-by-side, in the same networking namespace**. This will unlock a lot of exciting development and production scenarios for Docker users on Windows.
