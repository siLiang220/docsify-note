
## centos7安装zsh 和oh-my-zsh

1.  安装 zsh：在终端中执行以下命令
```sh
sudo yum install -y zsh
```
2.  切换默认 shell：执行以下命令将默认 shell 切换为 zsh
```sh
chsh -s /bin/zsh
```
3. 安装git
```sh
sudo yum install git
```
切换完成后，重新启动终端即可。
4. 安装oh-my-zsh
```sh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```
安装完成后，重新启动终端，即可看到 oh-my-zsh 的欢迎界面。