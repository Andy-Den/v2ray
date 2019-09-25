# v2ray
一键创建v2ray网络代理服务器访问外网
参考1: https://blog.chaos.run/dreams/debian-server-deploy-v2ray/index.html
参考2: https://www.imcaviare.com/2018-12-18-1/

<b>v2ray服务器安装(请使用root用户安装)</b>
下载脚本：
  wget https://install.direct/go.sh
执行脚本安装 V2Ray:
  ./go.sh
配置v2ray：
  nano /etc/v2ray/config.json
配置文件如下:
{
  "inbounds": [{
    "port": 10559,   //需要用在客户端配置上
    "protocol": "vmess",
    "settings": {
      "clients": [
        {
          "id": "8d28f71b-88c8-40b5-8c77-3b8966d7ed0c",   //需要用在客户端配置上
          "level": 1,
          "alterId": 4   //默认为64个tcp调整为4个
        }
      ]
    }
  }],
  "outbounds": [{
    "protocol": "freedom",
    "settings": {}
  },{
    "protocol": "blackhole",
    "settings": {},
    "tag": "blocked"
  }],
  "routing": {
    "domainStrategy": "IPOnDemand",
    "rules": [
      {
        "type": "field",
        "ip": ["geoip:private"],
        "outboundTag": "blocked"
      }
    ]
  }
}

配置完成后启动v2ray(重启: service v2ray restart), 不知道v2ray是否启动的情况下使用如下重启并查看启动状态:
  service v2ray stop && service v2ray start && service v2ray status
服务器如有防火墙影响则需要放开v2ray端口(查看端口号: netstat -ntulp | grep v2ray ):
  ufw allow 端口号
到此服务器安装配置已经完成.

v2ray客户端安装:
  1.linux桌面版系统的客户端安装与服务器安装相同.
  2.Windows 中的可执行文件为 v2ray.exe 和 wv2ray.exe。双击即可运行。
    * v2ray.exe 是一个命令行程序，启动后可以看到命令行界面。
    * wv2ray.exe 是一个后台程序，没有界面，会在后台自动运行。
  3.macOS 中的可执行文件为 v2ray。右键单击，然后选择使用 Terminal 打开即可。

客户端配置:
  1.linux桌面版系统的客户端配置文件(nano /etc/v2ray/config.json):  
{
  "log": {
    "loglevel": "warning"
  },
  "inbounds": [{
    "port": 1082,            //此端口用于windows,mac,linux或者应用程序的代理设置
    "listen": "127.0.0.1",   //此地址用于windows,mac,linux或者应用程序的代理设置
    "protocol": "socks",     //此入口协议用于windows,mac,linux或者应用程序的代理设置
    "settings": {
      "udp": true
    }
  }],
  "outbounds": [
    {
      "protocol": "vmess",
      "settings": {
        "vnext": [{
          "address": "此处修改为服务器的ip",
          "port": 10559,   //此处应与服务器inbounds下的port相同
          "users": [{
            "id": "8d28f71b-88c8-40b5-8c77-3b8966d7ed0c",   //此处应与服务器inbounds下的id相同
            "alterId": 64
          }]
        }]
      }
    },
    {
      "protocol": "freedom",
      "tag": "direct",
      "settings": {}
    }
  ],
  "routing": {
    "domainStrategy": "IPOnDemand",
    "rules": [{
      "type": "field",
      "ip": ["geoip:private"],
      "outboundTag": "direct"
    }]
  }
}
配置完成后需要重启v2ray客户端.
另外: VMess 协议的认证基于时间，一定要保证服务器和客户端的系统时间相差要在一分钟以内

为电脑设置网络代理:
  请看v2ray客户端的配置中inbound监听端口:1082，协议:socks，listen:127.0.0.1来配置电脑的网络代理

配置PAC
我们配置了网络代理后会发现它是全局的 当我们访问国内网站他还是会去走代理，反而影响了国内网站的访问速度。而且还有出现一些bug例如host修改失效,这种时候就需要用到PAC它会帮我们检测分辨网站是国内还是国外是否通过代理来访问。参考:https://www.imcaviare.com/2018-12-18-1/

