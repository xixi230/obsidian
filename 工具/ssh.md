远程开发利器
你可以在任意设备打开ssh连接到另一台无论物理空间多远的设备进行开发


---
>ssh是每一台linux电脑的标准配置


想象一下，你坐在咖啡馆，你放寒假在家里，用自己的一台破旧的小笔记本远程控制自己的学校的另一台电脑（比如运行代码或者传输文件）

SSH就是一个安全遥控器！！
 它能让你通过命令行（类似打字输入指令的方式）远程操作另一台电脑，而且所有操作都是加密的（就像用密码锁保护对话内容，别人看不懂）
 
 
 
### 安装ssh客户端
 debian类系统
 ```bash
 sudo apt install openssh-client
 ```
 redhat类系统
 ```shell
  sudo dnf install openssh-client
 ```
windows:
```shell
Add-WindowsCapability -Online -Name "OpenSSH.Client~~~~0.0.1.0"
```

### ssh通讯的原理


