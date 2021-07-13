---
title: 常见场景题
top: false
cover: false
toc: true
mathjax: true
date: 2021-05-24 22:24:00
password:
summary:
tags:
- interview
categories:
- interview
---

## 大数据

**1、100G 的手机号文件，找到重复的手机号，将重复手机号放入另一个文件。PC 机内存1G**

```
按照手机前三位分成1000 个文件，然后hashmap 或者bitmap 进行重复校验）
```

**2、1G大小的一个文件中找出出现频率最高的100个数**

```
（1）此处1G文件远远大于1M内存，分治法，先hash映射把大文件分成很多个小文件，具体操作如下：读文件中，对于每个词x，取hash(x)%5000，然后按照该值存到5000个小文件(记为f0,f1,...,f4999)中，这样每个文件大概是200k左右（每个相同的词一定被映射到了同一文件中）
（2）对于每个文件fi，都用hash_map做词和出现频率的统计，取出频率大的前100个词（怎么取？topK问题，建立一个100个节点的最小堆），把这100个词和出现频率再单独存入一个文件
（3）根据上述处理，我们又得到了5000个文件，归并文件取出top100（Top K 问题，比较最大的前100个频数）
```

**3、海量日志数据，提取出某日访问百度次数最多的那个IP。**

```
分而治之+Hash
1.IP地址最多有2^32=4G种取值情况，所以不能完全加载到内存中处理；
2.可以考虑采用“分而治之”的思想，按照IP地址的Hash(IP)%1024值，把海量IP日志分别存储到1024个小文件中。这样，每个小文件最多包含4MB个IP地址；
3.对于每一个小文件，可以构建一个IP为key，出现次数为value的Hash map，同时记录当前出现次数最多的那个IP地址；
4.可以得到1024个小文件中的出现次数最多的IP，再依据常规的排序算法得到总体上出现次数最多的IP；
```

**4、统计最热门的10个查询串，要求使用的内存不能超过1G，搜索引擎会通过日志文件把用户每次检索使用的所有检索串都记录下来，每个查询串的长度为1-255字节。**

```
第一步、先对这批海量数据预处理，在O（N）的时间内用Hash表完成统计;
第二步、借助堆这个数据结构，找出Top K，时间复杂度为N'logK。
即，借助堆结构，我们可以在log量级的时间内查找和调整/移动。因此，维护一个K(该题目中是10)大小的小根堆，然后遍历300万的Query，分别 和根元素进行对比所以，我们最终的时间复杂度是：O（N） + N'*O（logK），（N为1000万，N'为300万）。
```

**5、给定a、b两个文件，各存放50亿个url，每个url各占64字节，内存限制是4G，让你找出a、b文件共同的url？**

```
可以估计每个文件安的大小为5G×64=320G，远远大于内存限制的4G。所以不可能将其完全加载到内存中处理。考虑采取分而治之的方法。

遍历文件a，对每个url求取hash(url)%1000，然后根据所取得的值将url分别存储到1000个小文件（记为a0,a1,…,a999）中。这样每个小文件的大约为300M。

遍历文件b，采取和a相同的方式将url分别存储到1000小文件（记为b0,b1,…,b999）。这样处理后，所有可能相同的url都在对应的小文件（a0vsb0,a1vsb1,…,a999vsb999）中，不对应的小文件不可能有相同的url。然后我们只要求出1000对小文件中相同的 url即可。

求每对小文件中相同的url时，可以把其中一个小文件的url存储到hash_set中。然后遍历另一个小文件的每个url，看其是否在刚才构建的hash_set中，如果是，那么就是共同的url，存到文件里面就可以了。
```

**6、2.5亿个整数中找出不重复的整数，注，内存不足以容纳这2.5亿个整数。**

```
采用2-Bitmap（每个数分配2bit，00表示不存在，01表示出现一次，10表示多次，11无意义）进行，共需内存2^32 * 2 bit=1 GB内存，还可以接受。然后扫描这2.5亿个整数，查看Bitmap中相对应位，如果是00变01，01变10，10保持不变。所描完事后，查看 bitmap，把对应位是01的整数输出即可。
```

**7、给40亿个不重复的unsigned int的整数，没排过序的，然后再给一个数，如何快速判断这个数是否在那40亿个数当中？**

申请512M的内存，一个bit位代表一个unsigned int值。读入40亿个数，设置相应的bit位，读入要查询的数，查看相应bit位是否为1，为1表示存在，为0表示不存在。

**8、大日志中获取到指定时间段的数据**

> 与传统的二分搜索不一样，日志文件中的每一条日志长度都是不一样的，我们使用` mid = start + (end - start) / 2 `得到mid时，并不直接指向一行的行首。所以我们需要找到`mid`所在行的行首。

**9、海量数据如何排序**

**外部排序**

海量数据不能一次性读入内存，在对海量数据进行排序时，首先需要将海量数据拆分到多台机器或者多个文件，这些机器或文件称为拆分节点；然后在每个拆分节点上将数据全部读入内存并使用快速排序等方法进行排序；最后在合并节点使用**多路归并方法**将所有拆分节点的部分排序结果整合成最终的排序结果。外部排序也可以被称为外部归并排序。
如果不进行额外处理，**合并节点仍然无法将所有数据读入内存中。可以使用小顶堆**来解决这个问题：

- 假设有 k 个拆分节点，从这 k 个拆分节点分别读取一个最小的数据到小顶堆中。
- 将堆顶数据移出堆并写入合并节点的最终结果文件中。
- 确定刚才从堆中移除的数据属于哪个拆分节点，并从该拆分节点再读入一个数据。

上面的做法需要频繁地读写磁盘，可以设置输入缓存和输出缓存来解决这个问题。为每个拆分节点都设置一个输入缓存，每次将一部分数据读入输入缓存中，只有当输入缓存数据为空时才再从磁盘读入数据。并设置一个输出缓存，只有输出缓存满时才将数据写出磁盘中。

**BitMap**

如果待排序的数据是整数，或者其它范围比较小的数据时，可以使用 BitMap 对其进行排序。BitMap 相当于一个比特数组，如果某个数据存在时就将对应的比特数组位置设置为 1，最后从头遍历比特数组就能得到一个排序的整数序列。

这种方法**只能处理数据不重复的情况**，如果数据重复，就要将**比特数组转换成整数数组用于计数**，这种排序方法叫做计数排序。可以把整数数组看成 32 个比特数组，32 比特可以存放的计数最大值为 232 ，在某些场景下数据的重复量不会这么大，只需要几个比特数组就能完成计数操作。

**10、100亿数据找中位数**

> （1）我们要划分映射区域，一个有符号的32位整数的取值范围是[-2^31, 2^31-1]，总共有4294967296个取值，因此我们将它划分成100000组，即43000个数映射到一个组，将a1的区间[-2^31，-2^31+43000)，a2的区间[-2^31+43000，-2^31+86000)......一直到a100000的区间；（这是组数与项数的一个平衡问题）；
>
> （2.1）我们首先装载第一个1亿个数，遍历这些数，比较大小，看他落入a1至a100000的哪个区间，落入的对应区间统计计数增1；这次是对这里面的数区间的组映射；
>
> （2.2）重复步骤（2.1），装载100次，这样我们就得到了a1至a100000的区间统计计数的取值；
>
> （2.3）内存分析：1亿个数用来装载，100000个区间统计计数耗费400000个字节，足够使用；剩余内存（128M-1亿-100000）*4B;
>
> （3.1）使用sum依次累加a1至a100000的区间统计计数，直到累加某区间ai后sum大于50亿了；那么第50亿个数就在该区间中，用sum减去该区间ai的统计数的到first；即前面的区间统计总数位置为第first个（其中first < 50亿）;
>
> （3.2）那么我就在ai区间找到第50亿-first个数，或第50亿-first+1个数（第50亿-first+1个数这个数可能在ai后面的区间，但是概率很小，但是找到的原理类似）；
>
> （3.3）内存分析：每一个区间分割比较要花费100000个区间比较数，耗费400000个字节，足够使用；剩余内存（128M-1亿-100000-100000-2）*4B;
>
> （4.1）再次遍历这100亿个数，还是每组1亿个数，一共100组；对于若在ai区间的43000个数的每一个都开一个统计计数器 ，跟上面类似，这次是对这里面的数单个映射；
>
> （4.2）同样使用sum依次累加这1至43000的的统计计数，直到累加某区间后sum大于50亿-first；那么我们可以得到第（50亿-first）个数就在对应的位置；而且第（50亿-first+1）个数位置也有可能在，或在下一个统计计数大于0的位置；当然也有可能不在ai区间；（但原理类似）；
>
> （4.3）得到了第（50亿-first）个数值；而且第（50亿-first+1）个数值，可算出中位数了；
>
> （4.4）内存分析：上述的100000个比较数，此时我们只需要两个比较数；100000个区间统计计数全部释放掉，但增加了43000位置统计计数；剩余内存（128M-1亿-43000-2-2）*4B；还是足够使用的；
>
> （5）总共遍历两遍100亿数据

11、10000个IP，快速查找某IP是否存在，使用什么数据结构?

>使用BitMap，需要使用32位。

## 算法

### **银行家算法**

我们可以把操作系统看作是银行家，操作系统管理的资源相当于银行家管理的资金，进程向操作系统请求分配资源相当于用户向银行家贷款。
为保证资金的安全,银行家规定:

- 当一个顾客对资金的最大需求量不超过银行家现有的资金时就可接纳该顾客;
- 顾客可以分期贷款,但贷款的总数不能超过最大需求量;
- 当银行家现有的资金不能满足顾客尚需的贷款数额时,对顾客的贷款可推迟支付,但总能使顾客在有限的时间里得到贷款;
- 当顾客得到所需的全部资金后,一定能在有限的时间里归还所有的资金.

### 哲学家吃饭问题怎么解决？

> 五个哲学家共用一张圆桌，分别坐在周围的五张椅子上，在桌子上有五只碗和五只筷子，他们的生活方式是交替地进行思考和进餐。平时，一个哲学家进行思考，饥饿时便试图取用其左右最靠近他的筷子，只有在他拿到两只筷子时才能进餐。进餐毕，放下筷子继续思考。

可以用一个信号量表示筷子，由这**五个信号量构成信号量数组**。下面是死锁的解决：

**策略一**原理：至多只允许四个哲学家同时进餐，以保证至少有一个哲学家能够进餐，最终总会释放出他所使用过的两支筷子，从而可使更多的哲学家进餐。定义信号量count，只允许4个哲学家同时进餐，这样就能保证至少有一个哲学家可以就餐。

**策略二**原理：仅当哲学家的**左右两支筷子都可用**时，才允许他拿起筷子进餐。可以利用AND 型信号量机制实现，也可以利用信号量的保护机制实现。利用信号量的保护机制实现的思想是通过记录型信号量mutex对取左侧和右侧筷子的操作进行保护，使之成为一个原子操作，这样可以防止死锁的出现。

**策略三**原理：规定奇数号的哲学家先拿起**他左边**的筷子，然后再去拿他右边的筷子；而偶数号的哲学家则先拿起**他右边**的筷子，然后再去拿他左边的筷子。

## 系统设计

### 单点登录

单点登录就是**在多个系统中，用户只需一次登录，各个系统即可感知该用户已经登录。**

**Session不共享问题**

> - SSO系统生成一个token，并将用户信息存到Redis中，并设置过期时间
> - 其他系统请求SSO系统进行登录，得到SSO返回的token，写到Cookie中
> - 每次请求时，Cookie都会带上，拦截器得到token，判断是否已经登录

针对Cookie存在**跨域问题**，有几种解决方案：

1. 服务端将Cookie写到客户端后，客户端对Cookie进行解析，将Token解析出来，此后请求都把这个Token带上就行了
2. 多个域名共享Cookie，在写到客户端的时候设置Cookie的domain。
3. 将Token保存在SessionStroage中（不依赖Cookie就没有跨域的问题了）

**CAS（Central Authentication Service）原理**

- 用户想要访问系统A`www.java3y.com`受限的资源，系统A`www.java3y.com`发现用户并没有登录，于是**重定向到sso认证中心，并将自己的地址作为参数**。
- sso认证中心发现用户未登录，将用户引导至登录页面，用户进行输入用户名和密码进行登录，用户与认证中心建立**全局会话（生成一份Token，写到Cookie中，保存在浏览器上）**
- 认证中心**重定向回系统A**，并把Token携带过去给系统A
- 系统A去sso认证中心验证这个Token是否正确，如果正确，则系统A和用户建立局部会话（**创建Session**）。到此，系统A和用户已经是登录状态了。
- 此时，用户想要访问系统B`www.java4y.com`受限的资源，系统B`www.java4y.com`发现用户并没有登录，于是**重定向到sso认证中心，并将自己的地址作为参数**，这次系统B**重定向**到认证中心`www.sso.com`是可以带上Cookie的。
- 认证中心**根据带过来的Cookie**发现已经与用户建立了全局会话了，认证中心**重定向回系统B**，并把Token携带过去给系统B
- 系统B去sso认证中心验证这个Token是否正确，如果正确，则系统B和用户建立局部会话（**创建Session**）。到此，系统B和用户已经是登录状态了。

### JWT

#### **令牌结构**

由三部分组成，这些部分由点分隔

- Header：包括令牌的类型（即JWT）和所使用的签名算法（SHA256或者RSA）
- Payload：有关实体（通常是用户）和其他数据的声明
  - 注册声明：提供一组有用的可互操作的权利要求。其中一些是： iss（JWT的签发者）， exp（expires,到期时间）， sub（主题）， aud（JWT接收者），iat(issued at，签发时间)等
  - 公开声明：一般添加用户的相关信息或其他业务需要的必要信息。不建议添加敏感信息，因为该部分在客户端可解密。
  - 私有声明：提供者和消费者所共同定义的声明，一般不建议存放敏感信息
- Signature：Signature部分的生成需要base64编码之后的Header,base64编码之后的Payload,密钥（secret）,Header需要指定签字的算法。

#### 缺点

- JWT 的最大缺点是，由于服务器不保存 session 状态，因此无法在使用过程中废止某个 token，或者更改 token 的权限。也就是说，一旦 JWT 签发了，**在到期之前就会始终有效**，除非服务器部署额外的逻辑。
- JWT 本身包含了认证信息，**一旦泄露，任何人都可以获得该令牌的所有权限**。为了减少盗用，JWT 的有效期应该设置得比较短。对于一些比较重要的权限，使用时应该再次对用户进行认证。
- 为了减少盗用，**JWT 不应该使用 HTTP 协议明码传输，要使用 HTTPS 协议传输**。

#### 用户登出，如何设置token无效？

JWT是无状态的，用户登出设置token无效就已经违背了JWT的设计原则，但是在实际应用场景中，这种功能是需要的，那该如何实现呢？提供几种思路：

- 用户登出，浏览器端丢弃token
- 使用redis数据库，用户登出，从redis中删除对应的token,请求访问时，需要从redis库中取出对应的token,若没有，则表明已经登出

> 为了保持数据的一致性，每一次认证都需要从redis中取出对应的token，每一次都以redis中的token为准。

#### 使用redis,两个不同的设备，一个设备登出，另外一个设备如何处理？

请思考这样一种场景：

- 同一个用户从两个设备登陆到服务端（设备1，设备2）；
- 设备1登出，删除redis中的对应的token - 设备2再次请求数据时，redis中的数据为空，需要重新登录。

很明显，这种情况是不应该出现的，说一下自己的想法：

- 每一个设备与用户生成唯一的key,保存在redis中，即设备1的用户登出，只删除对应的token，设备2的token仍然存在

### OAuth

OAuth 就是一种授权机制。数据的所有者告诉系统，同意授权第三方应用进入系统，获取这些数据。系统从而产生一个短期的进入令牌（token），用来代替密码，供第三方应用使用。

**OAuth 2.0 规定了四种获得令牌的流程。**

- 授权码（authorization-code）

**指的是第三方应用先申请一个授权码，然后再用该码获取令牌。**

> 1. A 网站让用户跳转到 GitHub。
> 2. GitHub 要求用户登录，然后询问"A 网站要求获得 xx 权限，你是否同意？"
> 3. 用户同意，GitHub 就会重定向回 A 网站，同时发回一个授权码。
> 4. A 网站使用授权码，向 GitHub 请求令牌。
> 5. GitHub 返回令牌.
> 6. A 网站使用令牌，向 GitHub 请求用户数据。

- 隐藏式（implicit）
- 密码式（password）：
- 客户端凭证（client credentials）

### 验证码登录

验证码是由服务端产生，以图片的形式展示在客户端或页面，

用户端的用户根据图片识别验证码，并进行注册提交，

提交的验证码在服务层进行校验，如果校验成功，则用户注册成功并登陆

### 限流器

#### 令牌桶

 假设允许的请求速率为`r`次每秒，那么每过`1/r`秒就会向桶里面添加一个令牌。桶的最大大小是`b`。当请求到来时，检查桶内令牌数是否足够，如果足够，令牌数减少，请求通过。不够的话就会触发拒绝策略。

如果令牌不被消耗，或者被消耗的速度小于产生的速度，令牌就会不断地增多，直到把桶填满。令牌桶在保持整体上的请求速率的同时，允许某种程度的突发传输。

- 限制调用的平均速率的同时还允许一定程度的突发调用

#### 漏斗桶

漏斗桶控制请求必须在最大某个速率被消费，就像一个漏斗一样，入水量可大可小，但是最大速率只能到某一量值，不会像令牌桶一样，会有小的尖峰。

 主要实现方式是通过一个 FIFO （First in first out）的队列实现，这个队列是一个有界队列，大小为`b`，如果请求堆积满了队列，就会触发丢弃策略。假设允许的请求速率为`r`次每秒，那么这个队列中的请求，就会以这个速率进行消费。

- 能够限制请求调用的速率

> **漏桶**
>
> 漏桶的出水速度是恒定的，那么意味着如果瞬时大流量的话，将有大部分请求被丢弃掉（也就是所谓的溢出）。
>
> **令牌桶**
>
> 生成令牌的速度是恒定的，而请求去拿令牌是没有速度限制的。这意味，面对瞬时大流量，该算法可以在短时间内请求拿到大量令牌，而且拿令牌的过程并不是消耗很大的事情。
>
> 个人觉得令牌桶是一个更加精细化的东西，因此避免了集中丢失，本质无太大差别。网上说法可参考，个人觉得不一定对。

#### 滑动日志

根据缓存之前接受请求对应的时间戳，与当前请求的时间戳进行计算，控制速率。这样可以严格限制请求速率。

假设`n`秒内最多处理`b`个请求。那么会最多缓存 `b` 个通过的请求与对应的时间戳，假设这个缓存集合为`B`。每当有请求到来时，从`B`中删除掉`n`秒前的所有请求，查看集合是否满了，如果没满，则通过请求，并放入集合，如果满了就触发拒绝策略。

#### 固定时间窗口

 假设`n`秒内最多处理`b`个请求，那么每隔`n`秒将计数器重置为`b`。请求到来时，如果计数器值足够，则扣除并请求通过，不够则触发拒绝策略。实现简单，适用于一些要求不严格的场景。

- `突刺现象`：如果我在单位时间1s内的前10ms，已经通过了100个请求，那后面的990ms，只能眼巴巴的把请求拒绝

### 短连接生成转换设计

正确的原理就是通过发号策略，给每一个过来的长地址，发一个号即可，小型系统直接用mysql的自增索引就搞定了。如果是大型应用，可以考虑各种分布式key-value系统做发号器。不停的自增就行了。

第一个使用这个服务的人得到的短地址是 http://xx.xx/0 ，第二个是 http://xx.xx/1 ，第11个是 http://xx.xx/a ，依次往后，相当于实现了一个62进制的自增字段即可。
**如何保证同一个长地址，每次转出来都是一样的短地址**

用key-value存储，保存“最近”生成的长对短的一个对应关系。注意不保存全量的长对短的关系，而只保存最近的。比如采用一小时过期的机制来实现LRU淘汰。

>- 在这个“最近”表中查看一下，看长地址有没有对应的短地址，有就直接返回，并且将这个key-value对的过期时间再延长成一小时
>- 如果没有，就通过发号器生成一个短地址，并且将这个“最近”表中，过期时间为1小时
>  所以当一个地址被频繁使用，那么它会一直在这个key-value表中，总能返回当初生成那个短地址，不会出现重复的问题。如果它使用并不频繁，那么长对短的key会过期，LRU机制自动就会淘汰掉它。
>   当然，这不能保证100%的同一个长地址一定能转出同一个短地址，比如你拿一个生僻的url，每间隔1小时来转一次，你会得到不同的短地址。

**保证发号器的大并发高可用**

我们可以实现1000个逻辑发号器，分别发尾号为0到999的号。每发一个号，每个发号器加1000，而不是加1。这些发号器独立工作，互不干扰即可。

### RPC框架设计

#### 网络IO模型

最被广泛使用的是多路 I/O 复用，Linux 系统中的 select、epoll 等系统调用都是支持多路 I/O 复用模型的

#### 序列化方式

- 如果对于性能要求不高，在传输数据占用带宽不大的场景下可以使用 JSON 作为序列化协议；
- 如果对于性能要求比较高，那么使用 Thrift 或者 Protobuf 都可以。而 Thrift 提供了配套的 RPC 框架，所以想要一体化的解决方案，你可以优先考虑 Thrift；
- 在一些存储的场景下，比如说你的缓存中存储的数据占用空间较大，那么你可以考虑使用 Protobuf 替换 JSON 作为存储数据的序列化方式。

#### TCP相关

开启 tcp_nodelay，这个参数关闭了 Nagle算法

#### **服务寻址**

- 服务提供者启动后主动向服务（注册）中心注册机器ip、端口以及提供的服务列表。
- 服务消费者启动时向服务（注册）中心获取服务提供方地址列表，可实现软负载均衡和Failover。
- 提供者需要定时向注册中心发送心跳，一段时间未收到来自提供者的心跳后，认为提供者已经停止服务，从注册中心上摘取掉对应的服务等等。

**1）服务注册**

首先需要把服务注册到服务中心。其实就是在注册中心进行一个登记，注册中心存储了该服务的IP、端口、调用方式(协议、序列化方式)等。在zookeeper中，进行服务注册，实际上就是在zookeeper中创建了一个znode节点，该节点存储了上面所说的服务信息。

**2）服务发现**

服务消费者在第一次调用服务时，会通过注册中心找到相应的服务的IP地址列表，并缓存到本地，以供后续使用。当消费者调用服务时，不会再去请求注册中心，而是直接通过负载均衡算法从IP列表中取一个服务提供者的服务器调用服务。

## 逻辑题

1、很多根绳子，每一根都不一样且粗细不均匀，每根绳子从头烧到尾都是60分钟烧完，怎么用绳子去测量15分钟的时间

```
取出三条绳子。1、同时点燃“第一根的两头”和“第二根的一头”，第一根烧完时间过了“30分钟”；2、第一根烧完后马上点燃第二根的另一头，到第二根烧完时间又过了“15分钟”；
```

2、1000杯水，其中一杯有毒，用老鼠试毒，老鼠24h才会死，需要多少只才能找出这杯有毒的水（可以稀释）

```
将1000杯水编号(1-1000),将其转化为2进制码，取10只小白鼠（为什么是10只，因为其1000的2进制码长度是10位），给10只小白鼠编号1-10，第一只小白鼠喝第一位2进制码为1的（1000杯中2进制码第一位为1的都要喝），第二只小白鼠喝第二位2进制为1的（1000杯中2进制码第二位的都要喝）以此类推一直到第10只小白鼠喝完，然后1小时后看那几只小白鼠会死，死掉的小白鼠用1表示，未死的用0表示整理出10位2进制码
```

3、10个箱子，每个箱子100跟金条，每个1两，一个贪官，在其中一个箱子里面，每根都磨去了一钱，只能称一次，哪个箱子被磨去了一钱。

```
第一箱子拿1块，第二箱子拿2块，第n箱子拿n块，然后放在一起称，看看缺了几钱，缺了n钱就说明是第n个箱子。
```

4、有八个球，只能称两次（天平称）只有一个球最重谁能找出？

> 8个球分成3份，分别是2个球，3个球，3个球
>
> 3个球和3个球称，如果一样重的话，那证明重的球在那一份2个球的，两边各放一个，重的球就可以找到了。

5、回到扑克牌的这个主题，要求把一堆乱序的扑克牌进行 排序 ，如果要极致地压榨性能，应该怎么做？时间复杂度能达到多少？

> 扑克牌本身规律8个1,8个2...

6、假设现在有1-100号乘客顺序登机，正常情况下1号乘客需要坐1号位置，以此类推，大家顺序入座。假设现在1号乘客随机选择了一个座位坐下，2-100号乘客优先看自己的座位是否被占，自己座位被占的情况下会从剩下的座位中随机选一个坐下，否则坐自己的座位； 那么这种情况下，100号乘客可以做到100号位置的概率会是多少?

> 等价于这个描述：2-99号乘客登机后如果发现1号(疯子)坐在本属于自己的位子上，就会请疯子离开，然后疯子再随机找个空座。这样到100号登机时，2-99号都在自己座位上，1号疯子在自己座位上和100号乘客座位上概率相同，所以是1/2。
>
> 假设疯子坐到了第2号位置上，第二个人因此根据题目随机坐到1和3-100位置上，如果2号把疯子赶走后疯子再随机坐到1和3-100位置上，对于第3号人没有任何区别。从3号的角度看都是有一个坐在2号座位上的人，还有一个随机找座位的“疯子”

7、河里的水是无限的,现在有两个水桶分别是5L、6L，问如何从河里取3L水?

> 解1
> 设：A为5L 。B为6L。
> 解：
> （1）5L的装满,全倒向6L中；此时B中有5L水（空1L）.
> （2）5L的再装满,再倒向6L中,此时只能倒入1L；此时A剩有4L水.
> （3）把B中的的水全倒掉,把A中的4L倒入B中；此时B中有4L水（空2L）,A为空.
> （4）把A装满,倒向B,只能倒入2L,A中还剩3L.

8、64匹马，8个赛道，找出跑得最快的4匹马

>1、全部马分为8组，每组8匹，每组各跑一次，然后淘汰掉每组的后四名
>
>2、8组，取每组第一名进行一次比赛，然后淘汰最后四名所在组的所有马，产生了总冠军
>
>3、第一名组取前3名，第二名组取前3名，第三名组取前2名，第四名组第一名暂时不比。
>
>4、如果出现第1，2是二三组的第一名，那么第四组第一名需要加赛一场。

10、A B C D E 海盗，他们要瓜分 100 个金币。A B C D E，轮流提议，提议的人需要获得半票及以上，不然就会被杀死，下一个继续提议，你是 A 会怎么分配？

> 反推方案：
>
> D E 时， 100,0
>
> C D E 时， 99,0,1
>
> B C D E时，99,0,1,0
>
> ABCDE时，98,0,1,0,1

11、小明离家有50米，每走一米吃一个苹果，起点有100个苹果，每次最多背50个苹果，请问最多可以拿回家多少苹果？

- 每走1m，消耗3个苹果，可以走16米，抛弃2个苹果
- 一直走，剩16个苹果

>从A地往B运送3000L汽油，两地相距1000KM，一辆汽车最多可装载1000L汽油，每行驶1KM耗油1L，请问从B地最多可以得到多少L汽油？ 
>
>- 每走1km，消耗5升油，可以走200米
>- 每走1km，消耗3升油，可以走333米，抛弃剩下的1L,
>- 一直走就行，最后剩533L油

12、圆形湖中间一只鸭，岸边一只老虎，鸭的速度为s，老虎速度为4s，湖半径为r，鸭子到岸边即可安全逃脱，问什么情况下鸭子能顺利逃脱？

> 我们暂且假定它就在一个半径为R/4的小圆上围绕圆心游走，只要经过一段时间追赶，鸭子一定会游到这样一个位置，它和老虎在同一条直径上，但位置在圆心的两边。
>
> 此时鸭子立即改为沿半径方向往岸边P点游，显然，它离岸边的距离为3R/4，它登岸需要的时间是3R/4V；而老虎到P点的距离正好是半圆，即Rπ，老虎需要的时间是Rπ/4V。因为π=3.14>3，所以 鸭子先上岸。

13、抛硬币，先抛到正面的赢，第一个抛的人赢的概率

> 1/2 +1/8 + ...，等比数列求和，答案2/3

14、23枚硬币，有10个正面朝上。现在蒙住你的眼睛，如何将硬币分为两堆，保证两堆硬币中，正面朝上的硬币数相同。 

> 先将硬币分为两组，A组10个，B组13个，假设此时B组有x个朝上，那么A组有10-x个朝上。再将A组每个硬币翻转，此时A组有`(10-(10-x))=x`枚硬币朝上，和B组朝上的硬币数相等。

15\.有两个技巧相当的赌徒 A 和 B（即两人赌博胜率各为0.5），现在设定这样的获胜规则：

1. A只要赢了2局或以上就获
2.  B要赢3局或以上才能获胜。

问双方胜率各为多少？

>  `1/4+2/8+3/16=11/16`，另一方获胜5/16

16、在岛上有100只老虎和1只羊，老虎可以吃草，但他们更愿意吃羊。老虎吃羊会变成羊。问羊会不会被吃？

> ```undefined
> 1. 1 只老虎，肯定吃;
> 2. 2 只老虎肯定不吃，否则就被另一只吃了;
> 3. 3 只老虎，如果一只老虎吃掉了羊，这样问题就转换为 2 只老虎和 1 只羊的情况，显然另外两种老虎不敢轻举妄动，所以羊会被吃；
> 4. 4 只老虎，如果某一只老虎吃了羊，问题转化为 3 只老虎和 1 只羊的问题，它肯定会被接下来的某一只吃掉，然后其他两只只能等着，所以 4 只老虎，大家都不敢吃羊；
> 我们就可以发现如果老虎数目是奇数，那么羊肯定被吃，如果是偶数，那么羊肯定不会被吃。
> ```

17、两个人数数字，1~30，最少说一个，最多说三个，怎么保证第一个人一定输或者一定赢

> 4个时，先说是输的，即4n+4的状态。即谁处于这个状态就是输的
>
> 先说的30需要将对方转换到4n+4的状态，即先说2个，剩28，后面根据对方说的数字补充4-x即可。

18、Rand5 实现 Rand7

> rand5 *5 + rand5=rand25，小于等于21时对7求余数

19、有n个灯泡，按环状摆放，0为关，1为开，现在你单次操作能改变相邻三个灯泡的状态。能否将所有灯泡关掉？试着去证明你做法的正确性

> 如果给100个灯泡编号0～99 ，顺序往下走，如果灯是亮的就不管，灭的就按下后面灯泡使这个灯亮（在第一循环内，也就是还没有从99走到0）。
>
> 98号灯是灭的，看99号灯是灭还是亮。如果99号灯是灭的，那就再按一下99号灯的开关，这样全环只有一个灭的灯（0号灯）；如果99号灯是亮的，这样只有98号灯是灭的。总之，全环只有一个灯是灭的。
>
> 第二阶段，将那个灭的灯标记为0号灯，后面的灯依次按顺序排序，每3个一组，中间的那个灯按开关，整个循环跑完，所有的灯都灭了。
>
> 第三阶段，每个灯泡的开关都按一遍。以上步骤就可以使最后的灯泡全亮。

20、两个球，100层楼，每个球在一定高度扔下去会碎，怎么用最少的次数判断几层楼会把球摔碎？

> 动态规划
>
> 找到子问题。做如下的分析，假设f{n}表示从第n层楼扔下鸡蛋，没有摔碎的最少尝试次数。第一个鸡蛋，可能的落下位置(1,n)，第一个鸡蛋从第i层扔下，有两个情况：
>
> - 碎了，第二个鸡蛋，需要从第一层开始试验，有i-1次机会
> - 没碎，两个鸡蛋，还有n-i层。这个就是子问题了f{n-i} 所以，当第一个鸡蛋，由第i个位置落下的时候，要尝试的次数为1 + max(i - 1, f{n - i})，那么对于每一个i，尝试次数最少的，就是f{n}的值。状态转移方程如下： f{n} = min(1 + max(i - 1, f{n - 1}) ) 其中: i的范围为(1, n), f{1} = 1 。
>
> 数学推导
>
> 假设最少尝试次数为x，那么，第一个鸡蛋必须要从第x层扔下，如果碎了，前面还有x - 1层楼可以尝试，如果没碎，后面还有x-1次机会。如果没碎，第一个鸡蛋，第二次就可以从x +（x - 1）层进行尝试，为什么是加上x - 1，因为，当此时，第一个鸡蛋碎了，第二个鸡蛋还有可以从x+1 到 x + (x - 1) - 1层进行尝试，有x - 2次。如果还没碎，那第一个鸡蛋，第三次从 x + (x - 1) + (x - 2)层尝试。碎或者没碎，都有x - 3次尝试机会，依次类推。那么，x次的最少尝试，可以确定的最高的楼层是多少呢？ x + (x - 1) + (x - 2) + … + 1 = x(x+1) / 2 那反过来问，当最高楼层是100层，最少需要多少次呢？x(x+1)/2 >= 100, 得到x>=14，最少要尝试14次。

20、十层楼，每层楼有一颗钻石，大小不一，电梯从下往上，每层楼可以打开电梯门查看，但是只能拿走一颗，且不能交换，请问有什么方法可以拿到最大的

>37%法则：先放弃前 37%（就是1/e）的钻石，此后选择比前 37% 都大的第一颗钻石。

## 问题分析

### 哈希函数的冲突如何避免？

双哈希或者多哈希

### 直播视频卡住了，分析原因

- 帧率太低 如果主播端手机性能较差，或者有很占 CPU 的后台程序在运行，可能导致视频的帧率太低。
- 上传阻塞 主播的手机在推流时会源源不断地产生音视频数据，但如果手机的上传网速太小，那么产生的音视频数据都会被堆积在主播的手机里传不出去，上传阻塞会导致全部观众的观看体验都很卡顿。
- 下行不佳 就是观众的下载带宽跟不上或者网络很波动

### 线程池的核心线程数大小如何确定

- 计算密集型为N（N为CPU总核数）+1，IO密集型为2N。

$N_{threads}=N_{cpu}*U_{cpu}*(1+W/C),W等待时间,C计算时间$

## 参考

http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html

http://www.ruanyifeng.com/blog/2014/05/oauth_2_0.html

http://www.ruanyifeng.com/blog/2019/04/oauth_design.html