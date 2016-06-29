---
title: RHCA培训散记（333网络安全 – 4）《关于基于网络的身份验证》
date: 2012-01-26T11:10:48+00:00
layout: post
---
## NIS

**1、NSS（Network Security Services）网络安全服务**，包含SSL v2、SSL v3、 TLS、 PKCS#5、 PKCS#7、 PKCS#11、 PKCS#12、 S/MIME、 X.509v3 certificates和其它安全协议；

**2、网络登录也和其它的登录服务一样，分为验证和授权两部分。**反应为密码和其它的信息（登录shell、UID、GID等）分开存放。一般的实现是用NIS（Network Information Service）存储其它信息（NIS也能存放密码，不过是明文传输的，所以不安全），用kerberos存放密码；

**3、上述的“其它信息”除了NIS，还可以选择使用LDAP**（Lightweight Directory Access Protocol）、DNS（Hesiod记录）；

**4、我们通常说的Linux中的RPC其实是指SUN实现的Sun RPC**，也叫ONC RPC（Open Network Computing Remote Procedure Call），可以建立在TCP或者UDP上，传输过程中数据会被编码成XDR（External Data Representation standard）（在RFC 1014中描述）格式；

**5、RPC中每个应用程序都有一个自己的Program ID**，可以在文件/etc/rpc中找到所有的ID的列表。其中最常用到的两个是NFS和NIS；

**6、RPC的一个弱点**是它会向其它的机器暴露自己可以提供所有基于RPC的应用的列表；

**7、nmap** -sR -sU -p 1-65535可以扫描目标机器的所有UDP端口，这会是个非常慢的过程，因为一般的标准Linux系统每秒只允许扫描20个UDP端口；

**8、NIS服务器支持一个master**，可以支持若干个slave服务器；

**9、NIS协议是基于Sun RPC的**，所以服务器和客户端都必须运行portmap，而且都必需加入同一个NIS域（其实就是一个预定义的统一的字符串）；

**10、NIS可以的存储的东西**相当于/etc/passwd、/etc/shadow、/etc/group中的内容；

**11、NIS协议有1、2、3。**NIS3又叫NIS+，在RHEL中只有客户端。服务器端只有版本1、2；

**12、NIS虽然使用明文传输**，安全性十分低下，但是由于其部署和维护简单，所以在低安全要求的企业环境中还是十分常用；

**13、NIS的包名和进程名都是ypserv**，同时rpc.yppasswdd和rpc.ypxfrd也是为它服务的。配置文件存在于/var/yp和/etc/yp*；

**14、NIS到底不安全在哪？**
  
a> 最大的问题是由于缺乏对客户端的认证机制和消息完整性校验机制，会泄露用户认证的信息，使用中间人的攻击方式伪装成认证服务器也不难；如果认证服务器都被伪装了，那后果的严重性可想而知（比如说更改UID为0取得root，删除已有用户防止正常人登陆，更改密码hash为已知值让自己登录）；
  
b> 其次，默认的配置是所有人都可以把passwd文件dump出来。而passwd采用的md5方式已经被认可为不可逆的，Linux依然使用的原因是假设passwd文件是受到保护的。因此将passwd文件的内容暴露在网上是很不安全的，是可以被反向破解出来的。这里提供一个字典破解法：openssl passwd -table -1 -salt &#8216;SAMESALT&#8217; &#8211; stdin < /usr/share/dict/words | grep 'passwdhash'； c> 最后，即使攻击者不破解密码，dump出来的用户信息也给他进行下一步攻击提供了舞台；

**15、NIS的安全怎么做？**
  
a> NIS基于UDP协议，所以不能使用TLS，只能使用IPsec（一组端对端的网络加密协议，在IPv6中是强制的，在IP4zhog是可选的，详见http://zh.wikipedia.org/wiki/IPsec）；
  
b> NIS绝对不能暴露在公网或者无线网络中，要严格限制在私有的可信的网络里。限制的方法可以使用tcpwrapper，或者使用/var/yp/securenets来限制网段；
  
c> 上述手段这样依然可以被IP伪装攻击绕过，所以在一定程度上要尽量不泄露NIS domain name（前述的那个字符串，有这个服务器才会响应你的请求）；
  
d> 禁用广播的方式去发现NIS Server，而是使用指定NIS的域名或者IP的方式。以防止前述恶意伪造NIS Server的行为；
  
e> 攻击者可以伪装NIS的响应，用已知的密码hash代替正确的NIS Server响应，这样攻击者就可以登入目标服务器了。要防止这种攻击，就要使用Kerberos（下述）来代替NIS存储密码的那部分功能了（这可以防止攻击者登入目标主机了，但这依然不能防止攻击者获取用户信息）；

## Kerberos

**1、Kerberos是一个基于共享密钥对称加密的安全网络认证系统**，它避免了将密码（包括密码hash）在网上传输，而是将密码作为对称加密的密钥，通过能不能解密来验证用户的身份；

**2、Kerberos在验证完用户身份后会发给用户Ticket**，这个Ticket包含了用户的授权，用户拿着这个Ticket去享受各种服务，所以在Kerberos管理的范围内用户只需要登录一次就可以享用所有的服务；

**3、如果拿Kerberos作为集群服务器的登录管理**，那么每台工作机（或者是跳板机，但是sshd不提供Kerberos的ticket初始化，需要自己改造一下）都必须是Kerberos于的成员；

**4、负责管理发放Ticket和记录授权的中心服务器被称为KDC**（Key Distribution Center），它知道所有用户和服务的密码。这就是Kerberos服务器；

**5、类似NIS 的name domian（域）在Kerboeros被称为realm**，Kerberos可以管理多个realm；

**6、在Kerberos域（realm）中每添加一个服务或者用户就要添加一条principal**，每个principal都有一个密码。用户principal的密码用户自己记住，服务的principal密码服务自己记录在硬盘上（keytab文件中）；

**7、用户principal的命名**类似elis/admin@EXAMPLE.COM，形式是用户名／角色／realm域。服务principal的命名类似ftp/station1.example.com@EXAMPLE.COM，形式是服务名／地址（提供者）／realm域；

**8、Kerberos的进程**有krb5kdc（主进程，也就是KDC）、kadmind（用于远程管理pricipal数据库）、kpropd（slave同步数据用）。配置文件在/etc/krb5.conf和/var/kerberos/krb5kdc/下，数据库在/var/kerberos/krb5kdc/princical；

**9、Kerberos用户登录过程**：
  
a> 用户提供用户名密码；
  
b> login程序（/usr/bin/login）把用户名转成princical名称，给KDC发送登录请求；
  
c> KDC生成一个用于对称加密的session key，也叫TGT（Ticket-granting Ticket） key。KDC自己留一份，然后复制一份用用户的密码（其实是princical的密码）加密后传给用户（login程序）；
  
d> login程序收到之后用刚才用户输入的密码解密，获得了TGT key。
  
e> 认证过程完成，之后的通信使用TGT key加密；

**10、Kerberos用户获得TGT之后登录服务过程**：
  
a> 用户向KDC请求服务授权（用户和KDC的沟通是用TGT加密的）；
  
b> KDC在验证princical通过后生成一个新的session key。将此key用用户的密码加密一次，再用服务的密码加密一次，把两次加密后的内容都发给用户；
  
c> 用户把第一份自己能解开的用TGT解密后得到session key（用于和服务沟通）。然后用此session key加密一个时间戳（作为一个对服务的验证）和自己解不开的那份用服务的密码加密的session key一并发给服务；
  
d> 服务用存储在keytab文件中的密码解密，也得到session key（这证明这玩意儿来自拥有自己密码的KDC）。然后解开用户加密的时间戳（这证明用户确实也拿到session key了，意即通过了KDC的允许）。这里面的时间戳用于防止重放攻击，所以所有的机器都必需使用安全的类似NTP的机制把时间同步好；
  
e> 登录服务过程完成，整个过程中大家都不知道对方的密码，密码也没有在网络中传输过。之后用户与服务的通信使用session key来加密；

**11、可以使用DNS SRV记录来寻找realm**，默认realm也可以通过DNS的TXT记录来设置。但是由于DNS还是比较容易被伪造和攻击，在配置文件中直接写上realm和默认realm是比较安全的；

**12、kerberos最初创建KDC的数据库**（krb5_util create）时是需要密码的，以后管理KDC或更改princical都需要这个密码。但是Kerberos在系统启动时却并不会停下来请求用户输入密码。这是因为kerberos将密码存在一个藏起来的文件里面了（当然也可以让它不这么干，创建时不使用 -s选项就行）；

**13、KDC可以通过kadmin.local在本地进行管理，也可以通过kadmin远程管理**。由于kadmin本身也可以视为一个服务，所以它也需要dump一份keytab（存储了它的密码）到本地来（存放在/var/kerberos/krb5kdc/kadm5.keytab），并用ktadd命令做好关联（此命令用于将KDC中的密码dump到本地）；

**14、kadmin**可以通过/var/kerberos/krb5kdz/kadmin5.acl设置相当灵活的（控制可否添加、删除、修改、修改密码、查询、浏览principals）ACL；

**15、虽然默认使用UDP，但RHEL使用的MIT KDC也可以设置TCP端口同样监听**。Kerberos服务的实现除了MIT KDC还有microsoft KDC（用于MS域）等；

**16、对称加密的算法可以在配置文件里设置；**

**17、支持Kerberos的服务**有ftp、http、NFS、login、telnet等，大多数都需要专用的服务器端程序（声名自己支持kerberos的）；

**18、部署Kerberos服务**只需要配置好krb5.conf（可以设置Ticket更新时间，是否forward ticket，最小UID之类的）后，在KDC中添加好princical，然后将密码dump回本地存为keytab给服务（如果服务原生不支持，但是支持pam的话可以尝试一下在pam中使用pam_krb5.so来间接支持）去用就ok了；

**19、临时Ticket被存储在/tmp／目录下**，名称是krb5cc_UID；

**20、ktutil**（其实就是一个查看keytab文件的工具）可以查看本地存储的密码的版本号（每改一次密码，版本号就会步进一次），kvno可以查看KDC中存储的密码的版本号（在客户端使用，有TGT才能用），一对比就能知道本地是否dump了最新的密码回来。这招在调试kerberos时很有用；

**21、kerberos自身的安全工作如何做？**
  
a> kerberos的安全是建立在主机都是安全的而网络不是安全的假定之上的。所以kerberos的安全其实就在于把主机的安全做好；
  
b> 尤其是KDC那台机器的安全。出于安全的考虑，跑KDC的机器上不能再跑别的服务，如果KDC被攻陷了，那么所有的密码就全部泄露了；
  
c> 如果只是服务的机器被攻陷了，那么更改服务的principal的密码即可；
  
d> 如果是用户的机器被攻陷了，那么在ticket超时（一般是数小时的时间里）之前，用户都是不安全的，攻击者还有可能尝试反向用户的密码；
  
e> Kerberos依赖其它的服务来存放用户信息（登录shell、UID、GID啥的），因此需要注意到这些信息依然是很容易遭受攻击并且泄露的；

f> 由于任何人都可以向KDC请求任何用户的TGT（使用用户密码加密的session key），那么攻击者就有可能请求一个这样的包下来尝试解密，他们有充足的时间离线去做这个工作，一旦解开了，他们也就拿到了用户的密码。简单密码几乎一解就开，所以不能设置简单密码，也不能在字典里。另外还可以打开Kerberos的预验证机制来防御这种攻击，预验证机制就是在KDC收到用户请求TGT的请求之后，要求用户先发一个用自己密码加密的时间戳过来给KDC，KDC如果确实可以用自己存储的用户密码解密，才发TGT给用户，这样攻击者在没有用户密码的时候就拿不到可用于反向的包含用户密码的TGT包了。在MIT kerberos中在配置文件中default\_principal\_flags = + preauth可以打开这个机制。但这个机制也并不是无懈可击的，攻击者依然可以通过嗅探的方式在正常用户请求TGT时拿到上述的那个包（这个难度显然就高了一些）；

g> 还有一个问题是攻击者可能伪造一个KDC，然后用一个伪造的实际不存在的用户向这个KDC请求验证，通过后他就得到了一个用户登录系统的shell。这种攻击需要在客户端防御，需要客户端主动去验证一下KDC是否是正确的KDC。具体来说就是客户端在得到TGT后进一步要求KDC给一个本物理机的principal（也就是一个用物理机密钥加密的串），然后尝试用物理机存储的密码去解密，由于伪造的KDC没有物理机（host／hostname）的principal密码，所以它无法给出这个包，也就被客户端认定为是伪造的KDC，认证失败。这个机制需要在客户端开启（认证服务器端么），默认是关闭的，在krb5.conf里［appdefaults］章节里pam的部分中设置validate=true来开启；

**22、Kerberos可以在不同的realm之间建立信任关系**，这样用户在一个realm中登录后就可以享用多个realm中的服务。简单的建立互信的方法是在2个域名都建立一条共享密码的名为krbtgt的principal。更好的建立互信的方法是多个realm都建立一条信任到自己的父realm，这样在子realm之间也能建立起信任；

**23、Kerberos还能向SSL帮助网络流量套上一层加密**，比如说telnet这种明文协议就可能能用上。但是跟SSL或者SSH比起来安全性仍然略低，因为它的session key是用对称加密传输的（如果TGT被盗取了，流量的加密也就失效了），这在理论上增加了被嗅探破解的可能。而SSH和SSL协商session key采用的DH算法是理论安全的（被称为PFS即Perfect forward secrecy，详见http://en.wikipedia.org/wiki/Perfect\_forward\_secrecy）。还有一个问题是Kerberos采用的一些加密算法是已经不安全的了（如CRC32和DES），而另一些加密算法（AES）并没有被其他的kerberos实现广泛使用，所以会有一些兼容性的麻烦，选择加密算法时要避开这些算法；

**24、如果将princical密码设置为random**，那么每次将其dump到本地keytab中时，都会random新的密码并dump到本地（可用于更新密码）；
