# 福建师范大学停水停电通知提醒

### 1、前言

福建师范大学（旗山校区）地处福州郊区闽侯县，偶尔会因为电力设备损坏、水管老化、镇上水管爆裂等原因导致停水停电。例如，2017 年上半年，波及南部生活区（南区）的大大小小的停水停电事件将近 30 件，几乎是一周一次，其中部分有记录的事件可以在福建师范大学 [后勤集团通知](http://hqjt.fjnu.edu.cn/4236/list.htm) 页面中看到，南区的学生只能苦笑称上了“福建停水停电大学”。因常停水停电，于是萌生了写段脚本监控后勤集团的停水停电通知。  

### 2、准备

福建师范大学 [后勤集团通知](http://hqjt.fjnu.edu.cn/4236/list.htm) 页面可以查到有计划的停水停电活动。停水停电的关键词大约有以下几个：  

- 停水
- 停电
- 供电
- 供水
- 水压

因此可以监控这几个词，当出现这些关键词时，发送邮件通知。这里选用 Python 作为编程语言。配置文件采用 `yaml`，然后通过 `requests` 获取数据，`lxml` 解析页面结构。  

### 3、部署

#### 3.1、安装依赖的包  

这里，假设服务器上的 Linux 发行版为 Arch Linux 或者 Arch 系的其他发行版。部署时应该安装第三方的包。  

```  
sudo pacman -Syyu
sudo pacman -S python
sudo pacman -S python-requests python-beautifulsoup4 python-lxml python-ruamel-yaml
```

**若您使用其他发行版，例如 Ubuntu 或者 CentOS，请自行确认包名。**  

#### 3.2、YAML 配置文件

仓库中的 `hqjt.yml` 文件为 YAML 配置文件，便于人类读写，详细内容可以参考 [阮一峰的网络日志 - YAML 语言教程](http://www.ruanyifeng.com/blog/2016/07/yaml.html) 。  

（1）site  
网站根目录，网站中大量内容采用相对地址，保存时可以加上协议和域名补全为绝对地址。  

（2）certificate  

预留证书。HTTPS 是时代潮流，福建师范大学官网已经采用了 HTTPS，未来会有更多的子站采用 HTTPS。但担心管理人员不慎将证书配置错误，导致无法正常访问或抓取数据，因此这里预留证书。  

（3）keywords  

抓取的关键词。YAML 使用空格缩进（数目不重要），只要相同层级的元素左侧对齐即可，但不允许使用Tab键。  

（4）log  

日志文件，暂时未使用。  

（5）page  

福建师范大学后勤集团通知页面地址。  

（6）localdata  

未使用数据库，这里使用 csv 文件保留本地数据。**注意，需手动创建同名文件。**  

（7）subscribers  

订阅者。即邮件将发给谁。这里同样是一个列表。前为名称，后为邮箱地址。这些信息将用于发送的邮件的头部设置中。  

（8）smtp  

SMTP 设置。以 QQ 邮箱为例：  

 - SMTP 服务器： smtp.qq.com  
 - SMTP 账号：你的 QQ 邮箱地址  
 - SMTP 密码：你的 QQ 邮箱登录授权码（十六位，无空格）  

关于获取QQ 邮箱登录授权码，请参考 [什么是授权码，它又是如何设置？](http://service.mail.qq.com/cgi-bin/help?subtype=1&&id=28&&no=1001256)  

（9）mail  

邮箱的主题和发送者名称。  

（10）headers  

请求头中自定义设置，目前非必须，预留方便以后修改。  

注意：  

写完配置文件后，可以在 [YAML Lint](http://www.yamllint.com/) 在线校验是否正确 。 

#### 3.3、计划任务  

推荐使用 [Timer](https://wiki.archlinux.org/index.php/Systemd/Timers)，但 crontab 更常用，这里以 crontab 为例。  

```
crontab -e
---
0 * * * * /usr/bin/python /root/hqjt/hqjt.py >> /root/hqjt/log.txt
```

每小时访问页面一次。  


