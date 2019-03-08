# 申请SSL通配符证书

今早点击我的网站之一：https://notebook.huolong.tk/ 后网页提示我的 **SSL 证书过期了**。因为， certbot 申请的免费证书有效期是三个月，三个月后就需要 renew 一下证书，而现在距离我上次申请证书已经三个月了。

于是，当然，我屁颠屁颠的跑去更新之前申请的三个域名的通配符证书。下面是我对申请、使用 SSL 证书的总结记录。

## SSL 证书简介

简单说一下这个**证书的作用**：如果我们从服务商那个购买了域名使用权，通常我们只能通过 http 访问网站（网址打头是http），申请配置 SSL 证书后我们就可以通过 https 访问网站，同时浏览器如谷歌，也会在网址前面加个小锁，代表这个网站是安全的。

以前，我们访问一个网站，网址开头通常是 http ， 因为当时 https 还没那么普及，要使用也有一定的成本，一般比较重视安全的公司才会给网站 enable https。后来，互联网发展、网站愈加丰富多彩、网站上隐私信息也越来越多，网络安全成了大问题。由于 http 不会对传输数据进行加密，所以它是极不安全的。互联网世界呼唤着安全的多的 https-它会对信息进行加密传输，并使用数字证书验证网站。互联网世界也出现了如 **Let's Encrypt** 这样的组织，开源、发布免费的 SSL 证书。使得玩耍个人网站的站主也能享受到 https 服务。

## 我为什么要用 https

实际上三个月前，我并没有使用 https 的打算，因为我打算建的个人网站并不会通信啥敏感信息。但是，为什么又申请了 https 证书呢？因为，国内 http 访问的域名是要在**工信部备案**的，而我申请的 **\.tk 免费主域名是不能备案的**。因此，用 http 访问这个域名总会被**拦截**然后提醒你去备案。而 https 访问的域名目前并不需要备案。并且，备案是需要拍照、上传材料、等待的，有点麻烦。所以，无论从主观还是客观来说，我都需要使用 https 。

## 如何为网站申请 SSL 通配符证书

我在网上搜索发现 Let's Encrypt 是比较牛的数字证书认证机构，也有 **Certbot** 这样的 shell 端工具帮助从 Let's Encrypt 申请免费的 SSL 证书，以及管理、更新证书。

要申请证书，你需要证明自己**对网站有控制权**；要使用证书，你需要对 web **服务器进行配置**。我本身在腾讯云租用了一台 Server ，能够终端上登录并操作 Server ，也将自己的网站代码部署在了这台 Server 上。在 Server 上安装的 Web 服务器则是 Nginx 。

我们通常购买的是二级域名，然后可以自己随意配置三级域名、四级域名等，也就是说我们的网站可能**不定期增加域名**，如果我们要给每个新增的域名都手动申请一个 SSL 证书就显得很繁琐，证书太多管理也是问题。当然，你可以写脚本处理这个问题。

Let's Encrypt 提供了更好的方案，它在 18 年开始支持**通配符证书**了，也就是说 你可以在申请证书的时候使用 \* 通配符匹配各种子域名。不过，申请通配符证书和申请普通的固定域名的证书是有区别的。总结就是，申请固定域名 SSL 证书的方式不一定适用于申请通配符证书，而申请通配符证书的方式也可以用于申请固定域名 SSL 证书。由于使用 Certbot 支持**同时给多个域名申请同一个证书**，因此，如果你购买了一个叫 example.com 的域名，然后想给这个域名 `example.com` 及其下所有三级域名 `*.example.com` 都申请同一个证书，你就可以采用**申请通配符域名的方式**来进行申请。

申请通配符证书需要你去网站添加一个如 `_acme-challenge.example.com` 以 `_acme-challenge`作为三级域名的 的 cname Txt 解析。这在云服务商如腾讯云的[域名解析页面](https://console.cloud.tencent.com/cns)可以很方便的添加。

下面演示下我登录自己的腾讯服务器上进行的命令行操作，目的是为了给我的网站及它下面的三级域名申请同一个证书，而这个证书对我以后新增的三级域名也有效果。

**申请、下载、配置证书分为几步：**

1. 下载 certbot 提供的 certbot-auto 辅助工具。Certbot 官网给了不同操作系统、web 服务器下的[安装指南](https://certbot.eff.org/)。
2. 命令行运行 certbot-auto 指令。
3. 登陆腾讯云，给你申请的域名添加 cname 解析，并启用解析。
4. 配置 web 服务器。我这里使用的是 Nginx。Certbot 也提供了针对不同 web 服务器的插件，能够帮你自动配置 Web 服务器，不过我这里采用手动方式。

### Certbot-auto 申请、下载证书步骤

Centos 7 系统，Nginx 服务器环境，命令行操作：

```
// 选择并进入文件夹，安装 certbot-auto
$ wget https://dl.eff.org/certbot-auto
chmod a+x certbot-auto

// 给 example.cn *.example.cn 申请通配符证书
./certbot-auto certonly  -d *.example.tk -d example.tk --manual --preferred-challenges dns --server https://acme-v02.api.letsencrypt.org/directory

// 命令行运行上述命令回车后，会有操作提示。并告诉你添加什么域名解析

// [可选]
// 服务器或本地新开一个终端窗口
// 腾讯云添加解析后，可以使用下面命令查看是否添加成功
// 通常设置解析后能够立即生效，你不需要验证是否成功生效，不过也有可能出现意外情况
// 比如，域名解析不是默认修改就生效，而你忘记启动刚添加的域名解析
// 如果成功，命令行输入下面命令 enter 后命令行返回能看到刚才添加cname的 Txt 字符串
dig  -t txt _acme-challenge.example.com @8.8.8.8

// 看到成功添加域名后
// 回到 certbot-auto 命令的窗口，键盘 enter 以下，等待一下就可以看到证书申请、保存成功消息
```

证书申请下载完成后，证书信息默认保存在 `/etc/letsencrypt/` 文件夹中。`ls /etc/letsencrypt/archive/` 就能看到下载的所有证书。也可以使用 certbot-auto 管理证书，下面是常用命令：

```
// 进入 certbot-auto 所在文件夹

./certbot-auto --help // 查看帮助
./certbot-auto certificates // 查看证书
./certbot-auto delete // 运行后会给出提示删除哪些证书
// 撤销证书
./certbot revoke --cert-path /etc/letsencrypt/archive/${YOUR_DOMAIN}/cert1.pem
./certbot renew .... // 更新证书，目前只支持证书全部更新，不支持单个
./cert certonly ... // 申请、更新单个证书的时候可使用这个命令
```

### 配置Nginx服务器

Nginx 的服务器配置默认放在 `/etc/nginx/` 中，我们可以通过编辑 `/etc/nginx/nginx.conf` 或 `/etc/nginx/conf.d/` 中的文件来配置域名访问。

如果不对Nginx 做任何配置，访问 https://example.com 会返回一个默认的模版页面，你可以通过在 `/etc/nginx/conf.d/` 中新建一个`.conf` 文件来配置对它的访问，为了区别不同的域名，可将网站域名作为 conf 文件的文件名。如下：

```
vim /etc/nginx/conf.d/example.com.conf
// 编辑上面文件，添加一个 server

server {
    listen       443 ssl;
    listen       [::]:443 ssl http2;

    server_name *.example.com;
    root    /home/www/example.com;

    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

    error_page 404 /404.html;
        location = /40x.html {
    }

    error_page 500 502 503 504 /50x.html;
        location = /50x.html {
    }
}
```

http 默认使用的是 80 端口，https 默认使用的是 443 端口。`ssl_certificate` 和 `ssl_certificate_key` 即是使用让网站使用 SSL 证书的配置项。

配置好后需要重启 Nginx 服务器。下面是 Nginx 测试、启动、停止、重启、重加载操作。

```
nginx -t // 测试nginx配置
service nginx restart
service nginx stop
service nginx start
service nginx reload / nginx -s reload
```

## 如何更新通配符证书

运行下面命令就可以更新通配符证书了。目前 renew 不支持指定域名操作。

```
./certbot-auto certonly  -d *.example.tk -d example.tk --manual --preferred-challenges dns --server https://acme-v02.api.letsencrypt.org/directory
```

## 后记

如果只是给某几个域名添加 SSL 证书可以采用不添加 cname 解析的方式，直接在服务器上进行操作，certbot-auto 也提供了命令直接支持。因此，在服务器上设置定时任务就能够在三个月有效期快到的时候自动更新证书了。对于通配符证书，大多数云服务商如阿里云、腾讯云上提供了 api 接口能让你进行更新 cname 等操作，因此你可以自己写个脚本调用它们的 api 更新证书，再设定定时任务。

以及，证书快到期时，Certbot 也会发邮件催你更新证书。


## 资料

* [Certbot](https://certbot.eff.org/)
* [使用Certbot申请CA证书](https://blog.shengpan.net/https/)
* [申请Let's Encrypt通配符证书](https://www.jianshu.com/p/c5c9d071e395)