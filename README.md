
在mac上大家一般使用Iterm2来管理远程服务器，有时可能还经过了跳板机。今天分享一下在 Mac 的iterm2下实现与服务器进行便捷的文件上传和下载操作

### 常用命令 
下载 **sz xxx.txt** 下载远程文件到本地

上传 **rz**上传本地文件到远程服务器

### 一、安装支持rz和sz命令的lrzsz
> brew install lrzsz

### 二、创建2个脚本
比如保存到当前用户目录下  ~/inke_login/

1、iterm2-recv-zmodem.sh

```
#!/bin/bash
 
osascript -e 'tell application "iTerm2" to version' > /dev/null 2>&1 && NAME=iTerm2 || NAME=iTerm
if [[ $NAME = "iTerm" ]]; then
    FILE=$(osascript -e 'tell application "iTerm" to activate' -e 'tell application "iTerm" to set thefile to choose folder with prompt "Choose a folder to place received files in"' -e "do shell script (\"echo \"&(quoted form of POSIX path of thefile as Unicode text)&\"\")")
else
    FILE=$(osascript -e 'tell application "iTerm2" to activate' -e 'tell application "iTerm2" to set thefile to choose folder with prompt "Choose a folder to place received files in"' -e "do shell script (\"echo \"&(quoted form of POSIX path of thefile as Unicode text)&\"\")")
fi
 
if [[ $FILE = "" ]]; then
    echo Cancelled.
    # Send ZModem cancel
    echo -e \\x18\\x18\\x18\\x18\\x18
    sleep 1
    echo
    echo \# Cancelled transfer
else
    cd "$FILE"
    /usr/local/bin/rz -E -e -b --bufsize 4096
    sleep 1
    echo
    echo
    echo \# Sent \-\> $FILE
fi
```
2、iterm2-send-zmodem.sh


```
#!/bin/bash
 
osascript -e 'tell application "iTerm2" to version' > /dev/null 2>&1 && NAME=iTerm2 || NAME=iTerm
if [[ $NAME = "iTerm" ]]; then
    FILE=`osascript -e 'tell application "iTerm" to activate' -e 'tell application "iTerm" to set thefile to choose file with prompt "Choose a file to send"' -e "do shell script (\"echo \"&(quoted form of POSIX path of thefile as Unicode text)&\"\")"`
else
    FILE=`osascript -e 'tell application "iTerm2" to activate' -e 'tell application "iTerm2" to set thefile to choose file with prompt "Choose a file to send"' -e "do shell script (\"echo \"&(quoted form of POSIX path of thefile as Unicode text)&\"\")"`
fi
if [[ $FILE = "" ]]; then
    echo Cancelled.
    # Send ZModem cancel
    echo -e \\x18\\x18\\x18\\x18\\x18
    sleep 1
    echo
    echo \# Cancelled transfer
else
    /usr/local/bin/sz "$FILE" --escape --binary --bufsize 4096
    sleep 1
    echo
    echo \# Received $FILE
fi
```
### 三、在iTerm2中添加2条触发规则
#####  设置Iterm2的Tirgger特性，profiles->default->editProfiles->Advanced中的Tirgger

第1条触发规则：
选项| 值
---|---
Regular expression:  | rz waiting to receive.\*\*B0100
Action: | Run Silent Coprocess
Parameters:| ~/inke_login/iterm2-send-zmodem.sh
Instant: |checked


第2条触发规则：
选项| 值
---|---
Regular expression: |**\*\*B00000000000000**
Action: |**Run Silent Coprocess**
Parameters: |**~/inke_login/iterm2-recv-zmodem.sh**
Instant: |**checked**

#### 图片演示（我用的是中文版，中文不好的同学对着上面的英文配一下吧^_^）
![](https://tva1.sinaimg.cn/large/008i3skNly1gt6wc09yjaj30pl0h5my2.jpg)
![](https://tva1.sinaimg.cn/large/008i3skNly1gt6wcbb3g2j30po0h9q40.jpg)

![](https://tva1.sinaimg.cn/large/008i3skNly1gt6wbl6lilj30xp0d13ze.jpg)
