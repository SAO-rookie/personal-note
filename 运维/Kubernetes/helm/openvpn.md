# openvpn服务端
## 使用helm安装openvpn
[参考文档](https://artifacthub.io/packages/helm/devtron/openvpn?modal=install)
### 服务端配置
1.添加openvpn仓库
```
helm repo add devtron http://helm.devtron.ai/
```
2.下载安装包,自定义选择版本
```
helm pull devtron/openvpn --version 4.2.5
# 解压安装包
tar -zxvf openvpn-4.2.5.tgz
```
3.修改配置文件 values.yml
```
image: # 第15行
  repository: jfelten/openvpn-docker
  tag: 1.1.0
  pullPolicy: IfNotPresent
service:
  type: ClusterIP
ipForwardInitContainer: true	#必须开启路由转发   第41行
persistence:   # 第56行
  enabled: true  # 自己选择是否开启
# existingClaim: efs-openvpn-dev-pvc  # 有可用的pvc 直击使用
  storageClass: "nfs-client"	# 使用的是nfs动态存储卷
  accessMode: ReadWriteOnce
  size: 50Mi
openvpn: # 第76行 下面没有就添加
  OVPN_K8S_POD_NETWORK: "10.244.0.0"	#k8s pod地址
  OVPN_K8S_POD_SUBNET: "255.255.0.0"
  OVPN_K8S_SVC_NETWORK: "10.96.0.0"	#k8s svc地址
  OVPN_K8S_SVC_SUBNET: "255.255.0.0"
```
磁盘模式,只能二选一不用的请注释
* existingClaim  有可用的pvc 直击使用
* storageClass 使用nfs动态存储卷 自动创建
1. 运行起来
```
helm install openvpn -n openvpn -create--namespace .
```
2. 获取密钥文件
**注意：运行脚本必须等待服务全部启动**
编写获取密钥的脚本
```
#!/bin/bash

if [ $# -ne 3 ]
then
  echo "Usage: $0 <CLIENT_KEY_NAME> <NAMESPACE> <HELM_RELEASE>"
  exit
fi

KEY_NAME=$1
NAMESPACE=$2
HELM_RELEASE=$3
POD_NAME=$(kubectl get pods -n "$NAMESPACE" -l "app=openvpn,release=$HELM_RELEASE" -o jsonpath='{.items[0].metadata.name}')
SERVICE_NAME=$(kubectl get svc -n "$NAMESPACE" -l "app=openvpn,release=$HELM_RELEASE" -o jsonpath='{.items[0].metadata.name}')
SERVICE_IP=$(kubectl get svc -n "$NAMESPACE" "$SERVICE_NAME" -o go-template='{{range $k, $v := (index .status.loadBalancer.ingress 0)}}{{$v}}{{end}}')
kubectl -n "$NAMESPACE" exec -it "$POD_NAME" /etc/openvpn/setup/newClientCert.sh "$KEY_NAME" "$SERVICE_IP"
kubectl -n "$NAMESPACE" exec -it "$POD_NAME" cat "/etc/openvpn/certs/pki/$KEY_NAME.ovpn" > "$KEY_NAME.ovpn"
```
执行脚本，会报错请无视
```
# 第一个参数是密钥名，第二个是namespace 第三个是软件的名称 也是openvpn
./gen-client-key.sh test-client openvpn openvpn
```
3. 修改以.ovpn结尾的密钥文件
删除第10行到第17行，添加以下内容。
```
remote {ip} {port} tcp
```
* ip: 添加可访问的ip
* port：开发的端口
# 客户端配置
## 使用vpn软件连接
[pritunl下载地址-三端通用](https://client.pritunl.com/)
将以.ovpn结尾的密钥文件导入即可
## linux命令行请按照以下策略
**注意：在云服务器千万别使用，否则无法连接服务器。服务器网络都被VPN接管了**
### 安装openvpn客户端
```
# centos 
yum install openvpn -y
# debian 
 apt install -y openvpn
```
### 编写同步服务端推送的dns信息脚步
* 使用resolvectl 更新服务端推送的dns
```
#!/usr/bin/env bash

# 简单的OpenVPN辅助脚本，用于获取并设置服务端推送的DNS信息到systemd-resolved。

log() {
    logger -s -t "openvpn-helper" "$@"
}

set_dns_resolvectl() {
    local dns_domains=""
    local dns_search_domains=""
    
    for option in "${!foreign_option_@}"; do
        option_value="${!option}"
        if [[ "$option_value" == dhcp-option\ DNS\ * ]]; then
            dns_server="${option_value#dhcp-option DNS }"
            echo "设置 DNS 服务器: ${dns_server}"
            
            # 使用 resolvectl 设置 DNS
            resolvectl dns tun0 "$dns_server"
        elif [[ "$option_value" == dhcp-option\ DOMAIN-SEARCH\ * ]]; then
            dns_search_domain="${option_value#dhcp-option DOMAIN-SEARCH }"
            dns_search_domains+="$dns_search_domain "
        else
            echo "未处理的选项: $option_value"
        fi
    done

    # 去除尾部的空格
    dns_search_domains=$(echo "$dns_search_domains" | xargs)
    if [[ -n "$dns_search_domains" ]]; then
        echo "设置 DNS 搜索域: $dns_search_domains"
        # 使用 resolvectl 设置 DNS 搜索域
        resolvectl domain tun0 $dns_search_domains
    fi
}

revert_dns_resolvectl() {
    echo "恢复系统DNS设置"
    # 使用 resolvectl 清除 DNS 和 DNS 域设置
    resolvectl dns tun0
    resolvectl domain tun0
}

flush_dns_cache() {
    echo "刷新 systemd-resolved DNS 缓存"
    resolvectl flush-caches
}

# 脚本入口
main() {
    local script_type="${1:-unknown}"

    case "$script_type" in
        up)
            log "VPN连接已建立"
            set_dns_resolvectl
            flush_dns_cache
            ;;
        down)
            log "VPN连接已断开"
            revert_dns_resolvectl
            flush_dns_cache
            ;;
        *)
            log "未知的脚本类型: $script_type"
            ;;
    esac
}

main "$@"

```

* 直接修改/etc/resolv.conf 更新服务端推送dns信息
```
#!/usr/bin/env bash

# Simple OpenVPN helper script to get and set server-pushed DNS information to /etc/resolv.conf.

log() {
    logger -s -t "openvpn-helper" "$@"
}

update_resolv_conf() {
    local dns_servers=""
    local dns_search_domains=""

    # Backup the current resolv.conf
    cp /etc/resolv.conf /etc/resolv.conf.bak

    for option in "${!foreign_option_@}"; do
        option_value="${!option}"
        if [[ "$option_value" == dhcp-option\ DNS\ * ]]; then
            dns_server="${option_value#dhcp-option DNS }"
            echo "Setting DNS server: ${dns_server}"
            
            # Add DNS server to /etc/resolv.conf
            dns_servers+="nameserver ${dns_server}\n"
        elif [[ "$option_value" == dhcp-option\ DOMAIN-SEARCH\ * ]]; then
            dns_search_domain="${option_value#dhcp-option DOMAIN-SEARCH }"
            dns_search_domains+="$dns_search_domain "
        else
            echo "Unhandled option: $option_value"
        fi
    done

    # Update /etc/resolv.conf
    {
        echo -e "$dns_servers"
        if [[ -n "$dns_search_domains" ]]; then
            echo "search $dns_search_domains"
        fi
    } > /etc/resolv.conf

    echo "DNS configuration updated"
}

revert_resolv_conf() {
    echo "Restoring system DNS settings"

    if [[ -f /etc/resolv.conf.bak ]]; then
        # Restore the backup of /etc/resolv.conf
        mv /etc/resolv.conf.bak /etc/resolv.conf
        echo "DNS configuration restored"
    else
        echo "Backup file not found, unable to restore DNS configuration"
    fi
}

flush_dns_cache() {
    echo "Flushing system DNS cache"
    # Since we can only edit /etc/resolv.conf and have no direct cache flush command,
    # restarting the networking service may be needed
    systemctl restart networking || echo "Please manually restart the networking service to clear DNS cache"
}

# Script entry point
main() {
    local script_type="${1:-unknown}"

    case "$script_type" in
        up)
            log "VPN connection established"
            update_resolv_conf
            flush_dns_cache
            ;;
        down)
            log "VPN connection disconnected"
            revert_resolv_conf
            flush_dns_cache
            ;;
        *)
            log "Unknown script type: $script_type"
            ;;
    esac
}

main "$@"

```

### 修改.ovpn文件
``` 
# 将 cipher 修改成 data-ciphers 
data-ciphers AES-256-CBC

# 在文件最后写 添加以下代码
script-security 2
up "/path/fresh-dns.sh up"
down "/path/fresh-dns.sh down"
```
### 启动openvpn
```
openvpn --daemon openvpn  --config /path/prod-client.ovpn
```

### 关闭 
```
killall openvpn

```
### docker客户端
**注意：docker容器用VPN 别用host模式和端口，单机运行即可**
1. [Dockerfile](../../容器/docker/docker-file仓库.md#自定义Openvpn镜像)

