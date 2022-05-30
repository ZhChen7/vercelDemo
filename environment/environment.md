# 新手环境搭建指南 - 前端篇



## mac 准备工作

- brew - MacOS上的包管理工具
  - 推荐国内源：按提示选择下载源： `/bin/zsh -c "$(curl -fsSL https://gitee.com/cunkai/HomebrewCN/raw/master/Homebrew.sh)"`



## 开发工具

- [vscode](https://code.visualstudio.com/Download) （推荐）
- webstorm 

开发工具自行选择，不强作要求



## Git 安装

```SQL
brew install git

# 安装检查
git --version
```



## SSH 配置

```Plain%20Text
# 全局修改git的用户名和邮箱
git config --global user.name "xxxxx"
git config --global user.email xxxxx@email.com

#检查配置信息
git config --list

# 生成一个SSH-Key
ssh-keygen -t rsa -C "xxxxx@email.com"

cat ~/.ssh/id_rsa.pub
```



## nvm 安装-  Node版本控制

- 方便随时切换node版本，适应不同的项目版本

```Bash
# 第一步

sudo curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.38.0/install.sh | bash

# 第二步

export NVM_DIR="$HOME/.nvm"

[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm

[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion



# 第三步 

source ~/.zshrc



# 若遇到问题：Failed to connect to github.com port 443: Operation timed out

# 解决方案：配置本地host - switchhost

151.101.185.194 github.global-ssl.fastly.net 

192.30.253.112 github.com
```



## node & npm 安装

- 通过nvm 安装

```Nginx
# 查看node版本列表
nvm ls 

# 安装node 稳定版本
nvm install stable 

# 安装node 
nvm use stable

#node 检查
node -v

#npm 检查
npm -v
```



## 代理工具安装

- 本地代理 ： switchhost
  - 官网直接下载安装：  https://github.com/oldj/SwitchHosts/releases
- whistle 
  - 安装 ： `npm install -g whistle`
  - 使用： `w2 start`
  - 配合 [SwitchOmega](https://link.juejin.cn/?target=https://chrome.google.com/webstore/detail/proxy-switchyomega/padekgcemlokbadohgkifijomclgjgif) 插件，mock、代理比较方便。



## iterm2 安装

- iTerm2是一款相对比较好用的终端工具.
- iTerm2 下载地址：[https://www.iterm2.com/downloads.html](https://links.jianshu.com/go?to=https://www.iterm2.com/downloads.html)



## Oh My Zsh 安装

**Gitee ( 国内镜像 )**

```Bash
sh -c "$(curl -fsSL https://gitee.com/mirrors/oh-my-zsh/raw/master/tools/install.sh)"
```

参考 ：https://www.jianshu.com/p/9c3439cc3bdb



## VPN 梯子

- ### xcat —— 学习助理

- 地址： https://xcat.us/