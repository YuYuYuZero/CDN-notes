# CDN技术详解

editor: YuYuYu

## 第1章 引言

内容分发网络：为人们服务的网络  “隐形的快递员”

### 1.1 CDN的基本概念和产生背景

Content Distribute Network / Content Delivery Network

完成将内容从源站传递到用户端的任务。

**Why CDN？**

> key point: **效率**

- 互联网由两层组成：

  - 以TCP/IP为代表的网络层（狭义互联网）
  - 以万维网WWW为代表的应用层

- 网络层与应用层的磨合还存在问题。

  影响互联网传输的4个因素：

  1. “第一公里”，流量向用户传输的第一个出口，网站服务器**接入互联网的链路**所能提供的**带宽**。
  2. “最后一公里”，万维网流量向用户传送的最后一段接入链路，即**用户接入带宽**。
  3. **对等互联关口**。“对等互联”指不同基础运营商之间的互联互通。*一般两个营运商之间只有两三个互联互通点*
  4. **长途骨干传输**。从网站服务器到用户之间要经过网站所在IDC、骨干网、用户所在城域网、用户所在接入网等。

<u>服务响应时间</u>由<u>服务器响应时间</u>和<u>网络时延</u>组成。

- 影响<u>服务器响应时间</u>的因素包括<u>协议处理时间</u>、<u>程序性能优化</u>、<u>内容读取速度</u>等
- <u>网络时延</u>是由数据报文在网络传送中被各个路由器、交换机转发产生的<u>时延总和</u>

互联网“8秒定律”，1秒的延迟会导致转化率下降7%。

为了解决上述一系列（效率）问题，造就了如今的互联网基础服务——CDN。

### 1.2 CDN的基本工作过程

使用CDN会极大地简化网站的系统维护工作量，网站维护人员只需将网站内容注入CDN系统，通过CDN部署在各个物理位置的服务器进行全网分发，就可以实现跨运营商、跨地域的用户覆盖。

由于CDN将内容推送到网络边缘，大量的用户访问被分散在网络边缘，不再构成网站出口、互联互通点的资源挤占，也不再需要跨越长距离IP路由了。

B/S架构 Browser-Server(浏览器-服务器)架构



**在没有CDN服务时，网站向用户提供服务的过程：**

![1-2](https://raw.githubusercontent.com/YuYuYuZero/imgstorage/master/20190313191204.png)

**在网站和用户之间加入CDN以后，一个典型的CDN用户访问调度流程：**

![1-3](https://raw.githubusercontent.com/YuYuYuZero/imgstorage/master/20190313191821.png)

(1) 当用户点击网站页面上的内容URL，经过本地DNS系统解析，DNS系统会最终将域名的解析权**交给CNAME指向的CDN专用DNS服务器**。

(2) CDN的DNS服务器将**CDN的全局负载均衡设备IP地址**返回用户。

(3) 用户向**CDN的全局负载均衡设备**发起内容URL访问请求。

(4) CDN全局负载均衡设备根据用户IP地址，以及用户请求的内容URL，选择一台用户所属区域的**区域负载均衡设备**，告诉用户向这台设备发起请求。

(5) 区域负载均衡设备会为用户**选择一台合适的缓存服务器**提供服务，选择的依据包括：根据用户<u>IP地址</u>，判断哪一台服务器距用户最近；根据用户所请求的<u>URL中所携带的内容名称</u>，判断哪一台服务器上有用户所需内容；查询各个服务器当前的<u>负载情况</u>，判断哪一台服务器尚有服务能力。基于以上这些条件的综合分析，区域负载均衡设备会向全局负载均衡设备返回一台**缓存服务器的IP地址**。

(6) 全局负载均衡设备把**服务器的IP地址**返回给用户。

(7) **用户向缓存服务器发起请求**，缓存服务器响应用户请求，将用户所需内容传送到用户终端。<u>如果这台缓存服务器上并没有用户想要的内容，而区域均衡设备依然将它分配给了用户，那么这台服务器就要向它的上一级缓存服务器请求内容，直至追溯到网站的源服务器将内容拉到本地。</u>

DNS服务器根据用户IP，将域名解析成响应节点的缓存服务器IP地址，实现用户就近访问。使用CDN服务的网站，只需将其域名解析权交给CDN的GSLB设备，将需要分发的内容注入CDN，就可以实现内容加速了。

### 1.3 CDN的发展历史

CDN是为互联网上的应用服务的，它伴随着互联网的发展而逐步成长。

发展曲线如图：

![1-4](https://raw.githubusercontent.com/YuYuYuZero/imgstorage/master/20190313194041.png)

### 1.4 CDN对互联网产业的价值和

业界普遍认为，CDN最终将成为架设在传统IP网之上的一个必需的内容传递网络，将对整个互联网的架构和产业格局产生深远的影响。

较大的CDN网络，尤其是商业化CDN网络，都部署了多层节点覆盖不同大小的区域，就像一个规划完善的物流系统，如图所示：

![1-5](https://raw.githubusercontent.com/YuYuYuZero/imgstorage/master/20190313202735.png)




## 第2章 CDN技术概述

在本章中，将一一讲述CDN是如何完成<u>内容缓存</u>、<u>负载均衡</u>、<u>流媒体加速</u>、<u>动态内容加速等任务</u>的，以及完成这些任务涉及的各种关键技术。

### 2.1 CDN的系统架构

#### 2.1.1 功能架构

CDN技术自诞生以来一直在持续演进完善，但基本的CDN功能架构在2003年左右就已基本形成和稳定了。

从功能上划分，典型的CDN系统架构由**分发服务系统**、**负载均衡系统**和**运营管理系统**三大部分组成，如图2-1所示。

![2-1](https://raw.githubusercontent.com/YuYuYuZero/imgstorage/master/20190315144953.png)

**分发服务系统**的主要作用是将<u>内容从内容源中心向边缘的推送和存储</u>，承担实际的内容数据流的全网分发工作和面向最终用户的数据请求服务。

分发服务系统最基本的工作单元就是许许多多的<u>Cache设备（缓存服务器）</u>，Cache负责直接响应最终用户的访问请求，把缓存在本地的内容快速地提供给用户。

同时Cache还负责与源站点进行内容同步，把更新的内容以及本地没有的内容从源站点获取并保存在本地。

一般来说，根据承载内容类型和服务种类的不同，分发服务系统会分为多个子服务系统，如<u>网页加速子系统</u>、<u>流媒体加速子系统</u>、<u>应用加速子系统</u>等。每一个子服务系统都是一个分布式服务集群，由一群功能近似的、在地理位置上分布部署的Cache或Cache集群组成，彼此间相互独立。

对于分发服务系统，在承担内容的更新、同步和响应用户需求的同时，还需要向上层的调度控制系统提供每个Cache设备的健康状况信息、响应情况，有时还需要提供内容分布信息，一边调度控制系统根据设定的策略决定由哪个Cache（组）来响应用户的请求最优。

**负载均衡系统**是一个CDN系统的神经中枢，主要功能是负责<u>对所有发起服务请求的用户进行访问调度</u>，<u>确定提供给用户的最终实际访问地址</u>。

大多数CDN系统的负载均衡系统是**分级**实现的，这里以最基本的两级调度体系进行简要说明：两级调度体系一版分为<u>全局负载均衡（GSLB）</u>和<u>本地负载均衡（SLB）</u>。

其中，<u>全局负载均衡（GSLB）</u>主要根据用户<u>就近性原则，</u>通过对每个服务节点进行“最优”判断，确定向用户提供服务的Cache的物理位置。最通用的GSLB实现方法是基于**DNS解析**的方式实现，也有一些系统采用了**应用层重定向**等方式来解决（将在第5章进行讲解）。

<u>本地负载均衡（SLB）</u>主要负责节点内部的设备负载均衡，当用户请求从GSLB调度到SLB时，SLB会根据节点内<u>各Cache设备的实际能力或内容分布等因素</u>对用户进行重定向，常用的本地负载均衡方法有基于4层调度、基于7层调度、链路负载调度等（将在第4章进行讲解）。

CDN的**运营管理系统**与一般的电信运营管理系统类似，分为<u>运营管理</u>和<u>网络管理</u>两个子系统。

<u>运营管理子系统</u>是CDN系统的业务管理功能实体，负责处理业务层面的与外界系统交互所必需的一些收集、整理、交付工作，包含客户管理、产品管理、计费管理、统计分析等功能。其中客户管理指对使用CDN业务的客户进行基本信息和业务规则信息的管理，作为CDN服务提供的依据。产品管理，指CDN对外提供的聚体产品包属性描述、产品生命周期管理、产品审核、客户产品状态变更等。计费管理，指在对客户使用CDN资源情况的记录的基础上，按照预先设定的计费规则完成计费并输出账单。统计分析模块负责从服务模块收集日常运营分析和客户报表所需数据，包括资源使用情况、内容访问情况、各种排名、用户在线情况等数据统计和分析，形成报表提供给网管人员和CDN产品使用者。

<u>网络管理子系统</u>实现对CDN系统的网络设备管理、拓扑管理、链路监控和故障管理，为管理员提供对全网资源进行集中化管理操作的界面，通常是**基于Web方式**实现的。

图2-2为国内第一家CDN服务商——蓝汛（ChinaCache）公司在2005年发布的《CDN技术白皮书》中描述的CDN系统架构，是一套实际的、运行良好的商用CDN系统。我们以该架构为例，与图2-1进行对照分析，帮助读者理解CDN系统的组成。

![2-2](https://raw.githubusercontent.com/YuYuYuZero/imgstorage/master/20190320161842.png)

在蓝汛CDN架构中，<u>CCN（ChinaCache Nod）</u>模块对应于图2-1中的<u>分发服务系统</u>，是CDN的基本服务模块，由分布于各个城市、各个运营商网络中的Cache设备和辅助设备组成。

<u>GAC（Global Access Controller）</u>模块对应于图2-1中的<u>负载均衡系统</u>，主要采用了智能DNS解析方案，负责通过域名解析应答实现将用户请求调度到最优服务节点的目的。

<u>NOC（Network Operating Center）</u>模块对应图2-1中的<u>网络管理系统</u>，负责对全网进行7×24小时的监控和管理。NOC可以监控ChinaCache CDN中的链路状况、节点响应速度、设备运行状况，也可以监控到客户的源站点运行状况等信息，一旦发现异常马上采取相应措施予以解决，是保障CDN服务安全可靠性的重要系统。值得一提的是，蓝汛的NOC中设置了源站点监控功能，对客户源站进行可达性监控，从而减轻或者避免由于源站故障造成的服务中断。

<u>OSS（Operating Support System）</u>模块对应图2-1中的<u>运营管理子系统</u>，负责采集和汇总各个CCN的日志记录信息，然后由中央处理软件加以整理和分析，最后通过客户服务门户进行发布。蓝汛的客户可以通过OSS提供的查询界面来查询加速页面或频道的实时流量、流量分布、点击数量、访问日志等信息。

#### 2.1.2 部署架构

CDN系统设计的首要目标是尽量减少用户的访问响应时间，为达到这一目标，CDN系统应该尽量将用户所需要的内容放在距离用户最近的位置。也就是说，负责为用户提供内容服务的Cache设备应部署在物理上的网络边缘位置，我们称这一层为CDN边缘层。CDN系统中负责全局性管理和控制的设备组成中心层，中心层同时保存着最多的内容副本，当边缘曾设备未命中时，会向中心层请求，如果在中心层仍未命中，则需要中心层向源站回源。不同CDN系统设计之间存在差异，中心层可能具备用户服务能力，也可能不直接提供服务，只向下级节点提供内容。如果CDN网络规模较大，边缘层设备直接向中心层请求内容或服务会造成中心层设备压力过大，就要考虑在边缘曾和中心层之间部署一个区域层，负责一个区域的管理和控制，也保存部分内容副本供边缘层访问。

### 2.2 CDN系统分类

### 2.3 小结

> 
>
> ## 第3章 内容缓存工作原理及实现
>
> 
>
> ## 第4章 集群服务与负载均衡技术
>
> 
>
> ## 第5章 全局负载均衡工作原理及实现
>
> 
>
> ## 第6章 流媒体CDN系统的组成和关键技术
>
> 
>
> ## 第7章 动态内容加速服务的实现
>
> 
>
> ## 第8章 CDN商业化服务现状
>
> 
>
> ## 第9章 CDN发展展望


