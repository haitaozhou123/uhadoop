{{indexmenu_n>4}}

# 配置openvpn

按以下步骤，可以为绑定EIP的master节点或者绑定EIP的uhost安装openvpn

## 1.服务端

登录master节点或者绑定EIP的uhost，下载[安装脚本](http://uhadoop-new.ufile.ucloud.com.cn/openvpn/auto_deploy_openvpn.sh)并执行:

```
wget http://uhadoop-new.ufile.ucloud.com.cn/openvpn/auto_deploy_openvpn.sh
sh auto_deploy_openvpn.sh
```

执行完后，查看openvpn启动是否正常。

## 2.客户端

将服务端生成的key(ca.crt, client1.crt, client1.key,
ta.key)下载到本地，放到客户端配置文件同一目录下。

客户端配置文件client.ovpn内容如下：

```
client 
dev tun 
proto udp 
remote xxx.xxx.xxx.xxx 1194 ;(xxx.xxx.xx.xx是绑定的EIP) 
ca ca.crt ;(相对路径) 
cert xxx.crt ;(相对路径) 
key xxx.key ;(相对路径) 
; tls-auth ta.key 0 ;(这句要注释掉) 
comp-lzo
cipher AES-256-CBC 
user nobody ;(仅供非 windows 客户端配置，windows 请用“;”注释掉）
group nobody ;(仅供非 windows 客户端配置，windows 请用“;”注释掉） 
persist-key 
persist-tun
```

以下介绍各操作系统下客户端安装过程

### 2.1 MAC

下载[客户端安装软件](http://uhadoop-new.ufile.ucloud.com.cn/openvpn/Tunnelblick_3.6.9_build_4685.dmg)

修改配置文件后，打开Tunnelblick，添加设置文件，将client.ovpn放到私人设置文件夹。

最后，点击client.ovpn启用配置。

### 2.2 Windows

下载[客户端安装软件](http://uhadoop-new.ufile.ucloud.com.cn/openvpn/openvpn-2.2.2-install.exe)

将key和新建的client.ovpn放到c:/Program Files/OpenVPN/config/目录下。

到桌面双击openvpn图标即可。

### 2.3 Liunx

安装openvpn

```
yum install -y openvpn
```

将key和client.ovpn放在同一目录下，并在该目录下执行启动命令：

```
openvpn --daemon --config client.ovpn
```

## 3\. 配置hosts

确保服务端和客户端里，所有节点的主机名和IP相匹配，方便在浏览器服务。例如：

    10.10.229.204 uhadoop-2gz2ny-core3 
    10.10.240.154 uhadoop-2gz2ny-master2 
    10.10.227.236 uhadoop-2gz2ny-core2 
    10.10.222.134 uhadoop-2gz2ny-core1 
    10.10.243.138 uhadoop-2gz2ny-master1
