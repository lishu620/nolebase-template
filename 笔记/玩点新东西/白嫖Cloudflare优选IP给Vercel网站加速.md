
  

> [!提示]
> 本教程参考以下内容
> [CloudFlare公共优选Cname域名地址](https://www.baota.me/post-411.html)
> [CloudFlare SAAS(cname) 接入网站域名](https://www.baota.me/post-413.html)
> [白嫖Cloudflare最优CDN给自己网站加速，尽情享受CF的免费服务](https://www.bilibili.com/video/av1306011539/)
## 准备[​](https://blog.mnxy.eu.org/posts/tech/cdn#%E5%87%86%E5%A4%87)

- 一个`Cloudflare`账号并绑定支付渠道（没有信用卡的，可使用国区PayPal绑定银联）
- 两个托管在`Cloudflare`的域名，或一个托管在`Cloudflare`另一个托管在国内DNS服务商

> 注意：必须是两个域名，不能是顶级域分出的子域名

## 配置[​](https://blog.mnxy.eu.org/posts/tech/cdn#%E9%85%8D%E7%BD%AE)

### 回源域名创建回退源地址[​](https://blog.mnxy.eu.org/posts/tech/cdn#%E5%9B%9E%E6%BA%90%E5%9F%9F%E5%90%8D%E5%88%9B%E5%BB%BA%E5%9B%9E%E9%80%80%E6%BA%90%E5%9C%B0%E5%9D%80)

> 以123.com为例

在第一个域名中，DNS -> 记录

![Clip_2024-07-05_19-11-48](https://i0.hdslb.com/bfs/article/aca2dc79b026057d554aa7eb3ec905ea381745966.png)

创建一个A类型的记录

> origin可以是@也可以是任意的子域名前缀，但我比较建议使用子域名创建。
> 
> 76.76.21.21是Vercel的IP地址
> 
> 代理状态（小云朵）务必开启，如果关闭您后续添加在自定义主机名里面的网站域名将全部回源。

### 自定义主机名添加回退源地址[​](https://blog.mnxy.eu.org/posts/tech/cdn#%E8%87%AA%E5%AE%9A%E4%B9%89%E4%B8%BB%E6%9C%BA%E5%90%8D%E6%B7%BB%E5%8A%A0%E5%9B%9E%E9%80%80%E6%BA%90%E5%9C%B0%E5%9D%80)

同样在这个域名中，SSL/TLS -> 自定义主机名（这里需要绑定支付渠道）

![Clip_2024-07-05_18-54-42](https://i0.hdslb.com/bfs/article/7840db93c99a0da836dc666b29c9a6a5381745966.png)

将上面创建的回源域名添加进回退源中

### 自定义主机名添加网站域名[​](https://blog.mnxy.eu.org/posts/tech/cdn#%E8%87%AA%E5%AE%9A%E4%B9%89%E4%B8%BB%E6%9C%BA%E5%90%8D%E6%B7%BB%E5%8A%A0%E7%BD%91%E7%AB%99%E5%9F%9F%E5%90%8D)

注意

确保回退源已经生效

![Clip_2024-07-05_19-21-42](https://i0.hdslb.com/bfs/article/0c0fdeb6315d4b8e8f19eed046afa4fa381745966.png)

这里填写你想要在Vercel项目上配置的域名，例如blog.456.com或456.com

### 域名验证及优选IP[​](https://blog.mnxy.eu.org/posts/tech/cdn#%E5%9F%9F%E5%90%8D%E9%AA%8C%E8%AF%81%E5%8F%8A%E4%BC%98%E9%80%89ip)

> 以456.com为例
> 
> 这部分如果你只有两个域名，cloudflare和国内DNS服务商配置二选一即可

#### 在Cloudflare上验证[​](https://blog.mnxy.eu.org/posts/tech/cdn#%E5%9C%A8cloudflare%E4%B8%8A%E9%AA%8C%E8%AF%81)

在第二个域名中，DNS -> 记录，分别添加两个txt记录，其中

![Clip_2024-07-05_19-26-37](https://i0.hdslb.com/bfs/article/1623176161b567398d959f13e970dad6381745966.png)

![Clip_2024-07-05_19-27-09](https://i0.hdslb.com/bfs/article/117d9ee5ab6fe16ba4d9ff8aa0303e25381745966.png)

> 举个例子：如果是456.com，那么就按照如图所示添加；如果是blog.456.com，那么都要加.blog

回到第一个域名，刷新，看证书状态和主机名是否都有效

#### 在Cloudflare上配置优选域名[​](https://blog.mnxy.eu.org/posts/tech/cdn#%E5%9C%A8cloudflare%E4%B8%8A%E9%85%8D%E7%BD%AE%E4%BC%98%E9%80%89%E5%9F%9F%E5%90%8D)

> ~~这里以社区解析`yx.887141.xyz`为例~~此解析已无效，请选择第一篇参考内容里的任意社区解析使用
> 
> Cloudflare不能直接cname社区解析上

在第一个域名中，DNS -> 记录

![Clip_2024-07-05_19-37-05](https://i0.hdslb.com/bfs/article/667aad2c48a014e9b5238a0b1276df0b381745966.png)

在第二个域名中，DNS -> 记录

![Clip_2024-07-05_19-39-14](https://i0.hdslb.com/bfs/article/8233eb29f5fd2013e057f639f56df393381745966.png)

> blog是blog.456.com，cdn.123.com是上面创建的优选域名

#### 在国内DNS服务商上验证[​](https://blog.mnxy.eu.org/posts/tech/cdn#%E5%9C%A8%E5%9B%BD%E5%86%85dns%E6%9C%8D%E5%8A%A1%E5%95%86%E4%B8%8A%E9%AA%8C%E8%AF%81)

> 以DNSPod为例，使用DCV委派

![Clip_2024-07-05_19-50-56](https://i0.hdslb.com/bfs/article/2cd8ded03e579b6b2c2886eb2ec2c432381745966.png)

![Clip_2024-07-05_19-56-09](https://i0.hdslb.com/bfs/article/2e7eb749820792c5b616d4f538912a13381745966.png)

> hostname如果为blog.456.com，则DNSPod中填写的主机记录为_acme-challenge.blog，记录值为blog.456.com.打码内容
> 
> hostname如果为456.com，则DNSPod中填写的主机记录为_acme-challenge，记录值为456.com.打码内容

然后添加

#### 在国内DNS服务商上配置优选域名[​](https://blog.mnxy.eu.org/posts/tech/cdn#%E5%9C%A8%E5%9B%BD%E5%86%85dns%E6%9C%8D%E5%8A%A1%E5%95%86%E4%B8%8A%E9%85%8D%E7%BD%AE%E4%BC%98%E9%80%89%E5%9F%9F%E5%90%8D)

![Clip_2024-07-05_19-58-41](https://i0.hdslb.com/bfs/article/876a4565251f23283f5228f2cf51d5b1381745966.png)

![Clip_2024-07-05_19-58-58](https://i0.hdslb.com/bfs/article/c6f5c14270dc6cdf515985d52bc7c8ae381745966.png)

默认线路的记录值为回退源地址，境内线路的记录值使用社区解析

### Vercel配置域名[​](https://blog.mnxy.eu.org/posts/tech/cdn#vercel%E9%85%8D%E7%BD%AE%E5%9F%9F%E5%90%8D)

将blog.456.com配置到Vercel项目上即可