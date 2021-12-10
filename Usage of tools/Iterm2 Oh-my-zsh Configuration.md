Posted on 2018-12-28

### 安装 [oh-my-zsh](https://github.com/robbyrussell/oh-my-zsh)

获取安装脚本：`wget https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh`

执行脚本：`./install.sh`

安装成功后效果如下图：

<img src="https://tryme.wang/usr/images/sina/5cd95d1d56135.jpg" alt="image" width="1620" align="center">

------
### 安装及配置 [Melso](https://github.com/powerline/fonts) 字体

#### 安装
依次执行如下命令：
```sh
# clone到本地
git clone https://github.com/powerline/fonts.git
# 安装字体
cd fonts
./install.sh
# 清理
cd ..
rm -rf fonts
```

#### 配置
依次打开：<code>iTerm2>Preferences>Profiles>Text>Change Font</code>，如下图：

<img src="https://tryme.wang/usr/images/sina/5cd95d1d929cc.jpg" alt="image" width="1836" align="center">

------
### 配置 Solarized Dark 配色方案

依次打开：<code>iTerm2>Preferences>Profiles>Colors>Color prosets…</code>，勾选 Solarized Dark 即可。

------
### 配置使用 [agnoster](https://github.com/robbyrussell/oh-my-zsh/blob/master/themes/agnoster.zsh-theme) 主题

使用 vim 编辑 .zshrc 文件，设置 <code>ZSH_THEME="agnoster"</code> 即可。

------
### 安装语法高亮插件 [zsh-syntax-highlighting](https://github.com/zsh-users/zsh-syntax-highlighting)

两种方式，一种是从 GitHub clone 到本地，另一种是直接使用 <code>homebrew</code> 安装：
`brew install zsh-syntax-highlighting`
之后在 <code>.zshrc</code> 中加入：
`source /usr/local/share/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh`

------
### 配置个好玩的表情

编辑 <code>vim ~/.oh-my-zsh/themes/agnoster.zsh-theme</code>，修改大概92行对应一段为：

```shell
prompt_context() {
    if [[ "$USER" != "$DEFAULT_USER" || -n "$SSH_CLIENT" ]]; then
      prompt_segment black default "✨"
    fi
}
```

------
### 配置完成
效果如下：

<img src="https://tryme.wang/usr/images/sina/5cd95d1eef7cd.jpg" alt="image" width="1620" align="center">

---
### 问题
- IntelliJ IDEA 的 Terminal 终端中文乱码？
  **解决：** 编辑~/.zshrc 文件，添加：
  ```
  export LANG="zh_CN.UTF-8"
  export LC_ALL="zh_CN.UTF-8"
  ```
  执行 `source ~/.zshrc` 即可解决问题。
