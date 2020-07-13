# 功能介绍
[![FOSSA Status](https://app.fossa.com/api/projects/git%2Bgithub.com%2FSnapmali%2Fdiscord-monitor.svg?type=shield)](https://app.fossa.com/projects/git%2Bgithub.com%2FSnapmali%2Fdiscord-monitor?ref=badge_shield)


通过监听discord.py事件监测Discord中的消息及用户动态。

* 消息动态：可监测消息发送、消息编辑、消息删除、频道内消息标注（pin）
* 用户动态：通过Bot监视时可监测用户用户名及标签更新、Server内昵称更新、在线状态更新、游戏动态更新；使用用户（非Bot）监视时仅可监测用户用户名及标签更新、Server内昵称更新。
* Windows 10系统下可将动态推送至通知中心
* 可将监测到的动态由[酷Q](https://cqp.cc/)推送至QQ私聊及群聊
* 由Server及用户ID指定监测的Server及用户
* 可在配置文件中设置各QQ用户或群聊是否接受消息动态及用户动态推送

脚本的实现基于[discord.py库](https://pypi.org/project/discord.py/)，QQ推送部分代码参考了[lovezzzxxx](https://github.com/lovezzzxxx)大佬的[livemonitor](https://github.com/lovezzzxxx/livemonitor)脚本，在此感谢。

# 食用方法

## 环境依赖

<b>[Release](https://github.com/Snapmali/discord-monitor/releases)中发布了exe版本，在Windows下可直接运行，且包含酷Q及coolq-http-api，无需再安装依赖。</b>

基于python3.7版本编写，python3.8版本可正常运行，其他版本未测试。3.4及以下版本应无法运行。同时在Ubuntu 16.04上可正常运行。

外部依赖库：requests, discord.py, win10toast。可分别在命令行中执行`pip install requests` `pip install discord.py` `pip install win10toast`进行安装。

QQ推送部分采用[酷Q](https://cqp.cc/)及[coolq-http-api](https://github.com/richardchien/coolq-http-api/releases)插件实现。

* 在Windows下，直接下载安装酷Q软件，并在安装目录下新建`app`文件夹，将cool-http-api插件的`io.github.richardchien.coolqhttpapi.cpk`文件放入其中。运行并登录QQ小号后，右键点击悬浮窗，在应用->应用管理中启用cool-http-api插件即可。其默认监听端口为5700。
* 在Linux下，酷Q需要安装Docker环境，Docker安装有很多教程，可以百度或谷歌。之后可直接安装已启用ciilq-http-api的酷Q镜像，具体步骤请参阅[此文档](https://cqhttp.cc/docs/4.15/#/Docker)。

## 脚本运行

将`DiscordMonitor.py`和`config.json`放入同一文件夹下。运行前需要自定义`config.json`文件：

```
{
    //Discord用户或Bot的Token字段（你插的眼）
    "token": "User Token or Bot Token", 

    //上述Token是否属于Bot，是则为true，否则为false
    "is_bot": true, 

    //coolq-http-api插件的监听端口，默认为5700
    "coolq_port": 5700, 
    
    //网络代理的http地址，留空（即"proxy": ""）表示不设置代理
    "proxy": "Proxy URL, leave blank for no proxy, e.g. http://localhost:1080", 

    //非Bot用户时的轮询间隔时间，单位为秒
    "interval": 60,

    //是否将动态推送至Windows 10系统通知中，非Windows 10系统下此选项失效
    "toast": true

    "monitor": {
        //监听的用户列表，其中key为用户ID，为字符串；value为在推送中显示的名称，为字符串
        "user_id": {"User ID": "Display name", "123456789": "John Smith"},

        //监听的server列表，列表中值为服务器ID，为整型数。特别的，列表为[true]时表示监听所有Server
        "server": [1234567890, 9876543210]
        //"server: [true]"
    },
    "push": {
        //推送的QQ用户或群聊，为嵌套列表。底层列表第一个值为QQ号或群号；
        //第二个值为布尔型，表示是否推送消息动态；第三个值为布尔型，表示是否推送用户动态
        "QQ_group": [[1234567890, true, false], [9876543210, true, true]],
        "QQ_user": [[1234567890, true, false], [9876543210, true, true]]
    }
}
```

其中监测的Discord用户及Server的ID可在Discord UI中右键点击用户或Server中得到。

用于监测的Bot（电子眼）的Token可在Discord Developer中查看，非Bot用户（肉眼）的Token需在浏览器的开发者工具中获得，具体方法可观看视频[How to get your Discord Token(Youtube)](https://youtu.be/tI1lzqzLQCs)，不算复杂。

<b>需要注意，通过用户Token使用本脚本可能违反Discord使用协议（请参阅[Automated user accounts (self-bots)](https://support.discord.com/hc/en-us/articles/115002192352)），并可能导致账号封停。有条件的话建议使用Bot，否则</b>~~比如某fanbox server~~<b>请谨慎使用或使用小号（义眼）。</b>

<b>同时，通过非Bot用户监视时，利用事件监测用户动态方法失效，仅可通过定时查询api方法监测用户用户名及标签更新、Server内昵称更新，此时动态将不会及时推送，同时无法监测在线状态更新及游戏动态更新。</b>

配置文件修改完毕后，在命令行中运行`python DiscordMonitor.py`即可。推送消息中默认时区为东八区。

# 已知问题

#### 私聊推送失效

利用酷Q向私聊中推送消息时，需要双方互为好友且对方已向己方发送过消息才可向对方发送消息。

#### 编辑消息及删除消息监视失灵

目前仅可捕获脚本启动后发送的消息的编辑及删除事件，启动前的消息暂时不能获知其编辑或删除。

#### 无征兆断连

为程序监听消息发送事件时的异步执行导致主循环阻塞，已修复，问题未复现。


## License
[![FOSSA Status](https://app.fossa.com/api/projects/git%2Bgithub.com%2FSnapmali%2Fdiscord-monitor.svg?type=large)](https://app.fossa.com/projects/git%2Bgithub.com%2FSnapmali%2Fdiscord-monitor?ref=badge_large)