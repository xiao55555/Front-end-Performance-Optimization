# Front-end-Performance-Optimization
## 前端性能优化

[内容部分]
[css部分]
js部分
javascript, css 
图片
 cookie
移动端 
服务器
摘要：无论是在工作中，还是在面试中，web前端性能的优化都是很重要的，那么我们进行优化需要从哪些方面入手呢？可以遵循雅虎的前端优化34条军规，不过现在已经是35条了，所以可以说是雅虎前端优化的35条军规。已分类，挺好的，这样对于优化有一个比较清晰的方向
 
## 内容部分
 

### 1 尽量减少HTTP请求数

　　80%的终端用户响应时间都花在了前端上，其中大部分时间都在下载页面上的各种组件：图片，样式表，脚本，Flash等等。减少组件数必然能够减少页面提交的HTTP请求数。这是让页面更快的关键。

　　减少页面组件数的一种方式是简化页面设计。但有没有一种方法可以在构建复杂的页面同时加快响应时间呢？嗯，确实有鱼和熊掌兼得的办法。

　　合并文件是通过把所有脚本放在一个文件中的方式来减少请求数的，当然，也可以合并所有的CSS。如果各个页面的脚本和样式不一样的话，合并文件就是一项比较麻烦的工作了，但把这个作为站点发布过程的一部分确实可以提高响应时间。

　　CSS Sprites是减少图片请求数量的首选方式。把背景图片都整合到一张图片中，然后用CSS的background-image和background-position属性来定位要显示的部分。

　　图像映射可以把多张图片合并成单张图片，总大小是一样的，但减少了请求数并加速了页面加载。图片映射只有在图像在页面中连续的时候才有用，比如导航条。给image map设置坐标的过程既无聊又容易出错，用image map来做导航也不容易，所以不推荐用这种方式。

　　行内图片（Base64编码）用data: URL模式来把图片嵌入页面。这样会增加HTML文件的大小，把行内图片放在（缓存的）样式表中是个好办法，而且成功避免了页面变“重”。但目前主流浏览器并不能很好地支持行内图片。

　　减少页面的HTTP请求数是个起点，这是提升站点首次访问速度的重要指导原则。

 

### 2.减少DNS查找

　　域名系统建立了主机名和IP地址间的映射，就像电话簿上人名和号码的映射一样。当你在浏览器输入www.yahoo.com的时候，浏览器就会联系DNS解析器返回服务器的IP地址。DNS是有成本的，它需要20到120毫秒去查找给定主机名的IP地址。在DNS查找完成之前，浏览器无法从主机名下载任何东西。

　　DNS查找被缓存起来更高效，由用户的ISP（网络服务提供商）或者本地网络存在一个特殊的缓存服务器上，但还可以缓存在个人用户的计算机上。DNS信息被保存在操作系统的DNS cache(微软Windows上的”DNS客户端服务”)里。大多数浏览器有独立于操作系统的自己的cache。只要浏览器在自己的cache里还保留着这条记录，它就不会向操作系统查询DNS。

　　IE默认缓存DNS查找30分钟，写在DnsCacheTimeout注册表设置中。Firefox缓存1分钟，可以用network.dnsCacheExpiration配置项设置。(Fasterfox把缓存时间改成了1小时 P.S. Fasterfox是FF的一个提速插件)

　　如果客户端的DNS cache是空的（包括浏览器的和操作系统的），DNS查找数等于页面上不同的主机名数，包括页面URL，图片，脚本文件，样式表，Flash对象等等组件中的主机名，减少不同的主机名就可以减少DNS查找。

　　减少不同主机名的数量同时也减少了页面能够并行下载的组件数量，避免DNS查找削减了响应时间，而减少并行下载数量却增加了响应时间。我的原则是把组件分散在2到4个主机名下，这是同时减少DNS查找和允许高并发下载的折中方案。

 

### 3.避免重定向

重定向用301和302状态码，下面是一个有301状态码的HTTP头：
```javascript
HTTP/1.1 301 Moved Permanently
      Location: http://example.com/newuri
      Content-Type: text/html
```
　　浏览器会自动跳转到Location域指明的URL。重定向需要的所有信息都在HTTP头部，而响应体一般是空的。其实额外的HTTP头，比如Expires和Cache-Control也表示重定向。除此之外还有别的跳转方式：refresh元标签和JavaScript，但如果你必须得做重定向，最好用标准的3xxHTTP状态码，主要是为了让返回按钮能正常使用。

　　牢记重定向会拖慢用户体验，在用户和HTML文档之间插入重定向会延迟页面上的所有东西，页面无法渲染，组件也无法开始下载，直到HTML文档被送达浏览器。

　　有一种常见的极其浪费资源的重定向，而且web开发人员一般都意识不到这一点，就是URL尾部缺少一个斜线的时候。例如，跳转到http://astrology.yahoo.com/astrology会返回一个重定向到http://astrology.yahoo.com/astrology/的301响应（注意添在尾部的斜线）。在Apache中可以用Alias，mod_rewrite或者DirectorySlash指令来取消不必要的重定向。

　　重定向最常见的用途是把旧站点连接到新的站点，还可以连接同一站点的不同部分，针对用户的不同情况（浏览器类型，用户帐号类型等等）做一些处理。用重定向来连接两个网站是最简单的，只需要少量的额外代码。虽然在这些时候使用重定向减少了开发人员的开发复杂度，但降低了用户体验。一种替代方案是用Alias和mod_rewrite，前提是两个代码路径都在相同的服务器上。如果是因为域名变化而使用了重定向，就可以创建一条CNAME（创建一个指向另一个域名的DNS记录作为别名）结合Alias或者mod_rewrite指令。

 

### 4.让Ajax可缓存

　　Ajax的一个好处是可以给用户提供即时反馈，因为它能够从后台服务器异步请求信息。然而，用了Ajax就无法保证用户在等待异步JavaScript和XML响应返回期间不会非常无聊。在很多应用程序中，用户能够一直等待取决于如何使用Ajax。例如，在基于web的电子邮件客户端中，用户为了寻找符合他们搜索标准的邮件消息，将会保持对Ajax请求返回结果的关注。重要的是，要记得“异步”并不意味着“即时”。

　　要提高性能，优化这些Ajax响应至关重要。最重要的提高Ajax性能的方法就是让响应变得可缓存，就像在添上Expires或者Cache-Control HTTP头中讨论的一样。下面适用于Ajax的其它规则：

Gzip组件
减少DNS查找
压缩JavaScript
避免重定向
配置ETags
　　我们一起看看例子，一个Web 2.0的电子邮件客户端用了Ajax来下载用户的通讯录，以便实现自动完成功能。如果用户从上一次使用之后再没有修改过她的通讯录，而且Ajax响应是可缓存的，有尚未过期的Expires或者Cache-Control HTTP头，那么之前的通讯录就可以从缓存中读出。必须通知浏览器，应该继续使用之前缓存的通讯录响应，还是去请求一个新的。可以通过给通讯录的Ajax URL里添加一个表明用户通讯录最后修改时间的时间戳来实现，例如&t=1190241612。如果通讯录从上一次下载之后再没有被修改过，时间戳不变，通讯录就将从浏览器缓存中直接读出，从而避免一次额外的HTTP往返消耗。如果用户已经修改了通讯录，时间戳也可以确保新的URL不会匹配缓存的响应，浏览器将请求新的通讯录条目。

　　即使Ajax响应是动态创建的，而且可能只适用于单用户，它们也可以被缓存，而这样会让你的Web 2.0应用更快。

 

### 5.延迟加载组件

　　可以凑近看看页面并问自己：什么才是一开始渲染页面所必须的？其余内容都可以等会儿。

　　JavaScript是分隔onload事件之前和之后的一个理想选择。例如，如果有JavaScript代码和支持拖放以及动画的库，这些都可以先等会儿，因为拖放元素是在页面最初渲染之后的。其它可以延迟加载的部分包括隐藏内容（在某个交互动作之后才出现的内容）和折叠的图片。

　　工具可帮你减轻工作量：YUI Image Loader可以延迟加载折叠的图片，还有YUI Get utility是一种引入JS和CSS的简单方法。Yahoo!主页就是一个例子，可以打开Firebug的网络面板仔细看看。

　　最好让性能目标符合其它web开发最佳实践，比如“渐进增强”。如果客户端支持JavaScript，可以提高用户体验，但必须确保页面在不支持JavaScript时也能正常工作。所以，在确定页面运行正常之后，可以用一些延迟加载脚本增强它，以支持一些拖放和动画之类的华丽效果。

 

### 6.预加载组件

　　预加载可能看起来和延迟加载是相反的，但它其实有不同的目标。通过预加载组件可以充分利用浏览器空闲的时间来请求将来会用到的组件（图片，样式和脚本）。用户访问下一页的时候，大部分组件都已经在缓存里了，所以在用户看来页面会加载得更快。

实际应用中有以下几种预加载的类型：

无条件预加载——尽快开始加载，获取一些额外的组件。google.com就是一个sprite图片预加载的好例子，这个sprite图片并不是google.com主页需要的，而是搜索结果页面上的内容。
条件性预加载——根据用户操作猜测用户将要跳转到哪里并据此预加载。在search.yahoo.com的输入框里键入内容后，可以看到那些额外组件是怎样请求加载的。
提前预加载——在推出新设计之前预加载。经常在重新设计之后会听到：“这个新网站不错，但比以前更慢了”，一部分原因是用户访问先前的页面都是有旧缓存的，但新的却是一种空缓存状态下的体验。可以通过在将要推出新设计之前预加载一些组件来减轻这种负面影响，老站可以利用浏览器空闲的时间来请求那些新站需要的图片和脚本。
 

### 7.减少DOM元素的数量

　　一个复杂的页面意味着要下载更多的字节，而且用JavaScript访问DOM也会更慢。举个例子，想要添加一个事件处理器的时候，循环遍历页面上的500个DOM元素和5000个DOM元素是有区别的。

　　大量的DOM元素是一种征兆——页面上有些内容无关的标记需要清理。正在用嵌套表格来布局吗？还是为了修复布局问题而添了一堆的<div>s？或许应该用更好的语义化标记。

YUI CSS utilities对布局有很大帮助：grids.css针对整体布局，fonts.css和reset.css可以用来去除浏览器的默认格式。这是个开始清理和思考标记的好机会，例如只在语义上有意义的时候使用<div>，而不是因为它能够渲染一个新行。

　　DOM元素的数量很容易测试，只需要在Firebug的控制台里输入：
```javascript
document.getElementsByTagName('*').length
 ```

　　那么多少DOM元素才算是太多呢？可以参考其它类似的标记良好的页面，例如Yahoo!主页是一个相当繁忙的页面，但只有不到700个元素（HTML标签）。

 

### 8.跨域分离组件

　　分离组件可以最大化并行下载，但要确保只用不超过2-4个域，因为存在DNS查找的代价。例如，可以把HTML和动态内容部署在www.example.org，而把静态组件分离到static1.example.org和static2.example.org。

 

### 9.尽量少用iframe

　　用iframe可以把一个HTML文档插入到父文档里，重要的是明白iframe是如何工作的并高效地使用它。

<iframe>的优点：

引入缓慢的第三方内容，比如标志和广告
安全沙箱
并行下载脚本
<iframe>的缺点：

代价高昂，即使是空白的iframe
阻塞页面加载
非语义
 

### 10.杜绝404

　　HTTP请求代价高昂，完全没有必要用一个HTTP请求去获取一个无用的响应（比如404 Not Found），只会拖慢用户体验而没有任何好处。

　　有些站点用的是有帮助的404——“你的意思是xxx？”，这样做有利于用户体验，，但也浪费了服务器资源（比如数据库等等）。最糟糕的是链接到的外部JavaScript有错误而且结果是404。首先，这种下载将阻塞并行下载。其次，浏览器会试图解析404响应体，因为它是JavaScript代码，需要找出其中可用的部分。

 

 

回到顶部
css部分
 

### 11.避免使用CSS表达式

 

用CSS表达式动态设置CSS属性，是一种强大又危险的方式。从IE5开始支持，但从IE8起就不推荐使用了。例如，可以用CSS表达式把背景颜色设置成按小时交替的：

```javascript
background-color: expression( (new Date()).getHours()%2 ? "#B8D4FF" : "#F08A00" );
 ```

### 12.选择<link>舍弃@import

　　前面提到了一个最佳实践：为了实现逐步渲染，CSS应该放在顶部。

  在IE中用@import与在底部用<link>效果一样，所以最好不要用它。

 

### 13.避免使用滤镜

　　IE专有的AlphaImageLoader滤镜可以用来修复IE7之前的版本中半透明PNG图片的问题。在图片加载过程中，这个滤镜会阻塞渲染，卡住浏览器，还会增加内存消耗而且是被应用到每个元素的，而不是每个图片，所以会存在一大堆问题。

最好的方法是干脆不要用AlphaImageLoader，而优雅地降级到用在IE中支持性很好的PNG8图片来代替。如果非要用AlphaImageLoader，应该用下划线hack：_filter来避免影响IE7及更高版本的用户。

 

### 14.把样式表放在顶部

　　在Yahoo!研究性能的时候，我们发现把样式表放到文档的HEAD部分能让页面看起来加载地更快。这是因为把样式表放在head里能让页面逐步渲染。

　　关注性能的前端工程师想让页面逐步渲染。也就是说，我们想让浏览器尽快显示已有内容，这在页面上有一大堆内容或者用户网速很慢时显得尤为重要。给用户显示反馈（比如进度指标）的重要性已经被广泛研究过，并且被记录下来了。在我们的例子中，HTML页面就是进度指标！当浏览器逐渐加载页面头部，导航条，顶部logo等等内容的时候，这些都被正在等待页面加载的用户当作反馈，能够提高整体用户体验。

 

 


## js部分
 

### 15.去除重复脚本

　　页面含有重复的脚本文件会影响性能，这可能和你想象的不一样。在对美国前10大web站点的评审中，发现只有2个站点含有重复脚本。两个主要原因增加了在单一页面中出现重复脚本的几率：团队大小和脚本数量。在这种情况下，重复脚本会创建不必要的HTTP请求，执行无用的JavaScript代码，而影响页面性能。

　　IE会产生不必要的HTTP请求，而Firefox不会。在IE中，如果一个不可缓存的外部脚本被页面引入了两次，它会在页面加载时产生两个HTTP请求。即使脚本是可缓存的，在用户重新加载页面时也会产生额外的HTTP请求。

　　除了产生没有意义的HTTP请求之外，多次对脚本求值也会浪费时间。因为无论脚本是否可缓存，在Firefox和IE中都会执行冗余的JavaScript代码。

　　避免不小心把相同脚本引入两次的一种方法就是在模版系统中实现脚本管理模块。典型的脚本引入方法就是在HTML页面中用SCRIPT标签：

```javascript
<script type="text/javascript" src="menu_1.0.17.js"></script>
 ```

### 16.尽量减少DOM访问

用JavaScript访问DOM元素是很慢的，所以，为了让页面反应更迅速，应该：

缓存已访问过的元素的索引
先“离线”更新节点，再把它们添到DOM树上
避免用JavaScript修复布局问题
 

### 17.用智能的事件处理器

　　有时候感觉页面反映不够灵敏，是因为有太多频繁执行的事件处理器被添加到了DOM树的不同元素上，这就是推荐使用事件委托的原因。如果一个div里面有10个按钮，应该只给div容器添加一个事件处理器，而不是给每个按钮都添加一个。事件能够冒泡，所以可以捕获事件并得知哪个按钮是事件源。

 

### 18.把脚本放在底部

　　脚本会阻塞并行下载，HTTP/1.1官方文档建议浏览器每个主机名下并行下载的组件数不要超过两个，如果图片来自多个主机名，并行下载的数量就可以超过两个。如果脚本正在下载，浏览器就不开始任何其它下载任务，即使是在不同主机名下的。

　　有时候，并不容易把脚本移动到底部。举个例子，如果脚本是用document.write插入到页面内容中的，就没办法再往下移了。还可能存在作用域问题，在多数情况下，这些问题都是可以解决的。

　　一个常见的建议是用推迟（deferred）脚本，有DEFER属性的脚本意味着不能含有document.write，并且提示浏览器告诉他们可以继续渲染。不幸的是，Firefox不支持DEFER属性。在IE中，脚本可能被推迟，但不尽如人意。如果脚本可以推迟，我们就可以把它放到页面底部，页面就可以更快地载入。

 

回到顶部
javascript, css 
 

### 19.把JavaScript和CSS放到外面

　　很多性能原则都是关于如何管理外部组件的，然而，在这些顾虑出现之前你应该问一个更基础的问题：应该把JavaScript和CSS放到外部文件中还是直接写在页面里？

实际上，用外部文件可以让页面更快，因为JavaScript和CSS文件会被缓存在浏览器。HTML文档中的行内JavaScript和CSS在每次请求该HTML文档的时候都会重新下载。这样做减少了所需的HTTP请求数，但增加了HTML文档的大小。另一方面，如果JavaScript和CSS在外部文件中，并且已经被浏览器缓存起来了，那么我们就成功地把HTML文档变小了，而且还没有增加HTTP请求数。

　　

### 20.压缩JavaScript和CSS

　　压缩具体来说就是从代码中去除不必要的字符以减少大小，从而提升加载速度。代码最小化就是去掉所有注释和不必要的空白字符（空格，换行和tab）。在JavaScript中这样做能够提高响应性能，因为要下载的文件变小了。两个最常用的JavaScript代码压缩工具是JSMin和YUI Compressor，YUI compressor还可以压缩CSS。

　　混淆是一种可选的源码优化措施，要比压缩更复杂，所以混淆过程也更容易产生bug。在对美国前十的网站调查中，压缩可以缩小21%，而混淆能缩小25%。虽然混淆的缩小程度更高，但比压缩风险更大。

　　除了压缩外部脚本和样式，行内的<script>和<style>块也可以压缩。即使启用了gzip模块，先进行压缩也能够缩小5%或者更多的大小。JavaScript和CSS的用处越来越多，所以压缩代码会有不错的效果。

 

回到顶部
图片
 

### 21.优化图片

尝试把GIF格式转换成PNG格式，看看是否节省空间。在所有的PNG图片上运行pngcrush（或者其它PNG优化工具）
 

### 22.优化CSS Sprite

在Sprite图片中横向排列一般都比纵向排列的最终文件小
组合Sprite图片中的相似颜色可以保持低色数，最理想的是256色以下PNG8格式
“对移动端友好”，不要在Sprite图片中留下太大的空隙。虽然不会在很大程度上影响图片文件的大小，但这样做可以节省用户代理把图片解压成像素映射时消耗的内存。100×100的图片是1万个像素，而1000×1000的图片就是100万个像素了。
 

### 23.不要用HTML缩放图片

　　不要因为在HTML中可以设置宽高而使用本不需要的大图。如果需要

```javascript
<img width="100" height="100" src="mycat.jpg" alt="My Cat" />
```
　　那么图片本身（mycat.jpg）应该是100x100px的，而不是去缩小500x500px的图片。

 

### 24.用小的可缓存的favicon.ico（P.S. 收藏夹图标）

　　favicon.ico是放在服务器根目录的图片，它会带来一堆麻烦，因为即便你不管它，浏览器也会自动请求它，所以最好不要给一个404 Not Found响应。而且只要在同一个服务器上，每次请求它时都会发送cookie，此外这个图片还会干扰下载顺序，例如在IE中，当你在onload中请求额外组件时，将会先下载favicon。

所以为了缓解favicon.ico的缺点，应该确保：

足够小，最好在1K以下
设置合适的有效期HTTP头（以后如果想换的话就不能重命名了），把有效期设置为几个月后一般比较安全，可以通过检查当前favicon.ico的最后修改日期来确保变更能让浏览器知道。
 

回到顶部
 cookie
 

### 25.给Cookie减肥

　　使用cookie的原因有很多，比如授权和个性化。HTTP头中cookie信息在web服务器和浏览器之间交换。重要的是保证cookie尽可能的小，以最小化对用户响应时间的影响。

清除不必要的cookie
保证cookie尽可能小，以最小化对用户响应时间的影响
注意给cookie设置合适的域级别，以免影响其它子域
设置合适的有效期，更早的有效期或者none可以更快的删除cookie，提高用户响应时间
 

### 26.把组件放在不含cookie的域下

　　当浏览器发送对静态图像的请求时，cookie也会一起发送，而服务器根本不需要这些cookie。所以它们只会造成没有意义的网络通信量，应该确保对静态组件的请求不含cookie。可以创建一个子域，把所有的静态组件都部署在那儿。

　　如果域名是www.example.org，可以把静态组件部署到static.example.org。然而，如果已经在顶级域example.org或者www.example.org设置了cookie，那么所有对static.example.org的请求都会含有这些cookie。这时候可以再买一个新域名，把所有的静态组件部署上去，并保持这个新域名不含cookie。Yahoo!用的是yimg.com，YouTube是ytimg.com，Amazon是images-amazon.com等等。

　　把静态组件部署在不含cookie的域下还有一个好处是有些代理可能会拒绝缓存带cookie的组件。有一点需要注意：如果不知道应该用example.org还是www.example.org作为主页，可以考虑一下cookie的影响。省略www的话，就只能把cookie写到*.example.org，所以因为性能原因最好用www子域，并且把cookie写到这个子域下。

 

 

##移动端 
 

### 27.保证所有组件都小于25K

 

　　这个限制是因为iPhone不能缓存大于25K的组件，注意这里指的是未压缩的大小。这就是为什么缩减内容本身也很重要，因为单纯的gzip可能不够。

 

### 28.把组件打包到一个复合文档里

 

　　把各个组件打包成一个像有附件的电子邮件一样的复合文档里，可以用一个HTTP请求获取多个组件（记住一点：HTTP请求是代价高昂的）。用这种方式的时候，要先检查用户代理是否支持（iPhone就不支持）。

 

 
## 服务器
 

### 29.Gzip组件

　　前端工程师可以想办法明显地缩短通过网络传输HTTP请求和响应的时间。毫无疑问，终端用户的带宽速度，网络服务商，对等交换点的距离等等，都是开发团队所无法控制的。但还有别的能够影响响应时间的因素，压缩可以通过减少HTTP响应的大小来缩短响应时间。

从HTTP/1.1开始，web客户端就有了支持压缩的Accept-Encoding HTTP请求头。

```javascript
Accept-Encoding: gzip, deflate
```
　　如果web服务器看到这个请求头，它就会用客户端列出的一种方式来压缩响应。web服务器通过Content-Encoding相应头来通知客户端。

```javascript
Content-Encoding: gzip
```
　　尽可能多地用gzip压缩能够给页面减肥，这也是提升用户体验最简单的方法。

 

 

### 30.避免图片src属性为空

Image with empty string src属性是空字符串的图片很常见，主要以两种形式出现：

straight HTML
```javascript
<img src=””>
```
JavaScript
```javascript
var img = new Image();
img.src = “”;
```
这两种形式都会引起相同的问题：浏览器会向服务器发送另一个请求。

　

### 31.配置ETags

　　实体标签（ETags），是服务器和浏览器用来决定浏览器缓存中组件与源服务器中的组件是否匹配的一种机制（“实体”也就是组件：图片，脚本，样式表等等）。添加ETags可以提供一种实体验证机制，比最后修改日期更加灵活。一个ETag是一个字符串，作为一个组件某一具体版本的唯一标识符。唯一的格式约束是字符串必须用引号括起来，源服务器用相应头中的ETag来指定组件的ETag：

```javascript
HTTP/1.1 200 OK
      Last-Modified: Tue, 12 Dec 2006 03:03:59 GMT
      ETag: "10c24bc-4ab-457e1c1f"
      Content-Length: 12195
 ```
　　然后，如果浏览器必须验证一个组件，它用If-None-Match请求头来把ETag传回源服务器。如果ETags匹配成功，会返回一个304状态码，这样就减少了12195个字节的响应体。
  
```javascript
GET /i/yahoo.gif HTTP/1.1
      Host: us.yimg.com
      If-Modified-Since: Tue, 12 Dec 2006 03:03:59 GMT
      If-None-Match: "10c24bc-4ab-457e1c1f"
      HTTP/1.1 304 Not Modified
```　

 

### 32.对Ajax用GET请求

　　Yahoo!邮箱团队发现使用XMLHttpRequest时，浏览器的POST请求是通过一个两步的过程来实现的：先发送HTTP头，在发送数据。所以最好用GET请求，它只需要发送一个TCP报文（除非cookie特别多）。IE的URL长度最大值是2K，所以如果要发送的数据超过2K就无法使用GET了。

POST请求的一个有趣的副作用是实际上没有发送任何数据，就像GET请求一样。正如HTTP说明文档中描述的，GET请求是用来检索信息的。所以它的语义只是用GET请求来请求数据，而不是用来发送需要存储到服务器的数据。

 

 

### 33.尽早清空缓冲区

　当用户请求一个页面时，服务器需要用大约200到500毫秒来组装HTML页面，在这期间，浏览器闲等着数据到达。PHP中有一个flush()函数，允许给浏览器发送一部分已经准备完毕的HTML响应，以便浏览器可以在后台准备剩余部分的同时开始获取组件，好处主要体现在很忙的后台或者很“轻”的前端页面上（P.S. 也就是说，响应时耗主要在后台方面时最能体现优势）。

　　较理想的清空缓冲区的位置是HEAD后面，因为HTML的HEAD部分通常更容易生成，并且允许引入任何CSS和JavaScript文件，这样就可以让浏览器在后台还在处理的时候就开始并行获取组件。

例如：
```javascript
 ... <!-- css, js -->
    </head>
    <?php flush(); ?>
    <body>
      ... <!-- content -->
 ```

### 34.使用CDN（内容分发网络）

　　用户与服务器的物理距离对响应时间也有影响。把内容部署在多个地理位置分散的服务器上能让用户更快地载入页面。但具体要怎么做呢？

　　实现内容在地理位置上分散的第一步是：不要尝试去重新设计你的web应用程序来适应分布式结构。这取决于应用程序，改变结构可能包括一些让人望而生畏的任务，比如同步会话状态和跨服务器复制数据库事务（翻译可能不准确）。缩短用户和内容之间距离的提议可能被推迟，或者根本不可能通过，就是因为这个难题。

　　记住终端用户80%到90%的响应时间都花在下载页面组件上了：图片，样式，脚本，Flash等等，这是业绩黄金法则。最好先分散静态内容，而不是一开始就重新设计应用程序结构。这不仅能够大大减少响应时间，还更容易表现出CDN的功劳。

　　内容分发网络（CDN）是一组分散在不同地理位置的web服务器，用来给用户更高效地发送内容。典型地，选择用来发送内容的服务器是基于网络距离的衡量标准的。例如：选跳数（hop）最少的或者响应时间最快的服务器。

 

### 35.添上Expires或者Cache-Control HTTP头

这条规则有两个方面：

对于静态组件：通过设置一个遥远的将来时间作为Expires来实现永不失效
多余动态组件：用合适的Cache-ControlHTTP头来让浏览器进行条件性的请求
　　网页设计越来越丰富，这意味着页面里有更多的脚本，图片和Flash。站点的新访客可能还是不得不提交几个HTTP请求，但通过使用有效期能让组件变得可缓存，这避免了在接下来的浏览过程中不必要的HTTP请求。有效期HTTP头通常被用在图片上，但它们应该用在所有组件上，包括脚本、样式和Flash组件。

　　浏览器（和代理）用缓存来减少HTTP请求的数目和大小，让页面能够更快加载。web服务器通过有效期HTTP响应头来告诉客户端，页面的各个组件应该被缓存多久。用一个遥远的将来时间做有效期，告诉浏览器这个响应在2010年4月15日前不会改变。

```javascript
Expires: Thu, 15 Apr 2010 20:00:00 GMT
```　

如果你用的是Apache服务器，用ExpiresDefault指令来设置相对于当前日期的有效期。下面的例子设置了从请求时间起10年的有效期：
```javascript
ExpiresDefault "access plus 10 years"
```
