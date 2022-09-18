开启抓包方式：

1.  windows 上运行 ipconfig 命令，获取以太网适配器 vEthernet (WSL) 下的 IPv4 地址

2. adb shell settings put global http_proxy IP:PORT，IP 填入上一步获取的 ip 地址，PORT 填入你的抓包工具监听的端口。
3. 或者使用 8888 为fiddler 代理端口 adb shell "settings put global http_proxy `ip route list match 0 table all scope global | cut -F3`:8888" 
```bash

#连接设备
adb connect 127.0.0.1:58526

#设置全局代理方式1
adb shell settings put global http_proxy 172.18.0.1:8888
#删除全局代理
adb shell settings delete global http_proxy
adb shell settings delete global global_http_proxy_host
adb shell settings delete global global_http_proxy_port

#设置全局代理方式2
adb shell "settings put global http_proxy `ip route list match 0 table all scope global | cut -F3`:8888"
#删除全局代理
adb shell settings put global http_proxy :0


adb reboot
```
