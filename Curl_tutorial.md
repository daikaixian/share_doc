## What is cURL?
说法有很多种:

 - Command line URL viewer
 - see URL
 - Client URL Request Library
 - recursive version: "Curl URL Request Library"
 
## SYNOPSIS
```bash
curl [options] [URL...]

```

## Description

> curl is a tool to transfer data from or to a server, using one of the supported protocols (DICT, FILE, FTP, FTPS, GOPHER, HTTP, HTTPS, IMAP, IMAPS, LDAP, LDAPS, POP3, POP3S, RTMP, RTSP, SCP, SFTP, SMB, SMBS, SMTP, SMTPS, TELNET and TFTP). The command is designed to work without user interaction.

## Curl With SMTP

既然是支持多协议的数据传输工具，除了本文要重点介绍的HTTP，顺便也介绍一下curl在SMTP(Simple Mail Transfer Protocol)和SMTPS(Simple Mail Transfer Protocol Secure)协议上的简单使用.

例：使用curl 发送邮件。

command(以163的smtp服务器为例):

```bash
curl smtps://smtp.163.com:465 -v --mail-from "from@163.com" --mail-rcpt "to@example.com" --ssl -u from@163.com:password -T "mail.txt" -k --anyauth
```

mail.txt(该文件的格式有一定要求，尽量参照模板来写)

```bash
#cat mail.txt
From:from@163.com
To:to@example.com
Subject: curl发送邮件标题

这里是内容，上面有一个空行别忘记了
```


## Curl with HTTP

+ Get Request

```bash
curl http://www.duitang.com/napi/people/badge/user/list/by_top/

```

+ 进度条

```bash
curl -# https://curl.haxx.se/docs/manpage.html > curltest.txt

```

+ 一次抓两 & Download

```bash
curl -O  http://cdn.duitang.com/uploads/item/201510/17/20151017121234_VPnE8.jpeg  -O http://img5.duitang.com/uploads/item/201401/23/20140123204101_GYzYw.jpeg

```

+ Post Request

```bash
curl -d 'flag=101&user=&ip=&nickname=kaishui&email=waterdkx%40163.com&msg=curl%E5%A4%A7%E6%B3%95%E5%A5%BD&refer_url='  'http://www.duitang.com/napi/leave/message/'

```

+ COOKIES

```
curl -i http://www.duitang.com/public/personal/data/

```
发现结果是302，因为没有登录。传递cookie值，效果就不一样了。

```bash
curl -b "sessionid=b61b05cc-1d7c-4f81-8dc3-22fe3812b78b" http://www.duitang.com/public/personal/data/


```

## PyCurl

libcurl的python实现。

coding for fun(刷qq空间留言脚本):

```python
# coding=utf-8
import pycurl  #引入需要使用的模块
from StringIO import StringIO

buffer = StringIO()

c = pycurl.Curl()  #初始化curl
c.setopt(c.URL, 'http://m.qzone.qq.com/cgi-bin/new/add_msgb?g_tk=1292889141')  #设置要请求的url
c.setopt(pycurl.POST, 1) #设置请求方式为post

c.setopt(pycurl.COOKIE, "__Q_w_s_hat_seed=1; __Q_w_s__QZN_TodoMsgCnt=1; qm_username=935961250; qm_sid=20d362025d1f15d63a832e6aa8862f1c,cQajeoH6RoMI.; RK=GZ3T6Pbmdu; pgv_pvi=7873096704; pgv_si=s2550439936; o_cookie=935961250; pgv_info=ssid=s3271059100; pgv_pvid=701924864; FTN5K=ab374947; ptisp=ctc; ptui_loginuin=935961250; ptcz=3397e1c813e21aaded908906fc2e89cab0e637f44375901a918637b9f46c6546; zzpaneluin=; zzpanelkey=; pt2gguin=o0935961250; uin=o0935961250; skey=@E2vYYsm0b; p_uin=o0935961250; p_skey=5nS9EHttKcH0r-rfXrHAkvS0Arxu3PvUcBZIqyeRzsU_; pt4_token=g*GAKbAZ7Mf7VwDFcOG3LTV2z-f0EhN12CRY5LNE9tU_; qzone_check=935961250_1459437225; QZ_FE_WEBP_SUPPORT=1; cpu_performance_v8=0; qqmusic_uin=; qqmusic_key=; qqmusic_fromtag=; qzmusicplayer=qzone_player_1127551524_1459437286017; rv2=80E61EF211228846FF66C52BE60D67221606747161080D43A5; property20=53E514B5D30D0E3B93E607F00F59FE8506DEA9F6636841A8B8DB176548496BE71E7AF9DC3EB81144; Loading=Yes; qzspeedup=sdch")  #设置cookie

c.setopt(pycurl.POSTFIELDS, "qzreferrer=http%3A%2F%2Fcn.qzs.qq.com%2Fqzone%2Fmsgboard%2Fmsgbcanvas.html%23page%3D1&content=%E6%B5%8B%E8%AF%95&hostUin=1597267866&uin=935961250&format=fs&inCharset=utf-8&outCharset=utf-8&iNotice=1&ref=qzone&json=1&g_tk=1292889141") #设置post参数

for i in range(0, 50):  #循环遍历
    c.setopt(c.WRITEDATA, buffer) #将返回数据放在buffer中
    c.perform()  #执行
    print(buffer.getvalue) #打印buffer信息

c.close()  #关闭curl

```



## Reference
 - [curl官方文档](https://curl.haxx.se/docs/)
 - [pycurl](http://pycurl.io/)