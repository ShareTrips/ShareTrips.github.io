# 最新零基础保姆级VPN搭建教程！ 自建独享VPN节点！


> 自建科学上网节点包含：域名注册、域名解析、https证书申请、购买服务器、套CDN（可选）、Nginx 配置。  
> 所有操作我尽可能都编写成脚本， 按照脚本一步步执行即可完成。


## 自建科学上网独享节点、使用双向加密协议
1、注册一个域名（如果有则无需注册，域名必须是正常状态）[域名状态检测](https://api.uouin.com/)  
2、服务器一台  
&emsp;&emsp; 1）$49.9/年（优惠码：）[搬瓦工访问地址](https://bwh81.net/)  
&emsp;&emsp; 2）秒杀价：$2.49/月 [raksmart](https://www.raksmart.com/home/article/details.html?a_id=198)  
&emsp;&emsp; 3）服务器自行选择以上测试都可以。

## Xray 一键部署
```

# 执行安装服务Xray服务
sudo bash -c "$(curl -L https://github.com/XTLS/Xray-install/raw/main/install-release.sh)" @ install

# 我在这里贴一个 xray 的官方仓库吧，
# 免得小伙伴找错：https://github.com/XTLS/Xray-core。
# 社区版 v2ray 的仓库地址：https://github.com/v2fly/v2ray-core。

```

## 一键部署 https 工具
```
# 执行一下命令安装生成 https 证书工具(免费申请证书)
yum -y install epel-release && yum -y install certbot
```

## 一键部署 nginx
```
# 执行安装 nginx 
yum install nginx
```
