# 最新零基础保姆级VPN搭建教程！ 自建独享VPN节点！


> 自建科学上网节点包含：域名注册、域名解析、https证书申请、购买服务器、套CDN（可选）、Nginx 配置。  
> 所有操作我尽可能都编写成脚本， 按照脚本一步步执行即可完成。


## 部署加密VPN协议节点准备工作
1、注册一个域名（如果有则无需注册，域名必须是正常状态）[域名状态检测](https://api.uouin.com/)   
&emsp;&emsp; 1）推荐 dynadot 支持中文、支持支付宝付款 [官网地址](https://www.dynadot.com/zh)  

2、服务器一台  
&emsp;&emsp; 1）$49.9/年（优惠码：）[搬瓦工官网地址](https://bwh81.net/)  
&emsp;&emsp; 2）秒杀价：$2.49/月 [RAKsmart官网](https://www.raksmart.com/home/article/details.html?a_id=198)  
&emsp;&emsp; 3）服务器自行选择以上测试都可以。

*** 

### 1、执行行安装服务Xray服务
```bash
# 注意服务器为 CentOS, 
sudo bash -c "$(curl -L https://github.com/XTLS/Xray-install/raw/main/install-release.sh)" @ install

```


### 2、查看服务器的 uuid 查询后记下来
```bash
# 注意服务器为 CentOS, Ubuntu 系列服务器查询命令有区别
cat /proc/sys/kernel/random/uuid 
```

### 3、编辑配置文件 xray 服务器配置文件
```bash
# 安装 vim 编辑工具
yum install -y vim
# 新建配置文件
vim /usr/local/etc/xray/config.json
```
<details>
  <summary>**<font color=red>注意</font>**修改成自己服务器的 UUID， 修改完成后删除备注</summary>
  ```bash
  {
    "log": {
      "loglevel": "warning"
    },
    "inbounds": [
      {
        "listen": "/dev/shm/Xray-VLESS-WSS-Nginx.socket,0666",
        "protocol": "vless",
        "settings": {
          "clients": [
            {
              "id": "a9e8d878-1ec4-40a7-9630-12abc74b2c0b" # 这里填写你自己服务器的 uuid
            }
          ],
          "decryption": "none"
        },
        "streamSettings": {
          "network": "ws",
          "wsSettings": {
            "path": "/book" # 这里填写你的路径，自定义，于nginx 配置文件哪里是一致的
          }
        }
      }
    ],
    "outbounds": [
      {
        "tag": "direct",
        "protocol": "freedom",
        "settings": {}
      },
      {
        "tag": "blocked",
        "protocol": "blackhole",
        "settings": {}
      }
    ],
    "routing": {
      "domainStrategy": "AsIs",
      "rules": [
        {
          "type": "field",
          "ip": [
            "geoip:private"
          ],
          "outboundTag": "blocked"
        }
      ]
    }
  }

  ```
</details>

#### 3.2、启动 xray 服务
```bash
# 启动 xray 服务
systemctl start xray
# 启动后查询是否启动成功
ps -ef | grep 'xray'
```

### 4、安装 nginx 服务
```bash
# 添加 CentOS7 nginx yum 资源库
sudo rpm -Uvh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm

# 使用 yum 安装 nginx 
yum install nginx

```
### 5、增加 nginx 配置文件
```bash
vim /etc/nginx/conf.d/vless.conf
```
<details>
  <summary>**<font color=red>注意</font>** Nginx配置文件，修改域名后在复制，修改后删除配置文件中的注释信息</summary>
  ```bash
  server {
    listen 443 ssl http2;
    server_name xxx.xxxxx.com; # 注意这里填写你自己的域名
  
    index index.html;
    root /var/www/html;
  
    ssl_certificate /etc/letsencrypt/live/xxx.xxxxx.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/xxx.xxxxx.com/privkey.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;
    
    location /book {
    if ($http_upgrade != "websocket") {
      return 404;
    }
          proxy_pass http://unix:/dev/shm/Xray-VLESS-WSS-Nginx.socket;
    proxy_redirect off;
          proxy_http_version 1.1;
          proxy_set_header Upgrade $http_upgrade;
          proxy_set_header Connection "upgrade";
          proxy_set_header Host $host;
          proxy_set_header X-Real-IP $remote_addr;
          proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
          proxy_read_timeout 52w;
      }
  }
  ```
</details>

### 6、安装 https 证书管理工具
```bash
yum -y install epel-release && yum -y install certbot
```
### 7、生成域名 https 证书
```bash
# 申请时需要关闭 80 端口， 并且域名需要解析到执行命令服务器，否则验证无法通过
# 修改邮箱、域名为自己的域名
certbot certonly --standalone --email 'xxxxxxx@sina.com' -d 'book.xxxx.com'
```

### 8、客户端配置文件
<details>
  <summary>**<font color=red>注意</font>**客户端配置文件，修改后删除注释部分</summary>
  ```bash
  {
    "log": {
      "error": "",
      "loglevel": "info",
      "access": ""
    },
    "inbounds": [
      {
        "listen": "127.0.0.1",
        "protocol": "socks",
        "settings": {
          "udp": false,
          "auth": "noauth"
        },
        "port": "1080"
      },
      {
        "listen": "127.0.0.1",
        "protocol": "http",
        "settings": {
          "timeout": 360
        },
        "port": "1087"
      }
    ],
    "outbounds": [
      {
        "mux": {
          "enabled": false,
          "concurrency": 8
        },
        "protocol": "vless",
        "streamSettings": {
          "wsSettings": {
            "path": "/book?ed=2048",  # /book 这个于服务器配置路径一致 
            "headers": {
              "host": ""
            }
          },
          "tlsSettings": {
            "allowInsecure": false
          },
          "security": "tls",
          "network": "ws"
        },
        "tag": "proxy",
        "settings": {
          "vnext": [
            {
              "address": "xxx.xxxxx.com", # 这里你自己的域名
              "users": [
                {
                  "encryption": "none",
                  "id": "a9e8d878-1ec4-40a7-9630-12abc74b2c0b",  # 服务器查询出来的 uuid 
                  "level": 0,
                  "flow": ""
                }
              ],
              "port": 443
            }
          ]
        }
      },
      {
        "tag": "direct",
        "protocol": "freedom",
        "settings": {
          "domainStrategy": "UseIP",
          "userLevel": 0
        }
      },
      {
        "tag": "block",
        "protocol": "blackhole",
        "settings": {
          "response": {
            "type": "none"
          }
        }
      }
    ],
    "dns": {},
    "routing": {
      "settings": {
        "domainStrategy": "AsIs",
        "rules": []
      }
    },
    "transport": {}
  }
  ```
</details>

### 9、已支持图形化配置 VLESS 的部分客户端列表，推荐使用：
+ Windows  
    + [v2rayN](https://github.com/2dust/v2rayN)
    + [Qv2ray](https://github.com/Qv2ray/Qv2ray)
+ Android  
    + [v2rayNG](https://github.com/2dust/v2rayNG)
    + [Kitsunebi](https://github.com/rurirei/Kitsunebi/tree/release_xtls)
+ iOS / Mac  
    + [Shadowrocket](https://apps.apple.com/app/shadowrocket/id932747118)
    + [V2RayXS](https://github.com/tzmax/V2RayXS)



