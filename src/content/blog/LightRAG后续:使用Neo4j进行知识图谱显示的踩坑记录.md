---
title: LightRAG后续:使用Neo4j进行知识图谱显示的踩坑记录
description: 使用LightRAG完成构建之后，可以结合Neo4j工具实现可视化，但是需要在本机上部署Neo4j的环境和软件，但第一次接触Neo4j的我参考现有帖子去做，总是碰壁。本篇把踩的坑记录下来，帮助后来的小白。
pubDatetime: 2024-11-23
tags:
  - LLM
  - 知识图谱
  - 技术
share: "true"
---
> 注意: 这里主要涉及JDK, Neo4j和APOC，它们之间都有版本对应关系！第一次接触的话，很多设置细节问题需要注意，我搞了好久才弄好，记录下来帮人避坑。
## 0. 版本匹配提醒
- 依赖关系：JDK <- Neo4j <- APOC
- 简单理解：JDK版本决定Neo4j的版本，而APOC作为Neo4j的插件，又要去适配Neo4j的版本。可以根据情况自行选择和匹配。
- 我的选择：
  · JDK：11
  · Neo4j：4.4.0.x (我选的是4.4.0.25)
  · APOC：4.4.0
## 1.安装 JDK
### 1.1 下载地址
> 我用的是4.4.0.x版本的 `Neo4j`，因此需要下载`JDK11`，其它的各位可以自己确定一下。反正我是换了`Neo4j`之后运行不了，错误提示里面告诉我需要JDK11的，大不了看错误提示改一下。
1. [JDK11官方下载](https://www.oracle.com/cn/java/technologies/javase/jdk11-archive-downloads.html) 建议使用官网下载，但是需要注册和邮箱激活！
2. [OpenJDK Downloads | Download Java JDK 8, 11, 17, & 21 | OpenLogic](https://www.openlogic.com/openjdk-downloads?field_java_parent_version_target_id=406&field_operating_system_target_id=436&field_architecture_target_id=391&field_java_package_target_id=396) 这个没有测试过，可能是开源版本的JDK？据说可以平替（自行验证，我不负责🤪）
### 1.2 安装设置
正常点击安装完成后，需要设置一下环境变量。在“系统变量”中添加：
- 变量名(N):`JAVA_HOME`
- 变量值(V):`C:\Program Files\Java\jdk-11`
当然，变量值那里要换成自己电脑上`JDK`的安装位置！
![ 2024-11-22 220708.png](https://s2.loli.net/2024/11/23/h3sEy85YlMdfwLJ.png)
## 2. 安装 NEO4J
> 因为大部分资料是基于Community版本的Neo4j，因此这里也选择Community，但是桌面版（Desktop）也可以共存，感兴趣的话可以自行研究一下。
### 2.1 下载地址
[Index of /doc/neo4j/](https://we-yun.com/doc/neo4j/)
自行选择版本下载即可，但是再次提醒，它和`APOC`版本之间是有对应的，一定要选择匹配的版本，省的像我一样返工！
我选择了`4.4.0`版。（下表是`NEO4J`和`APOC`的版本兼容表，来源于`APOC`官方页面）

| apoc version                                                                         | neo4j version  |
| ------------------------------------------------------------------------------------ | -------------- |
| [4.4.0.1](https://github.com/neo4j-contrib/neo4j-apoc-procedures/releases/4.4.0.1)   | 4.4.0 (4.3.x)  |
| [4.3.0.4](https://github.com/neo4j-contrib/neo4j-apoc-procedures/releases/4.3.0.4)   | 4.3.7 (4.3.x)  |
| [4.2.0.9](https://github.com/neo4j-contrib/neo4j-apoc-procedures/releases/4.2.0.9)   | 4.2.11 (4.2.x) |
| [4.1.0.10](https://github.com/neo4j-contrib/neo4j-apoc-procedures/releases/4.1.0.10) | 4.1.11 (4.1.x) |
| [4.0.0.18](https://github.com/neo4j-contrib/neo4j-apoc-procedures/releases/4.0.0.18) | 4.0.12 (4.0.x) |
| [3.5.0.15](https://github.com/neo4j-contrib/neo4j-apoc-procedures/releases/3.5.0.15) | 3.5.30 (3.5.x) |
| [3.4.0.8](https://github.com/neo4j-contrib/neo4j-apoc-procedures/releases/3.4.0.8)   | 3.4.18 (3.4.x) |
| [3.3.0.4](https://github.com/neo4j-contrib/neo4j-apoc-procedures/releases/3.3.0.4)   | 3.3.9 (3.3.x)  |
| [3.2.3.6](https://github.com/neo4j-contrib/neo4j-apoc-procedures/releases/3.2.3.6)   | 3.2.14 (3.2.x) |
| [3.1.3.9](https://github.com/neo4j-contrib/neo4j-apoc-procedures/releases/3.1.3.9)   | 3.1.9 (3.1.x)  |
| [3.0.8.6](https://github.com/neo4j-contrib/neo4j-apoc-procedures/releases/3.0.8.6)   | 3.0.12 (3.0.x) |
| [3.5.0.0](https://github.com/neo4j-contrib/neo4j-apoc-procedures/releases/3.5.0.0)   | 3.5.0-beta01   |
| [3.4.0.2](https://github.com/neo4j-contrib/neo4j-apoc-procedures/releases/3.4.0.2)   | 3.4.5          |
| [3.3.0.3](https://github.com/neo4j-contrib/neo4j-apoc-procedures/releases/3.3.0.3)   | 3.3.5          |
| [3.2.3.5](https://github.com/neo4j-contrib/neo4j-apoc-procedures/releases/3.2.3.5)   | 3.2.3          |
| [3.1.3.8](https://github.com/neo4j-contrib/neo4j-apoc-procedures/releases/3.1.3.8)   | 3.1.5          |
### 2.2 安装和设置
Community社区版是个zip文件，因此只要选个目标位置解压就可以了，我选择如下位置：
```shell
C:\Program Files\neo4j-community-4.4.0
```
接下来就是添加环境变量了，包括两个操作：
1. 在“系统变量”中添加`NEO4J_HOME`，直接上图：
   ![N4J.png](https://s2.loli.net/2024/11/23/MRZlnuwQIoqNKkO.png)
2. 在“系统变量”的`Path`变量中，增加一项：
   `%NEO4J_HOME%\bin`
   ![PATH.png](https://s2.loli.net/2024/11/23/UBXJmxqLMjlvuhz.png)
两项完成之后，你打开命令行窗口，输入`neo4j`，应该就能可以看到下图这样的回应了，凡是提示找不到命令之类的，都是环境变量设置的问题，对比上面改就是了！也可以查询一下版本：
```shell
neo4j --version
```
![image](https://s2.loli.net/2024/11/23/aoyJPeUVY9DGnjb.png)
## 3. APOC 插件的安装
### 3.1 下载和放置位置
国内下载，用[这个链接](http://doc.we-yun.com:1008/doc/neo4j-apoc/)，你懂的。根据版本的匹配关系，我选了个匹配我的`Neo4j`版本`APOC 4.4.0.1`。里面有好几个，但最好下载`apoc-4.4.0.1-all`这个，因为这是最全的，避免后面又出幺蛾子。下载后放在你`Neo4j`文件夹下的`plugins`子文件夹下就可以了。以下是我的：
```shell
C:\Program Files\neo4j-community-4.4.0\plugins
```
### 3.2 设置!!!很重要
APOC只是下载了放在那是不会起作用的，当你运行`LightRAG`显示构建的图谱时，会一直报找不到`apoc`，必须要进行配置才可以。根据网上搜到的若干帖子，直接照抄过来都无法成功，经过反复测试，下面这两个改动部分按我的来就可以了。
找到`C:\Program Files\neo4j-community-4.4.0\conf`文件夹下`neo4j.conf`文件，在其中定位到如下被注释掉的两行（这两行不相邻，但很近）：
```python
#dbms.security.procedures.unrestricted=my.extensions.example,my.procedures.*
#dbms.security.procedures.allowlist=apoc.coll.*,apoc.load.*,gds.*
```
把它们改成如下的样子，并取消注释：
```python
dbms.security.procedures.unrestricted=apoc.*
dbms.security.procedures.allowlist=apoc.*
```
尤其是**最后一行**，就是我一直按网上帖子改不成功的原因。因为`LightRAG`好像使用了`apoc`的某个方法，但这里按原来写法，只放开了`apoc.coll.*`和`apoc.load.*`两个方法，所以一直报错找不到，我这里就把`apoc`下面的全放开了。
然后，应该就可以放心的启动`Neo4j`的服务了！
## 4. 跑起来！
### 4.1 安装并启动Neo4j服务
管理员模式打开命令行窗口，然后输入：
```shell
neo4j install-serive
```
接下来就可以启动服务了，两个方式：
- `neo4j console`：带状态反馈信息的启动，你可以在窗口看到服务状态信息，结束关闭需要`Ctrl+C`终止进程；
- `neo4j start`：不带状态，直接启动，可以在同一个窗口使用`neo4j stop`来终止进程。
之后，启动成功后，会出现下面的提示，这是用第一个方式启动的：
```shell
C:\Windows\System32>neo4j console
Directories in use:
home:         C:\Program Files\neo4j-community-4.4.0
config:       C:\Program Files\neo4j-community-4.4.0\conf
logs:         C:\Program Files\neo4j-community-4.4.0\logs
plugins:      C:\Program Files\neo4j-community-4.4.0\plugins
import:       C:\Program Files\neo4j-community-4.4.0\import
data:         C:\Program Files\neo4j-community-4.4.0\data
certificates: C:\Program Files\neo4j-community-4.4.0\certificates
licenses:     C:\Program Files\neo4j-community-4.4.0\licenses
run:          C:\Program Files\neo4j-community-4.4.0\run
Starting Neo4j.
2024-11-22 15:51:30.878+0000 INFO  Starting...
2024-11-22 15:51:31.453+0000 INFO  This instance is ServerId{baa33dc0} (baa33dc0-ad7c-4169-bd41-55a5d4c46e31)
2024-11-22 15:51:32.138+0000 INFO  ======== Neo4j 4.4.0 ========
2024-11-22 15:51:36.236+0000 INFO  Performing postInitialization step for component 'security-users' with version 3 and status CURRENT
2024-11-22 15:51:36.236+0000 INFO  Updating the initial password in component 'security-users'
2024-11-22 15:51:37.110+0000 INFO  Called db.clearQueryCaches(): Query cache already empty.
2024-11-22 15:51:37.472+0000 INFO  Bolt enabled on kubernetes.docker.internal:7687.
2024-11-22 15:51:37.896+0000 INFO  Remote interface available at http://localhost:7474/
2024-11-22 15:51:37.898+0000 INFO  id: 8C6AA1531BE4544853FE5EBC28F2E483EEEECFE841379E6A844F0B7C174E9384
2024-11-22 15:51:37.899+0000 INFO  name: system
2024-11-22 15:51:37.899+0000 INFO  creationDate: 2024-11-22T14:14:10.777Z
2024-11-22 15:51:37.899+0000 INFO  Started.
```
### 4.2 服务页面的配置
根据上面提示打开`http://localhost:7474`，就能够呈现这个页面：
![browser.png](https://s2.loli.net/2024/11/23/SGEUYOwbM76WteP.png)
首次需要连接到Neo4j的服务页面，在上面填写信息,初始用户和密码如下：
- Username: neo4j
- Password: neo4j
接着就是要求你改密码，按要求改就是了！然后就可以登录到新的页面：
![browser2.png](https://s2.loli.net/2024/11/23/GswLntcRkgV8qCe.png)
不放心可以用下面的命令测试一下：
```bash
neo4j$ return apoc.version()
```
应该能够成功返回你之前下载的`apoc`版本号。
### 4.3 展现LightRAG构建的图谱
关于LightRAG怎么部署和使用，以后有机会再单独说吧，按照官网的指导就可以完成构建。这里展示的是在完成LightRAG的构建之后，会生成一系列文件，我们把它对接到刚刚部署的`Neo4j`中进行展示。在`(lightrag) C:\Users\Dyson\Documents\LightRAG\examples>`目录下，先修改一下`graph_visual_with_neo4j.py`中关于登录页面的验证，就是改成之前设置的用户名和密码：
```python
# Neo4j connection credentials
NEO4J_URI = "neo4j://localhost:7687"
NEO4J_USERNAME = "neo4j"
NEO4J_PASSWORD = "YOU_PASSWORD"
```
然后运行:
```python
python graph_visual_with_neo4j.py
```
大功告成！
![KG.jpeg](https://s2.loli.net/2024/11/23/3XZf1iEYOBqVbTk.jpg)
如果出现：
```python
Error occurred: {code: Neo.ClientError.Security.Unauthorized} {message: The client is unauthorized due to authentication failure.}
```
说明刚才的密码设置不对！去改过来就可以了。
