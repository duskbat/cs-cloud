# WSL2 网络配置

## WSL1 & WSL2
- WSL1
    WSL1 Linux 子系统和 Windows 共享网络端口
- WSL2
    WSL2 基于 Hyper-V 运行, Linux 子系统和 Windows 在网络上是各自独立的，从 Linux 子系统访问 Windows 首先需要找到 Windows 的 IP。  

## 配置 WSL2 访问Windows上的代理
有两个关键步骤:
1. WSL2 中配置的代理要指向 Windows 的 IP  
2. Windows 上的代理客户端需要允许来自本地局域网的请求  

Linux 子系统是通过 Windows 访问网络的，所以 Linux 子系统中的网关指向的是 Windows，DNS 服务器指向的也是 Windows，基于这两个特性，我们可以将 Windows 的 IP 读取出来。
通过 `cat /etc/resolv.conf` 查看 DNS 服务器 IP;  
通过 环境变量 ALL_PROXY 配置代理:  
```sh
export ALL_PROXY="http://127.0.0.1:7890"
```
在 Windows 代理客户端上允许本地局域网请求: Allow LAN