# 介绍

> *译自：[Introduction](https://github.com/sqlmapproject/sqlmap/wiki/Introduction)*

## 检测和利用 SQL 注入漏洞

假设你正在进行 Web 应用审计，发现有某个 Web 页面接受来自用户端提供的动态数据，这些数据通过 `GET`，`POST` 或 `Cookie` 参数或 HTTP `User-Agent` 请求头发送。  
因而，你想测试是否可以通过参数构造出 SQL 注入漏洞，如果有漏洞，则可以利用它们从后端 DBMS（Database Management System，数据库管理系统）中获取尽可能多的信息，甚至进一步控制底层的文件系统和操作系统。

简而言之，考虑下面的 url：

```shell
http://192.168.136.131/sqlmap/mysql/get_int.php?id=1
```

假设：

```shell
http://192.168.136.131/sqlmap/mysql/get_int.php?id=1+AND+1=1
```

页面显示跟原来的一样（AND+1=1 条件取值为 **True**（译者注：url 编码中 `+` 会被转成空格）），而：

```shell
http://192.168.136.131/sqlmap/mysql/get_int.php?id=1+AND+1=2
```  

页面显示跟原来的不一样（AND+1=2 条件取值为 **False**）。这可能说明 `index.php` 的 `GET` 参数 `id` 存在 SQL 注入漏洞。此外，这种情形也表明用户输入的数据在 SQL 语句传送到 DBMS 之前没有被过滤。

这种设计缺陷在动态网页应用中十分常见，此类型漏洞与后端 DBMS 或后端编程语言并没有关系，漏洞的引入通常存在于代码的编写逻辑里面。从 2013 年开始，[OWASP（Open Web Application Security Project，开放 Web 应用安全组织）](http://www.owasp.org)已将此类漏洞列入[最常见](https://www.owasp.org/images/f/f8/OWASP_Top_10_-_2013.pdf)（译者注：原链接失效，重新添加了有效链接）严重 Web 应用漏洞的[前十](http://www.owasp.org/index.php/Category:OWASP_Top_Ten_Project)名单。

从上面的例子我们发现了存在可以利用的参数，现在我们可以通过在每一次的 HTTP 请求中修改 `id` 进行相关的漏洞检测。

重回上面的情景，根据以往经验，我们可以对 `get_ini.php` 页面中如何使用用户提交的参数值构建出相应的 `SELECT` 语句做一个大致的猜测。参考下面的 PHP 伪代码：

```shell
$query = "SELECT [column name(s)] FROM [table name] WHERE id=" . $_REQUEST['id'];
```

如你所见，通过在 `id` 参数后面添加符合语法并且布尔值为 **True** 的 SQL 语句（例如 `id=1 AND 1=1`），能够使 Web 应用返回和之前合法请求（没有添加其它的 SQL 语句）一模一样的页面。由此可见，后端 DBMS 执行了先前我们注入的 SQL 语句。上面的例子展示了一个基于布尔值的简单 SQL 盲注。当然，sqlmap 可以检测出任意类型的 SQL 注入漏洞，并相应地调整其后续的工作流程。

在这个简单的场景中不只是可以添加一个或多个 SQL 判断条件语句，还可以（取决于 DBMS 类型）添加更多的 SQL 堆叠查询（Stacked queries）语句。例如：`[...]&id=1; 其他 SQL 查询语句#`。

sqlmap 能自动识别和利用这类型漏洞。将源链接 `http://192.168.136.131/sqlmap/mysql/get_int.php?id=1` 添加到 sqlmap，它能够自动完成下面操作：

* 识别有漏洞的参数（比如这个例子中的 `id`）
* 针对有漏洞的参数，自动选取对应类型的 SQL 注入技术
* 识别后端 DBMS 的相关指纹信息
* 根据用户使用的选项，它还能采集尽可能多的指纹信息，拉取数据或是掌管整个数据库服务器

网上还有很多深入讲解如何检测、利用和防止 SQL 注入漏洞的资源。推荐你在深入学习使用 sqlmap 之前阅读这些材料。

## 直连 DBMS

在 sqlmap **0.8** 版本推出前，sqlmap 被认为是 Web 应用渗透测试工程师/新手/技术爱好者等受众使用的**又一款 SQL 注入工具**。然而，随着使用者的成长，渗透技术的发展，sqlmap 同样也跟着演化。现在 sqlmap 支持一个新的开关选项 `-d`，它允许你通过 DBMS 守护进程监听的 TCP 端口直接连接目标数据库服务器，便于你使用 SQL 注入技术对目标数据库进行攻击。
