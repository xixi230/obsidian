
 AstrBot 是一个松耦合、异步、支持多消息平台部署

目标，在了解python的语言特性，以及大模型最基本的实践使用场景


https://docs.astrbot.app/
![[Pasted image 20260201150950.png]]

核心架构：
![[Pasted image 20260201151037.png]]
![[Image_1769932485417.jpg]]


上面的拓扑图我们可以大致分为以下几个层次：
### 消息平台适配层
**入口** 为系统从多个消息平台接收用户输入，包括 QQ、企业微信等
**组件**包含注册器和管理器，负责不同平台的链接和认证

协议使用为webhook和websockets

### 事件流水层
核心：使用 **Event Bus**（事件总线）作为消息枢纽，负责接收、分发和调度所有传入事件。

**事件的注册和调度**

### 大模型适配层
适配大模型

### 插件层
实现复杂能力，支持高度自定义拓展
**内置插件**：包括知识库插件（用于 RAG 检索增强）、长期记忆插件、函数调用工具管理器、代码执行器插件、对话管理器等
**上下文管理**：插件与系统核心通过“上下文”进行交互，执行流水线阶段 1, 2, 3... 的具体任务


### 后端服务与基础设置
**支持服务**：包括配置文件管理器、向量数据库（用于知识库）、软件更新器、对象存储等。
**部署与运维**：系统通过 Docker 部署，并使用 caddy、Grafana（用于指标收集）、Cloudflare R2（对象存储）等工具支持运行和监控。
**管理与访问**：提供一个 WebUI 仪表盘前端页面和 API 后端服务器，用于配置和管理机器人，可通过 HTTP 访问。


## 部署方式（本文档聚焦于windwos系统的powershell）
astrBot文档本事提供多种部署方式
### docker部署
docker的优势是使得不再需要折腾不同平台的环境配置，极大的节约了时间成本
docker：[Docker: Accelerated Container Application Development](https://www.docker.com/)
通过docker compose：
和NapCat(其中一个通讯协议端)一起部署：
```powershell
mkdir astrbot 
cd astrbot 
wget https://raw.githubusercontent.com/NapNeko/NapCat-Docker/main/compose/astrbot.yml -OutFile astrbot.yml -UseBasicParsing
docker compose -f astrbot.yml up -d
```

![[Pasted image 20260201170617.png]]
![[Pasted image 20260201171541.png]]

我们查看拉取的yml可以发现napcat和astrbot开放的端口分别是6099和6185


检查docker
![[Pasted image 20260201171703.png]]
确认docker正在运行，端口正确


于浏览器输入两个服务端的端口进行访问

```
localhost:6185
localhost:6099
```
![[Pasted image 20260201171830.png]]![[Pasted image 20260201171839.png]]


页面如上，其中astrbot给出了默认用户名和密码，我们可以直接进入：
![[Pasted image 20260201171930.png]]
进入后请尽快修改密码和账号，防止被盗

但是我们发现napcat协议端并不知道token，这是因为我们需要通过在docker内进行登录，才能在日志中看到


终端输入：
```powershell
 docker logs napcat
```

输出末尾：
![[Pasted image 20260201172407.png]]

我们可以看到登录扫码请求，我们直接用小号扫描即可。

>Tips：
过期了则重启容器：

```powershell
 docker restart napcat
```

重新查看日志：
```
dokcer logs napcat
```

寻找到token值即可

如果日志过多，可以使用以下命令过滤无关值：
```powershell
 docker logs napcat 2>&1 | Select-String "Token"
```




___


### 创建机器人
选择onebot v11
默认配置
选择启用即可







