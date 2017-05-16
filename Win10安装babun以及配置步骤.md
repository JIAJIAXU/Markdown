---
title: Win10安装babun以及配置步骤
tags: windows10, babun, zsh, oh-my-zsh, powerline
grammar_cjkRuby: true
---


## 在win10系统中安装babun，并对其进行配置美化
### 1、babun的安装
* [首先下载babun安装包][1]，将其解压，然后在当前目录以管理员身份打开命令行工具（cmd或powershell或者是cmder），运行```install.bat```（需管理员权限）
* 也可以使用```install.bat /t "D:\target_folder"```的模式制定安装目录
* 安装完成后使用一下命令检查安装：
  * ```babun check```（用于判断环境是否正确）
  * ```babun update```(用于判断是否有新的更新包)
### 2、babun常用插件的安装和配置
* ```oh-my-zsh```的配置：babun内置使用bash，zsh，默认为zsh，并且安装了```oh-my-zsh```主题，[相应的主题样式可以去github上预览][2]，大众比较喜欢的是ys和agnoster主题，其中agnoster使用powerline样式
							![agnoster主题样式][3]
可以修改C:/user/username/下的.zsrc文件，更换主题，如```ZSH_THEME='ys',
ZSH_THEME='random'```
* powerline字体的安装：在babun中如果要使用powerline样式，则必须使用相应的[powerline字体][4]，windows下powerline字体文件必须放到```C:\Windows\Fonts\```文件夹下，然后在babun中鼠标右键点击右上方，选择设置->text，修改字体
* ```powerline-shell```插件美化：在系统根目录的```.oh-my-zsh/custom/theme/```中打开babun，
	* 输入安装命令：```git clone https://github.com/milkbikis/powerline-shell```
	* 复制```config.py.dist``` 粘贴为``` config.py```
	* 然后在当前目录下运行命令```./install.py```，在当前目录会生成powerline-shell.py文件
	* 随后创建系统根目录下的powerline-shell.py与powerline-shell文件夹下的powerline-shell进行链接
	* 最后配置.zshrc文件，在最后行加入以下语句，_在不适用powerline-shell语句时屏蔽下面的语句就行_：
	* powerline-shell/config参数：去掉 'username', 'hostname',（为了节省显示的路径，改动config.py后，需要重新执行install.py）
```bash?linenums
function powerline_precmd() {
    PS1="$(~/powerline-shell.py $? --shell zsh 2> /dev/null)"
}
function install_powerline_precmd() {
  for s in "${precmd_functions[@]}"; do
    if [ "$s" = "powerline_precmd" ]; then
      return
    fi
  done
  precmd_functions+=(powerline_precmd)
}

if [ "$TERM" != "linux" ]; then
    install_powerline_precmd
fi
```
* ```zsh-syntax-highlighting```插件的安装：
	* ```cd ~/.oh-my-zsh/custom/plugins```，进入.oh-my-zsh/custom/plugins文件夹
	```git clone git://github.com/zsh-users/zsh-syntax-highlighting.git```，git clone相应的程序包
```plugins=( [plugins...] zsh-syntax-highlighting)```，打开.zshrc文件，在plugins=()中加入```zsh-syntax-highlighting```
```source ~/.zshrc or src``` 刷新配置
* **powerline的安装**，这个暂时还不会
* 使用[**Powerlevel9k**][5]主题
	* ```cd ~/.oh-my-zsh/custom/themes```，进入.oh-my-zsh/custom/themes文件夹
	* ```git clone https://github.com/bhilburn/powerlevel9k.git ~/.oh-my-zsh/custom/themes/powerlevel9k```，git clone相应程序包
	* 修改```.zshrc```文件，设置```ZSH_THEME="powerlevel9k/powerlevel9k"```


**注：.oh-my-zsh主题的设置，互相有冲突，```oh-my-zsh、powerline-shell、Powerlevel9k```这三个主题选择一个使用**

  [1]: http://babun.github.io/
  [2]: https://github.com/robbyrussell/oh-my-zsh
  [3]: ./images/1494909340817.jpg
  [4]: https://github.com/powerline/fonts
  [5]: https://github.com/bhilburn/powerlevel9k
