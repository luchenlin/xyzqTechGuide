# RESTFUL API规范

<div align=center> 
<img src="./images/restful.png" width = 80%/>
</div>

RESTful API 是每个程序员都应该了解并掌握的基本知识，我们在开发过程中设计 API 的时候也应该至少要满足 RESTful API 的最基本的要求（比如接口中尽量使用名词，使用 POST 请求创建资源，DELETE 请求删除资源等等，示例：<code>GET /notes/id</code>：获取某个指定 id 的笔记的信息）。  


**RESTful API 可以通过 url + http method 就知道这个 url 是干什么的，看到了 http 状态码（status code）就知道请求结果如何，例如：**  

>GET    /classes：列出所有班级  
POST   /classes：新建一个班级
***
### 1、重要概念  
REST,即 REpresentational State Transfer 的缩写。这个词组的翻译过来就是"表现层状态转化"。这样理解起来甚是晦涩，实际上 REST 的全称是 Resource Representational State Transfe ，直白地翻译过来就是 “资源”在网络传输中以某种“表现形式”进行“状态转移” 。如果还是不能继续理解，请继续往下看，相信下面的讲解一定能让你理解到底啥是REST 。  

我们分别对上面涉及到的概念进行解读，以便加深理解，不过实际上你不需要搞懂下面这些概念，也能看懂我下一部分要介绍到的内容。不过，为了更好地能跟别人扯扯 “RESTful API”我建议你还是要好好理解一下！  

**资源（Resource）** ：我们可以把真实的对象数据称为资源。一个资源既可以是一个集合，也可以是单个个体。比如我们的班级 classes 是代表一个集合形式的资源，而特定的 class 代表单个个体资源。每一种资源都有特定的 URI（统一资源定位符）与之对应，如果我们需要获取这个资源，访问这个 URI 就可以了，比如获取特定的班级：/class/12。另外，资源也可以包含子资源，比如 /classes/classId/teachers：列出某个指定班级的所有老师的信息  
**表现形式（Representational）**："资源"是一种信息实体，它可以有多种外在表现形式。我们把"资源"具体呈现出来的形式比如 json，xml，image,txt 等等叫做它的"表现层/表现形式"。  
**状态转移（State Transfer）** ：大家第一眼看到这个词语一定会很懵逼？内心 BB：这尼玛是啥啊？ 大白话来说 REST 中的状态转移更多地描述的服务器端资源的状态，比如你通过增删改查（通过 HTTP 动词实现）引起资源状态的改变。ps:互联网通信协议 HTTP 协议，是一个无状态协议，所有的资源状态都保存在服务器端。  

总结如下：  
1、每一个 URI 代表一种资源；  
2、客户端和服务器之间，传递这种资源的某种表现形式比如 json，xml，image,txt 等等；  
3、客户端通过特定的 HTTP 动词，对服务器端资源进行操作，实现"表现层状态转化"。  

### 2、REST 接口规范  
#### 2.1 动作
+ **GET** ：请求从服务器获取特定资源。举个例子：<code>GET /classes</code>（获取所有班级）
+ **POST** ：在服务器上创建一个新的资源。举个例子：<code>POST /classes</code>（创建班级）
+ **PUT** ：更新服务器上的资源（客户端提供更新后的整个资源）。举个例子：<code>PUT /classes/12</code>（更新编号为 12 的班级）
+ **DELETE** ：从服务器删除特定的资源。举个例子：<code>DELETE /classes/12</code>（删除编号为 12 的班级）
+ **PATCH** ：更新服务器上的资源（客户端提供更改的属性，可以看做作是部分更新），使用的比较少，这里就不举例子了。  

#### 2.2 路径（接口命名）  
   路径又称"终点"（endpoint），表示 API 的具体网址。实际开发中常见的规范如下：  
**1.网址中不能有动词，只能有名词，API 中的名词也应该使用复数。** 因为 REST 中的资源往往和数据库中的表对应，而数据库中的表都是同种记录的"集合"（collection）。如果 API 调用并不涉及资源（如计算，翻译等操作）的话，可以用动词。  
比如：<code>GET /calculate?param1=11& param2=33</code>  
**2.不用大写字母，建议用中杠** - 不用下杠 _ 比如邀请码写成 invitation-code而不是 ~~invitation_code~~  

**接口尽量使用名词，禁止使用动词。** 下面是一些例子：  

    GET    /classes：列出所有班级  
    POST   /classes：新建一个班级  
    GET    /classes/classId：获取某个指定班级的信息  
    PUT    /classes/classId：更新某个指定班级的信息（一般倾向整体更新）  
    PATCH  /classes/classId：更新某个指定班级的信息（一般倾向部分更新）  
    DELETE /classes/classId：删除某个班级  
    GET    /classes/classId/teachers：列出某个指定班级的所有老师的信息  
    GET    /classes/classId/students：列出某个指定班级的所有学生的信息  
    DELETE classes/classId/teachers/ID：删除某个指定班级下的指定的老师的信息  

反例：  

    /getAllclasses
    /createNewclass
    /deleteAllActiveclasses
### 3、过滤信息（Filtering）
在查询的时候需要添加特定条件的话，建议使用 url 参数的形式。如要查询 state 状态为 active 并且 name 为 guidegege 的班级：

    GET    /classes?state=active&name=guidegege  
要实现分页查询：  

    GET    /classes?page=1&size=10 //指定第1页，每页10个数据

### 4、状态码（Status Codes）

|2xx：成功|	3xx：重定向|	4xx：客户端错误|	5xx：服务器错误|
|  ----  | ----  |    ----  |          ----  |
|200 成功|	301 永久重定向|	400 错误请求|	500 服务器错误|
|201 创建|  	304 资源未修改|	401 未授权|	502 网关错误|
|        |             |403 禁止访问	|  504 网关超时|
|        |             |404 未找到	|              |
|        |             |405 请求方法不对|            |