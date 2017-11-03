# Web-Front-end-Performance-Optimization3
# 一. 输入url后的加载过程

从输入 URL 到页面加载完成的过程中都发生了什么

### 计算机网络体系结构

* 应用层(HTTP、SMTP、FTP、POP3)
* 运输层(TCP、UDP)
* 网络层(IP(路由器))
* 数据链路层(网桥(CSMA/CD、PPP))
* 物理层(集线器)
### 1. 查找域名对应IP地址

这一步包括 DNS 具体的查找过程，包括：浏览器缓存->系统缓存->路由器缓存... 
> (1) 浏览器搜索自己的 DNS 缓存（维护一张域名与 IP 地址的对应表）；
> 
> (2) 搜索操作系统中的 DNS 缓存（维护一张域名与 IP 地址的对应表）；
> 
> (3) 搜索操作系统的 hosts 文件（ Windows 环境下，维护一张域名与 IP 地址的对应表）；
> 
> (4) 操作系统将域名发送至 LDNS（本地区域名服务器），LDNS 查询 自己的 DNS 缓存（一般查找成功率在 80% 左右），查找成功则返回结果，失败则发起一个迭代 DNS 解析请求：
>> ① LDNS 向 Root Name Server （根域名服务器，如 com、net、org等的解析的顶级域名服务器的地址）发起请求，此处，Root Name Server 返回 com 域的顶级域名服务器的地址；
    
>> ② LDNS 向 com 域的顶级域名服务器发起请求，返回 baidu.com 域名服务器地址；
    
>> ③ LDNS 向 baidu.com 域名服务器发起请求，得到 www.baidu.com 的 IP 地址；
> 
> (5) LDNS 将得到的 IP 地址返回给操作系统，同时自己也将 IP 地址缓存起来；
> 
> (6)  操作系统将 IP 地址返回给浏览器，同时自己也将 IP 地址缓存起来；

### 2. 建立连接(TCP的三次握手)

(1) 主机向服务器发送一个建立连接的请求；

(2) 服务器接到请求后发送同意连接的信号；

(3) 主机接到同意连接的信号后，再次向服务器发送了确认信号 ;

> 注意：这里的三次握手中主机两次向服务器发送确认，第二次是为了防止已失效的连接请求报文段传至服务器导致错误。

### 3. 构建网页

(1) 浏览器根据 URL 内容生成 HTTP 请求，请求中包含请求文件的位置、请求文件的方式等等；

(2) 服务器接到请求后，会根据 HTTP 请求中的内容来决定如何获取相应的 HTML 文件；

(3) 服务器将得到的 HTML 文件发送给浏览器；

(4) 在浏览器还没有完全接收 HTML 文件时便开始渲染、显示网页；

(5) 在执行 HTML 中代码时，根据需要，浏览器会继续请求图片、音频、视频、CSS、JS等文件，过程同请求 HTML ；

#### 浏览器渲染展示网页过程(参见[Web-Front-end-Performance-Optimization](https://github.com/lilangxiong/Web-Front-end-Performance-Optimization))
* HTML代码转化为DOM(DOM Tree)
* CSS代码转化成CSSOM（CSS Object Model）
* 结合DOM和CSSOM，生成一棵渲染树（包含每个节点的视觉信息）(Render Tree)
* 生成布局（layout），即将所有渲染树的所有节点进行平面合成
* 将布局绘制（paint）在屏幕上

### 4. 断开连接(TCP的四次挥手)

(1) 主机向服务器发送一个断开连接的请求；

(2) 服务器接到请求后发送确认收到请求的信号；(此时服务器可能还有数据要发送至主机)

(3) 服务器向主机发送断开通知；(此时服务器确认没有要向主机发送的数据)

(4) 主机接到断开通知后断开连接并反馈一个确认信号，服务器收到确认信号后断开连接；

> 注意：这里的四次挥手中服务器两次向主机发送消息，第一次是回复主机已收到断开的请求，第二次是向主机确认是否断开，确保数据传输完毕。

# 二. 居于减少请求的前端优化措施

### 1、压缩js，合并第三方js库，或者尽可能避免第三库的使用，在SPA中使用异步加载
* [点击查看vue如何实现路由的懒加载](https://router.vuejs.org/zh-cn/advanced/lazy-loading.html)
* [点击查看webpack如何实现代码分离](https://doc.webpack-china.org/guides/code-splitting)
> 在vue中实现路由懒加载：
```javascript
    // 未设置前
    import Remonmend from 'componnents/recommend/recommend';

    // 设置后,此处也可以加上错误处理
    const Recommend = (resolve) => {
      import('components/recommend/recommend').then((module) => {
        resolve(module)
      })
    }
```
### 2、压缩、合并css
尤其在移动端的首屏加载时，尽可能的减少css的请求数量，减少页面渲染时间，提高用户体验。
### 3、合并图片（css sprites），或者使用iconFont替代，或者使用base64
> webpack的module里面设置图片使用img还是base64:
```javascript
    module: {
        rules: [
            {
                test: /\.(png|jpe?g|gif|svg)(\?.*)?$)/,
                loader: 'url-loader',
                options: {
                    limit: 10000, // 小于10000/1024kb的就会转成base64
                    name: util.assetsPath('img/[name].[hash:7].[ext]')
                }
            }
        ]
    }
```

### 4、图片较多的页面使用 lazyLoad
### 5、使用CDN加速
国内直接买个腾讯云、阿里云CDN服务，直接将静态资源网上放，我们就不用操心什么了。毕竟花钱买的。
### 6、避免不必要的cookie，减少请求时需要携带的cookie的大小、数量
cookie只保留跟用户登录有关的信息即可，其他信息可以使用localStorage、userData、indexDB存储。
