## WSL是什么
Windows Subsystem for Linux

## 安装前提
1.  开启cpu虚拟化-> 任务管理器 -> 性能 -> CPU
   检查CPU虚拟化已经开启， 没有开启的情况下，进入BIOS开启
2.  开启两个windows功能
	*   适用于Linux的Windows子系统
	*   虚拟机平台 Virturl Machine Platform

## 安装
管理员身份运行CMD
```
wsl -- install --web-download
wsl --list --online
wsl --list -v
wsl --set-default kali-linux 
```

## 启动
```
wsl -d Ubuntu
```
## 关闭
```
wsl --shutdown
```
## 退出
```
exit
```

## 卸载
```
wsl --unregister kali-linux 
```

## 备份
```
wsl --export Ubuntu ubuntu.tar
```

## 恢复
```
cd D:
mkdir wsl
wsl --import Ubuntu2 d:/wsl c:\Users\ubuntu.tar
wsl --list -v
```

## 命令混用
```
Get-ChildItem | wsl grep Video
```

## 显卡直通
```
nvidia-smi
```